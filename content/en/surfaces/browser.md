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
is not repainted. It may also implement `Ticker`, `Animator`, `Resizer`,
`Scroller` or `ScrollConsumer`.

### Several canvases on one page

`Run` never returns and serves one canvas. `webcanvas.Mount(canvas, scene)`
puts a scene on any `<canvas>` element, returns a `*Surface` at once, and can
be called as often as the page needs (an editor in each cell of a notebook,
say). `Run` is `Mount`, then `Focus`, then a park.

- **Keys** go to the surface that has focus and to no other. The canvas gets
  `tabindex="0"` unless the page set one, so a click or Tab focuses it, and
  `Surface.Focus()` does the same from code. (Before webcanvas v0.10.0 the key
  listeners were on `window`, and every canvas received every key.)
- **The wheel** goes to the surface under the pointer. A scene that implements
  `ScrollConsumer`, `Scroll(x, y, dx, dy int) (repaint, consumed bool)`,
  returns `consumed == false` when it did not use the event, and then the page
  scrolls under the canvas. A `Scroller` always takes the wheel, as before.
- **`Surface.Close()`** removes every listener, timer and animation frame the
  surface installed.

The [gallery](https://go-widgets.github.io/gallery/) and
[`app-template`]({{< relref "/getting-started.md" >}}) run this way.

### Accessibility: the mirror

To a screen reader a `<canvas>` is one opaque image. A scene that also
implements `webcanvas.A11yAware`, `A11yNodes() []webcanvas.A11yNode`, gets an
**accessibility mirror** (webcanvas v0.11.0): a DOM subtree beside the canvas
that states in ARIA what the scene paints. A scene that does not implement it
is unchanged.

```go
func (s *scene) A11yNodes() []webcanvas.A11yNode {
	return []webcanvas.A11yNode{
		{Key: "Button#0", Role: "button", Name: "Save", X: 10, Y: 20, W: 80, H: 30},
		{Key: "@status", Role: "status", Name: s.message, X: 10, Y: 60, W: 200, H: 16},
	}
}
```

- **Nodes** are listed in reading order, in surface pixels, with a `Key` that
  stays the same for the same widget from frame to frame (the toolkit's
  `A11yNode.Key`). The mirror diffs by `Key`, so an unchanged widget keeps the
  same DOM element and a screen reader's cursor is not reset on every frame.
- **Roles** include button (a native `<button>`), checkbox, switch and radio,
  slider, progressbar, meter and spinbutton, text, textbox, status, alert and
  log. Any other WAI-ARIA role name is passed through.
- **Hidden from the eye, not from assistive technology.** The mirror is clipped
  (`clip-path: inset(50%)`), never `display: none`, `visibility: hidden` or
  `aria-hidden`, which would remove it from the accessibility tree. It ignores
  the pointer, so the mouse still reaches the canvas.
- **Focus and keys.** The element of the focused widget holds DOM focus, and
  keys are heard there.
- **Activation.** A click on a mirror element, which is what a screen reader's
  "activate" sends, is replayed as a press and release at the widget's centre.
- **Announcements.** Text that appears or changes in a `status` or `log` node is
  announced from one persistent polite live region, and `alert` text from an
  assertive one.
- **Cost.** A changed tree reaches the DOM at most every 200 ms, and a change
  held back is applied later, never lost. An unchanged tree costs a signature.
  In headless Chrome, 2,000 nodes take about 70 ms to build and about 6 ms to
  update when one node changes (indicative, measured on a busy machine).
- **Untrusted text.** Names and values are set only as text and attribute
  values, never as markup, so a notebook from anyone cannot inject HTML through
  a widget name.

`Surface.Refresh()` draws a frame and updates the mirror, for a host that
changed the scene from outside.

A scene that also implements `webcanvas.TextInputHost`,
`TextInputFor(key string) textinput.Client`, gives a text widget a **text
layer**: its `textbox` is then one element that holds the text, so the browser's
find, selection and assistive technology see it, and that takes the keys and
IME input for the widget.

### From a toolkit tree

You do not have to convert the toolkit's accessibility tree yourself.
`github.com/go-widgets/window/webmirror` does it once: `webmirror.Host` implements
`A11yAware` and `TextInputHost` over one or several widget trees (give each tree
its own `Prefix` so keys cannot collide), and `webmirror.Convert` is the single
node conversion. A `TextView` or `CodeEditor` gets its text layer through its
`textinput.Client`. A nameless field is still published, as on the native
bridges. The browser-tab back-end below and the
[gallery](https://go-widgets.github.io/gallery/) both use it, so both are
readable by a screen reader.

Or let `window` choose. Since window v0.87.0,
[`window.Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}})
built for `js/wasm` on an ordinary page returns a back-end that draws into the
page's `<canvas>` (`Config.Canvas`, default `"screen"`) through `webcanvas`. An
application written against `window` then runs unchanged as a native window, in
a browser tab, and in wasmdesk. From window v0.93.0 the tab publishes the
accessibility mirror and text layer from the widget tree, with no extra code.

## Repainting only what changed

Without help, every frame is `Draw` of the whole scene, a copy of the whole
framebuffer into the browser and a `putImageData` of the whole canvas, even when
one caret blinked or only a guest's frame in a `Layer` is new. A scene that
knows what it changed implements `DrawDamaged(buf []byte) []Rect`
(`webcanvas.DamageDrawer`, webcanvas v0.15.0), and the host copies and puts only
those rectangles.

- **The contract** is the one [`window`'s `DamageRenderer`]({{< relref "/surfaces/native-window.md" >}})
  gives a desktop host, in the browser's terms: the scene repaints what changed
  into a framebuffer that persists between frames and returns the rectangles it
  repainted (`webcanvas.Rect` is `toolkit.Rect`). An **empty** result means
  nothing changed and nothing is put. `Draw` still runs for the first frame and
  after every resize.
- **Layers** are drawn again every frame, but cost no scene repaint. A layer
  that moved, shrank or went has its old rectangle put back from the scene
  buffer; a layer with `Alpha < 1`, or not marked `Opaque`, has the scene put
  back under it every frame. `Opaque: true` promises every pixel is opaque, and
  is what lets a still guest frame put nothing.
- **Not using it** changes nothing: a scene without `DrawDamaged` is drawn and
  put whole, as before. The `webcanvas/gpu` presenter has its own version of
  it, below.
- **Measured** under `GOOS=js` in node at 1280×800, with a fake 2D context: a
  frame that changes one 120×40 widget costs 0.031 ms instead of 2.89 ms, and a
  frame with no damage and one opaque layer 0.021 ms instead of 2.90 ms.

## On the GPU

`webcanvas.RunWith(id, app, presenter)` takes a `Presenter`, and the package
`webcanvas/gpu` provides one that draws frames with WebGPU.
- **How:** an App that also implements `Paint(painter.Painter)` is recorded
  through [gpupainter](https://github.com/go-widgets/gpupainter) into a
  [go-gfx/gpu2d](https://github.com/go-gfx/gpu2d) list. Its WebGPU renderer
  draws the frame, which is copied into the canvas texture on the GPU.
- **Why a presenter:** a canvas takes one context kind for life, so the
  presenter decides once, before the first frame.
- **Exactness:** the pixels are the CPU `Draw` frame's, byte for byte (a test
  holds it).
- **Damage:** an App that also implements `PaintDamaged(painter.Painter) []Rect`
  (`gpu.DamagePainted`, webcanvas v0.16.0 and v0.17.0) paints into the painter
  only what changed, confined to the rectangles it returns, as toolkit's
  `RenderDamaged` does. An **empty** result means the frame is neither recorded
  nor presented: an idle frame sends no command buffer to the GPU (2 before, 0
  after, counted in Chrome, 0 allocations) and its recording costs a median
  11.4 ns instead of 122.9 µs (Apple M4 Max, one core, 5 rounds at load below
  3, against recording the whole scene). A frame with damage is recorded as
  `PaintDamaged` painted it, so the scene must repaint the ground under the
  damage: with one cell of 1,200 damaged, the list handed to the renderer goes
  from 2,401 instances to 2, and recording it costs a median 129.9 ns instead
  of 133.2 µs (same machine and rounds). The first frame, a resize and an App without
  `PaintDamaged` are recorded whole.
- **The canvas texture does not persist:** after each presentation the WebGPU
  spec gives `getCurrentTexture()` a transparent black texture (Chrome does the
  same), so the renderer's own target keeps the rest of the frame and each
  presented frame still copies all of it to the canvas, on the GPU.
- **Fallbacks:** a browser without WebGPU, or an App without `Paint`, stays on
  the 2D canvas. Neither is an error.

The [web gallery](https://go-widgets.github.io/gallery/) draws this way.

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
