---
title: "A native window"
linkTitle: "Native window"
weight: 10
description: "go-widgets/window: one Open/Run API over X11, Wayland, GTK4, Cocoa, Win32, Android, wasmbox and a browser tab, all with CGO_ENABLED=0."
tags: [surfaces, window, x11, wayland, macos, windows]
---

[`go-widgets/window`](https://github.com/go-widgets/window) opens a real window
and runs a widget tree in it: layout, draw, present, translate input into
`toolkit.Event`, repeat until the window closes.

```go
w, err := window.Open(window.Config{Title: "Demo"})
if err != nil {
	return err
}
defer w.Close()
return w.Run(root) // root is any toolkit.Widget
```

`Config` also takes the initial size in **logical points** (zero asks for a
readable default), the `WM_CLASS` instance and class, a `Theme`, and a
`RenderScale`.

## How Open chooses

| Platform and environment | Back-end | How it reaches the screen |
|---|---|---|
| Linux, `$GO_WIDGETS_GTK` set | GTK4 | GTK owns the window; the framebuffer is a `GtkPicture`, and native controls are real GTK widgets above it. Needs the libgtk-4 runtime. |
| Linux, `$WAYLAND_DISPLAY` set | Wayland | xdg-shell over the compositor's unix socket |
| Linux, otherwise `$DISPLAY` | X11 | the X11 core protocol over the unix socket, with MIT-SHM |
| macOS | Cocoa | `NSWindow` and `NSView` through [go-macos/objc](https://github.com/go-macos/objc) (purego) |
| Windows | Win32 | a top-level `HWND` through user32 and gdi32 syscalls, `StretchDIBits` |
| Android, `$GW_ANDROID_SOCKET` set | Android host | a framed protocol to [the Java host]({{< relref "/surfaces/android.md" >}}), pixels in a shared memfd |
| Android, otherwise | Wayland or X11 | a shell under Termux still has a display server to dial |
| `js/wasm`, inside wasmdesk | wasmbox | a client of the [wasmdesk compositor]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}}) |
| `js/wasm`, an ordinary page | browser tab | a `<canvas>` of the page (`Config.Canvas`, default `"screen"`) through `webcanvas`: no compositor, no `SharedArrayBuffer`, no cross-origin isolation |
| anything else | — | `window.ErrUnsupported`, so a cross-build still compiles |

Every back-end is `CGO_ENABLED=0`. The X11 one is the core protocol written
from scratch over the socket — no Xlib, no XCB — with both byte orders, the
Xauthority cookie, keysym mapping, `PutImage` tiled under the server's
request size, and the MIT-SHM fast path, the shared segment passed over
`SCM_RIGHTS`. Windows is reached through the process's own DLLs with
`syscall.NewLazyDLL` and a `syscall.NewCallback` window procedure; macOS
through the Objective-C runtime with purego.

The platform-independent part of each back-end — event mapping, coordinate
maths, pixel packing, damage rectangles — lives in a codec covered at 100%,
on every operating system. The thin platform glue is proven live: see
[Status]({{< relref "/status.md" >}}).

## Only what changed

A root that implements `DamageRenderer` (as `toolkit/scene.HostRoot` does)
reports the rectangles it repainted, and every back-end presents only those:
X11 through MIT-SHM `ShmPutImage`, Wayland through `wl_shm` damage, GTK4
through a partial texture update, Cocoa, Win32, Android, wasmbox and the browser
tab. The first frame, a resize and an X11 `Expose` still present the whole
surface.

On GTK4 this needs libgtk-4 4.16 or newer: the frame becomes a
`GdkMemoryTextureBuilder` texture with `set_update_texture` and
`set_update_region`, so GTK re-renders and re-uploads only the damaged region.
An older runtime keeps the whole upload, chosen at run time. The framebuffer
lives in GLib memory that the texture shares, so a present copies no byte on the
CPU side (it was the whole frame, 128000 bytes for 200x160, every time). The
GTK4 window follows resizes: the picture fills the window, the framebuffer is
reallocated, one whole frame is presented at the new size, and damage resumes.

## On the GPU

