---
title: "Where it paints"
linkTitle: "Surfaces"
weight: 30
description: "Every surface a go-widgets tree can be shown on, and what chooses between them."
tags: [surfaces, back-ends]
---

| Surface | Module | Painter |
|---|---|---|
| [A native window]({{< relref "/surfaces/native-window.md" >}}) on X11, Wayland, macOS, Windows | `window` | pixels |
| [An application]({{< relref "/surfaces/application.md" >}}) with a tray icon and the system appearance | `application`, `tray` | pixels |
| [A browser tab]({{< relref "/surfaces/browser.md" >}}): a plain `<canvas>` | `webcanvas`, or `window` | pixels |
| [A browser desktop]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}}): a window of the wasmdesk compositor | `window` (wasmbox) | pixels |
| [A terminal]({{< relref "/surfaces/terminal.md" >}}) | `tui` | cells |
| [An Android APK]({{< relref "/surfaces/android.md" >}}) | `android`, `window` | pixels |
| [A picture]({{< relref "/surfaces/snapshots.md" >}}): SVG or PNG | `svg` | pixels |

The widget tree does not change between them. What changes is who owns the
buffer and who delivers input — a display server, a browser, a terminal, a
Java host — and each back-end turns that input into the same `toolkit.Event`.

## Chosen by what is there

`window.Open` takes no back-end argument. It looks at the environment — the
platform it was built for, `$WAYLAND_DISPLAY`, `$DISPLAY`, the socket an
Android host exports — and picks the one that can work. A binary built for
Linux opens a Wayland window under a Wayland session and an X11 window under
X; the same Android binary dials its host inside an APK and opens an ordinary
Linux window under Termux. See
[how `Open` chooses]({{< relref "/surfaces/native-window.md#how-open-chooses" >}}).
