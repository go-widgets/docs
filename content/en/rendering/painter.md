---
title: "The painter seam"
linkTitle: "Painter"
weight: 10
description: "The primitives every widget is drawn with, the optional capabilities a painter may add, and the pixel and cell painters."
tags: [rendering, painter]
---

[`go-widgets/painter`](https://github.com/go-widgets/painter) defines the one
interface every widget draws through:

```go
type Painter interface {
	FillRect(r Rect, c RGBA)
	StrokeRect(r Rect, c RGBA, lineW int)
	FillRoundRect(r Rect, radius int, c RGBA)
	StrokeRoundRect(r Rect, radius int, c RGBA, lineW int)
	PutPixel(x, y int, c RGBA)
	Text(x, y int, s string, ink RGBA)
	Size() (w, h int)
}
```

The set is small on purpose: every back-end must implement all of it, and a
terminal grid cannot round a corner or vary a stroke width. So the contract
says what happens when it cannot — `StrokeRect`'s width is a hint a cell grid
ignores, and `FillRoundRect` falls back to a square fill there.

## Optional capabilities

What not every surface can do is not in the base interface. A widget that
needs it type-asserts, and draws something sensible when the answer is no:

| Interface | Methods | Used for |
|---|---|---|
| `Clipper` | `PushClip`, `PopClip` | keeping a scrolled child inside its parent's bounds |
| `Translator` | `PushTranslate`, `PopTranslate` | drawing a child in its own coordinates |
| `PathPainter` | `FillPath`, `StrokePath` | curves, charts, anti-aliased shapes |
| `ImagePainter` | `DrawImage` | icons, thumbnails, pictures |
| `MaskPainter` | `DrawMask` | glyphs and other coverage masks in one colour |
| `FacePainter` | `TextFace` | text in a specific font face and size |

```go
if c, ok := p.(painter.Clipper); ok {
	c.PushClip(bounds)
	defer c.PopClip()
}
```

## The two painters

| | Writes into | Presented by |
|---|---|---|
| `NewPixelPainter(buf, w, h)` | a caller-owned RGBA `[]byte` (`NewPixelPainterBGRA` for BGRA surfaces) | a native window, a browser canvas, an Android surface, a PNG or SVG |
| `NewCellPainter(w, h)` | a grid of cells (rune, foreground, background) with a 24-bit ANSI serialiser | a terminal, through [`tui`]({{< relref "/surfaces/terminal.md" >}}) |

The buffer stays the caller's. A `PixelPainter` only translates primitives
into writes, so the same `[]byte` can be handed to a window back-end, a
`<canvas>` or an image encoder without a copy in between.

## Trying it

The repository's own demos draw the same three widgets through both painters:

```sh
go run ./cmd/wui-demo --out demo.png     # pixels, written as a PNG
go run ./cmd/tui-demo --theme dark       # cells, written as ANSI to stdout
```

and [go-widgets.github.io/painter](https://go-widgets.github.io/painter/) shows
the pixel painter in a browser.