On macOS, `Config{GPU: true}` draws frames on the GPU.
- **How:** the widgets are recorded through
  [gpupainter](https://github.com/go-widgets/gpupainter) into a
  [go-gfx/gpu2d](https://github.com/go-gfx/gpu2d) list. Its Metal renderer draws
  the frame and presents it in a `CAMetalLayer`, without the frame returning to
  the CPU.
- **Exactness:** the pixels are the CPU path's, byte for byte.
- **Cost:** a whole frame costs the processor 0.59 ms instead of 1.58 ms at
  960 × 640, and 0.99 ms instead of 5.96 ms at 1920 × 1200 (Apple M4 Max).
- **Fallbacks:** a window with a translucent material stays on the CPU path,
  and so does a machine where Metal will not start. Neither is an error.
  On Linux, see below; the other back-ends ignore the option.

On Linux (X11 and Wayland), `Config{GPU: true}` draws frames with Vulkan.
- **How:** the same recording, drawn by go-gfx/gpu2d's Vulkan renderer. Its
  pixels are copied into the window's framebuffer, which is presented as ever:
  MIT-SHM (or `PutImage`) on X11, `wl_shm` on Wayland. The frame crosses the CPU
  once; nothing is shared with the display server by dma-buf yet.
- **Exactness:** on software Vulkan (lavapipe) the pixels are the CPU path's,
  byte for byte, proven live on both protocols: the window read back from the X
  server, and the `wl_shm` buffer the window committed, equal the CPU path's.
  go-gfx/gpu2d measured its Vulkan renderer on a hardware GPU (AMD Radeon PRO
  W7700, RADV): `Draw` is 52 times faster than the CPU `Rasterize` on one core at 1080p; the
  window itself, with its copy into the framebuffer, has not been measured on
  hardware.
- **Fallback:** where Vulkan is unavailable, or a frame is refused (too large
  for the device, device lost), the window draws on the CPU and keeps running.
  `GPUError()` on the back-end says why, and every later frame is a CPU frame.
- **Cost (software only):** lavapipe is a CPU rasterizer, so the GPU path costs
  about ten times the CPU time of the CPU path: 25.8 ms instead of 2.48 ms per
  frame at 960 × 640, and 89.7 ms instead of 8.46 ms at 1920 × 1200 (X11,
  lavapipe on 16 threads in an arm64 container on an Apple M4 Max). Whether it
  pays on a real GPU is not measured yet.

## HiDPI

The default is one framebuffer pixel per logical point: the UI is laid out
and painted at a readable size, and the compositor up-samples it. On Windows
the window declares per-monitor DPI awareness and the OS scales the logical
frame to the physical client area. `RenderScale` asks for a framebuffer at
the panel's real resolution, which is correct only for a root that lays out
in device pixels.

## Screens, and showing and hiding

`window.Screens()` lists the attached displays, primary first, in logical
points with the desktop's panels excluded, and may be called before `Open`.
A screen's name is the panel's own (`"DELL U2720Q"`, from its EDID), falling
back to the connector (`"HDMI-1"`) when it publishes none — and also when two
attached panels publish the same one, since a name that cannot tell them
apart is not a name.

`window.Show`, `Hide` and `Raise` take an open window off the screen without
closing it and bring it back — what a [tray application]({{< relref "/surfaces/application.md#the-tray" >}})
needs.

| Back-end | Show | Hide | Raise |
|---|---|---|---|
| X11 | `MapWindow` | ICCCM withdraw, so it leaves the taskbar | map, raise, `_NET_ACTIVE_WINDOW` |
| Wayland | commit again | null buffer and commit | `ErrNotSupported`: xdg-shell has no such request |
| macOS | `orderFront:` | `orderOut:` | activate the app, `makeKeyAndOrderFront:` |
| Windows | `SW_SHOWNA` | `SW_HIDE` | `SW_RESTORE` and `SetForegroundWindow` |
| GTK4 | `gtk_widget_set_visible(true)` | `gtk_widget_set_visible(false)` | `gtk_window_present` |
| Android, wasmbox, browser tab | `ErrNotSupported` | `ErrNotSupported` | `ErrNotSupported` |

Focus is the platform's to grant: under focus-stealing prevention, `Raise`
may only mark the window as wanting attention, and cannot tell (a Wayland
compositor under GTK4 does exactly that).

`Config.HideOnClose` is honoured by X11, Wayland, GTK4, macOS and Windows: the
window's own close control then hides the window instead of closing it, and
`Show` or `Raise` brings it back.

## Context menu and picture on native controls

`NativeControl.Menu` and `Image` / `ImageOnly` are applied by every native
back-end, not only Cocoa. On GTK4 the menu is a popover opened by a secondary
click and the picture sits in the button, alone or beside the caption. On Win32
the menu is a real `HMENU` shown with `TrackPopupMenu` on `WM_CONTEXTMENU`, and
the picture is a bitmap on the button. A disabled entry or a separator is not
pickable. Both are reapplied only when they changed, since the descriptors are
rebuilt every frame.

The PNG is bounded: at most 1 MiB and 1024 pixels a side, checked from the
header before any pixel is allocated, and a picture that cannot be read leaves
the button as it was. It is fitted inside the button less a 4-pixel margin by
`painter.IconSize` and `painter.IconBGRAOver`, never scaled up. On Windows
the image beside the caption needs the visual-styles common controls; without a
manifest the button shows the picture alone. Windows edit fields keep their own
system menu.

## Native controls on Windows

A `toolkit.NativeControl` becomes a real child window of the Win32 window, one
per `Key`, reconciled every frame: a value is pushed into a control only when it
differs from what the control last reported, so a person's own edits are never
fought. Besides the text, button, slider, progress and stepper kinds, five more
are hosted:

| Kind | Window class | Value |
|---|---|---|
| `NativeList` | `LISTBOX` | `Number` is the chosen row, `-1` for none |
| `NativeLink` | `SysLink` | `OnActivate` fires on a click or Return |
| `NativeDate` | `SysDateTimePick32` | `Text` is `YYYY-MM-DD` |
| `NativeColor` | `GoWidgetsColorWell` | `Text` is `#RRGGBB`; a click opens the system colour dialog |
| `NativeSegmented` | `SysTabControl32` | `Text` is the selected segment's title |

The rows of a list and the segments are fixed when the control is created;
change them by changing the `Key`. A link never opens anything itself: it only
calls `OnActivate`, so which addresses are followed (http and https only) stays
the decision of `application.OpenURL`. `SysLink` exists only in the visual-styles
common controls, which a program without a manifest does not get; the window
creates the link under a manifest of its own and, should that be unavailable,
shows a flat push button that activates the same way.

## Native tables, trees and tabs

Three more kinds carry structured data. Build them with
`NewNativeTable(columns, rows, selected)`, `NewNativeTree(nodes, selectedPath)`
and `NewNativeTabs(titles, selected)`; the `NativeControl` gained `Columns`,
`Rows`, `Tree` and `Revision` for them, and `Native.SetRows` replaces the rows
of a table afterwards.

| Kind | Cocoa | GTK4 | Win32 | Selection |
|---|---|---|---|---|
| `NativeTable` | `NSTableView` | `GtkColumnView` | `SysListView32` (`LVS_REPORT`, `LVS_OWNERDATA`) | `Number` / `OnNumber` is the row, `-1` for none |
| `NativeTree` | `NSOutlineView` | `GtkListView` over a `GtkTreeListModel` | `SysTreeView32` | `Text` / `OnText` is a path such as `2.0.1` |
| `NativeTabs` | `NSTabView` | `GtkNotebook` | `SysTabControl32` | `Number` / `OnNumber` is the tab, `-1` for none |

Sorting is not implemented: rows appear in the order given. A tab control is
only the tab strip; the content under a tab is laid out by the toolkit as
usual. The painted fallbacks (`Table`, `TreeView` with `HideRoot`, `TabBar`)
draw the same data where no native control is hosted. The window back-ends that
host these classes (hosted by `go-widgets/window`) arrive in the next window
release; the Cocoa side needs the new `GridView`, `OutlineView` and `TabView`
kinds of `go-macos/appkit`.

The data is bounded: at most `MaxNativeRows` (100000) rows, `MaxNativeColumns`
(64) columns, `MaxNativeTreeDepth` (32) levels, `MaxNativeTreeNodes` (100000)
nodes and `MaxNativeTabs` (64) tabs; a larger value is refused, not truncated.

Reconciling is change-only. The frame is keyed on `Native.Revision()`: while the
revision is unchanged the rows are not walked at all, and the rows are pushed to
the control only when it moves. On Cocoa a frame with 10,000 rows costs 160 us
with an unchanged revision and 687 us when the rows change (including building
the 10,000 rows); `SetGrid` itself takes 1.3 us because the data source is
virtual. On Win32 an unchanged revision costs 0.26 ns and no allocation, against
43.8 us to walk 10,000 rows (measured on an M4 Max for Cocoa, one core).
