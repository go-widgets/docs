---
title: "Die Painter-Schnittstelle"
linkTitle: "Painter"
weight: 10
description: "Die Primitive, mit denen jedes Widget gezeichnet wird, die optionalen Fähigkeiten, die ein Painter hinzufügen kann, und die Pixel- und Zellen-Painter."
tags: [rendering, painter]
---

[`go-widgets/painter`](https://github.com/go-widgets/painter) definiert die eine
Schnittstelle, über die jedes Widget zeichnet:

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

Der Satz ist absichtlich klein: Jedes Back-End muss ihn vollständig implementieren, und ein
Terminalraster kann keine Ecke abrunden und keine Strichbreite variieren. Deshalb legt der Vertrag
fest, was geschieht, wenn es das nicht kann – die Breite bei `StrokeRect` ist ein Hinweis, den ein Zellenraster
ignoriert, und `FillRoundRect` fällt dort auf eine eckige Füllung zurück.

## Optionale Fähigkeiten {#optional-capabilities}

Was nicht jede Oberfläche kann, steht nicht in der Basisschnittstelle. Ein Widget, das
es braucht, macht eine Typzusicherung und zeichnet etwas Sinnvolles, wenn die Antwort Nein lautet:

| Schnittstelle | Methoden | Wofür |
|---|---|---|
| `Clipper` | `PushClip`, `PopClip` | ein gescrolltes Kind innerhalb der Grenzen seines Elternelements halten |
| `Translator` | `PushTranslate`, `PopTranslate` | ein Kind in seinen eigenen Koordinaten zeichnen |
| `PathPainter` | `FillPath`, `StrokePath` | Kurven, Diagramme, kantengeglättete Formen |
| `ImagePainter` | `DrawImage` | Icons, Vorschaubilder, Bilder |
| `MaskPainter` | `DrawMask` | Glyphen und andere Deckungsmasken in einer Farbe |
| `FacePainter` | `TextFace` | Text in einer bestimmten Schrift und Größe |

```go
if c, ok := p.(painter.Clipper); ok {
	c.PushClip(bounds)
	defer c.PopClip()
}
```

## Die beiden Painter {#the-two-painters}

| | Schreibt in | Dargestellt von |
|---|---|---|
| `NewPixelPainter(buf, w, h)` | einen RGBA-`[]byte`, der dem Aufrufer gehört (`NewPixelPainterBGRA` für BGRA-Oberflächen) | einem nativen Fenster, einem Browser-Canvas, einer Android-Oberfläche, einem PNG oder SVG |
| `NewCellPainter(w, h)` | ein Zellenraster (Rune, Vordergrund, Hintergrund) mit einem 24-Bit-ANSI-Serialisierer | einem Terminal, über [`tui`]({{< relref "/surfaces/terminal.md" >}}) |

Der Puffer bleibt im Besitz des Aufrufers. Ein `PixelPainter` übersetzt nur Primitive
in Schreibzugriffe, sodass derselbe `[]byte` an ein Fenster-Back-End, ein
`<canvas>` oder einen Bild-Encoder übergeben werden kann, ohne dazwischen zu kopieren.

## Ausprobieren {#trying-it}

Die Demos des Repositorys zeichnen dieselben drei Widgets über beide Painter:

```sh
go run ./cmd/wui-demo --out demo.png     # pixels, written as a PNG
go run ./cmd/tui-demo --theme dark       # cells, written as ANSI to stdout
```

und [go-widgets.github.io/painter](https://go-widgets.github.io/painter/) zeigt
den Pixel-Painter in einem Browser.
