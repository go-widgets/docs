---
title: "bricolint: no hand-drawn UI"
linkTitle: "bricolint"
weight: 20
description: "Fails a pull request that paints chrome with painter primitives in application code, or rebuilds a widget on every frame."
tags: [ci, lint, painter]
---

*Bricolage* — improvised, hand-rolled drawing — is how an interface quietly
loses its pressed, hover and focus feedback, its theming, its HiDPI scaling
and its accessibility. [`go-widgets/bricolint`](https://github.com/go-widgets/bricolint)
keeps an application that has moved onto the toolkit from drifting back.

```go
p.FillRect(bar, theme.Bg)              // flagged: hand-drawn chrome
toolkit.NewBackdrop(theme.Bg).Draw(p)  // a toolkit widget

func (v *view) Draw(p painter.Painter) {
	b := toolkit.NewButton("ok")       // flagged: a new widget every frame
	b.Draw(p)
}
```

1. **A painter primitive in application code**: a drawing method (`FillRect`,
   `StrokeRect`, `FillPath`, `DrawImage`, `Text`, `PutPixel`, …) called on a
   receiver whose static type is a painter type. Querying or clipping the
   surface (`Size`, `PushClip`, `PushTranslate`) is not drawing and is not
   flagged. `-primitives=` replaces the list.
2. **A widget thrown away every frame**: a `toolkit.New…` call inside a method
   named `Draw`, `Paint` or `Render`. A widget rebuilt on each paint keeps no
   interaction state. Build it once, keep it in a field, drive it through a
   binding. `-checkthrow=false` turns this off.

Some code is a genuine leaf — a game framebuffer, a painter back-end, the
toolkit's own internals — and says so explicitly:

```go
p.FillRect(bg, c) //bricolint:allow engine SVG raster blit — a genuine leaf
```

```go
//bricolint:allowfile painter back-end — this file IS the leaf
package pdfsurface
```

**The reason is mandatory**: a directive without one is ignored, and the
finding keeps firing until somebody writes the justification down.

```yaml
jobs:
  bricolint:
    uses: go-widgets/bricolint/.github/workflows/bricolint.yml@main
```
