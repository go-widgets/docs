---
title: "Snapshots: a widget as SVG or PNG"
linkTitle: "Snapshots"
weight: 60
description: "go-widgets/svg wraps any RGBA render in a self-contained SVG, bit-exact, or writes it as a PNG."
tags: [surfaces, svg, png]
---

[`go-widgets/svg`](https://github.com/go-widgets/svg) turns an RGBA buffer into
a self-contained SVG: a `viewBox` pinned to the render's pixel size, and the
pixels as a base-64 PNG under `image-rendering: pixelated`. Every pixel shown
is the pixel the widget drew — no re-vectorisation, no font substitution — and
it stays sharp at any zoom.

```go
// any RGBA producer
svg.Snapshot(f, surface, w, h, "my image")

// a toolkit widget
btn := toolkit.NewButton("Click me", nil)
btn.SetBounds(toolkit.Rect{W: 200, H: 40})
widget.Snapshot(f, btn, 200, 40, toolkit.DefaultLight(), "widget: button")
widget.PNG(p, btn, 200, 40, toolkit.DefaultLight())
```

The label becomes the SVG's `<title>` and `aria-label`, escaped. The root
package depends on nothing; the `widget` subpackage is separate so a consumer
with its own pixel producer does not pull the toolkit into its module graph.

`svg/cmd/gallery-render` renders the widget catalogue, light and dark:
[go-widgets.github.io/svg](https://go-widgets.github.io/svg/).

## Animated SVG

`svg.Animate` stitches a timed list of SVG frames (for example from the
painter's [`svgrec`](../../rendering/painter/) recorder) into one animated SVG.
Every subtree that does not change between frames is stored once in `<defs>`
and reused with `<use>`; adjacent identical frames merge, and the timing is
pure CSS (`@keyframes`, no script), stopped by `prefers-reduced-motion`.

```go
out, err := svg.Animate([]svg.Frame{
    {At: 0, SVG: first},
    {At: 100 * time.Millisecond, SVG: second},
}, svg.Options{End: time.Second, Title: "demo"})
```

On 60 frames of 1000×700, a screen with one live region goes from 1.49 MB of
frames to 65 KB (22.9× smaller); a static screen to 24 KB (60×). Input is
parsed with an allowlist and bounded (frames, bytes, duration, size), and every
refusal is a typed error. `Document.Video` drives a rasteriser and a video
encoder you supply through two small interfaces; no WebM encoder is bundled.
