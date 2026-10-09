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
