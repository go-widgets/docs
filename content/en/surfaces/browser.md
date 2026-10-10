---
title: "A browser tab"
linkTitle: "Browser"
weight: 30
description: "Run a widget tree in a plain <canvas> with go-widgets/webcanvas, or as a window of the wasmdesk compositor."
tags: [surfaces, wasm, browser, webcanvas, wasmdesk]
---

The toolkit is pure Go with no DOM dependency, so `GOOS=js GOARCH=wasm` builds
it like any other target. Two ways to put the result in front of somebody:

## A plain canvas

[`go-widgets/webcanvas`](https://github.com/go-widgets/webcanvas) blits an RGBA
framebuffer into a `<canvas>` and routes DOM pointer and keyboard events back
into the scene. No compositor, no `SharedArrayBuffer`, no cross-origin
isolation headers: any static host serves it.

```go
//go:build js && wasm

package main

import "github.com/go-widgets/webcanvas"

func main() { webcanvas.Run("screen", myScene()) }
```

The scene implements `App`: `Size`, `Draw(buf []byte)`, and one method per
kind of event, each reporting whether anything changed so an unchanged frame
is not repainted. It may also implement `Ticker`, `Animator`, `Resizer` or
`Scroller`.

The [gallery](https://go-widgets.github.io/gallery/) and
[`app-template`]({{< relref "/getting-started.md" >}}) run this way.

Or let `window` choose. Since window v0.87.0,
[`window.Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}})
built for `js/wasm` on an ordinary page returns a back-end that draws into the
page's `<canvas>` (`Config.Canvas`, default `"screen"`) through `webcanvas`. An
application written against `window` then runs unchanged as a native window, in
a browser tab, and in wasmdesk.

## A window on the wasmdesk desktop

Inside a wasmdesk worker, [`window.Open`]({{< relref "/surfaces/native-window.md" >}})
returns a client of the [wasmdesk/wasmbox](https://github.com/wasmdesk/wasmbox)
browser compositor. It allocates its surface in a `SharedArrayBuffer`, says
`hello` over its `MessagePort`, waits for `welcome`, paints into the shared
buffer and posts `commit` — the whole surface, or only the damaged rectangles.
The compositor's `input` messages become `toolkit.Event`, as on X11.

So an application written against `window` runs unchanged as a native window
and as a window of a desktop in a browser tab. The
[desktop shell demo](https://github.com/go-widgets/desktop) does exactly
that, from one source.

The protocol codec is covered at 100% on every platform. The live proof runs
headless Chromium against the real wasmdesk compositor: it spawns the client,
reads the compositor's own composited pixels, clicks, and checks that the
click reached the widget.
