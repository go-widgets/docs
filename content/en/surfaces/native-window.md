---
title: "A native window"
linkTitle: "Native window"
weight: 10
description: "go-widgets/window: one Open/Run API over X11, Wayland, Cocoa, Win32, GTK4, Android and wasmbox, all with CGO_ENABLED=0."
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
| `js/wasm` | wasmbox | a client of the [wasmdesk compositor]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}}) |
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
reports the rectangles it repainted, and the Cocoa, Win32 and wasmbox
back-ends present only those.

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
| GTK, Android, wasmbox | `ErrNotSupported` | `ErrNotSupported` | `ErrNotSupported` |

Focus is the platform's to grant: under focus-stealing prevention, `Raise`
may only mark the window as wanting attention, and cannot tell.
