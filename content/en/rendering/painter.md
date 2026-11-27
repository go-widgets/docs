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

## Recording to SVG

`svgrec.Recorder` (package `github.com/go-widgets/painter/svgrec`) is a third
painter: it turns the same calls into a **native SVG document**, `<rect>`,
`<path>` and `<clipPath>` elements, and an `<image>` only where a raster is
really drawn. The result is lossless and resizable, and deterministic: the same
calls give the same bytes.

```go
r, _ := svgrec.New(1000, 700)
widget.Draw(r, painter.LightTheme())
doc, err := r.SVG() // err is the first call that was refused, if any
```

It implements `Clipper`, `Translator`, `PathPainter`, `ImagePainter` and
`MaskPainter`. Where SVG has no faithful equivalent the mapping says so rather
than approximating silently:

| Call | SVG |
|---|---|
| `FillRect`, `PutPixel` | `<rect>` |
| `FillRoundRect` | `<rect rx>`; the corner is anti-aliased differently from the pixel painter's |
| `StrokeRect`, `StrokeRoundRect` | an `evenodd` frame, rings drawn inward as the pixel painter does |
| `Text` | glyph outlines (below) |
| `PushClip` / `PushTranslate` | `<clipPath>` plus a nested `<g>`; translation is applied to the coordinates |
| `FillPath`, `StrokePath` | `<path>`; a stroke width is in user units, so it scales with the viewBox, and a zero-length subpath paints a dot in SVG where the pixel painter paints nothing |
| `DrawImage`, `DrawMask` | `<image>` with a PNG data URI |

**Text becomes outlines, not `<text>`.** The built-in 5x7 font is a pixel grid,
so each string is one `<path>` of merged runs, wrapped in a `<g>` whose
`<title>` holds the text for assistive technology. An embedded subset font
would need a font writer and a viewer that honours it, and a viewer that does
not would fall back to another face and move the layout. The cost: the text is
not selectable. Text set in a TrueType face is not recorded as outlines yet;
`FacePainter` is not offered, so it arrives as a raster `DrawMask`.

Painter methods return no error, so a refused call (invalid UTF-8, a character
XML cannot carry, a coordinate beyond 1<<24, a call or document over its
`Limits`) is dropped and the first error is reported by `Err()` and `SVG()`.
The fuzz targets parse every output back with `encoding/xml` and fail on any
element or attribute the writer does not own.

Against the pixel painter, rasterised with the pure-Go `go-gfx/gfx/svg`, the
sample widgets, rectangles, bitmap text, clips and translation match to the
pixel; paths differ by at most 1 in a channel; anti-aliased rounded corners by
up to 61 in a channel on 7.5 % of the pixels of the worst scene.

## Trying it

The repository's own demos draw the same three widgets through both painters:

```sh
go run ./cmd/wui-demo --out demo.png     # pixels, written as a PNG
go run ./cmd/tui-demo --theme dark       # cells, written as ANSI to stdout
```

and [go-widgets.github.io/painter](https://go-widgets.github.io/painter/) shows
the pixel painter in a browser.
