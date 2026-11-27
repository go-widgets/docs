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

## Als SVG aufzeichnen {#recording-to-svg}

`svgrec.Recorder` (Paket `github.com/go-widgets/painter/svgrec`) ist ein dritter
Painter: Er macht aus denselben Aufrufen ein **natives SVG-Dokument** aus den
Elementen `<rect>`, `<path>` und `<clipPath>` sowie einem `<image>` nur dort, wo
tatsächlich ein Raster gezeichnet wird. Das Ergebnis ist verlustfrei, skalierbar
und deterministisch: Dieselben Aufrufe ergeben dieselben Bytes.

```go
r, _ := svgrec.New(1000, 700)
widget.Draw(r, painter.LightTheme())
doc, err := r.SVG() // err is the first call that was refused, if any
```

Er implementiert `Clipper`, `Translator`, `PathPainter`, `ImagePainter` und
`MaskPainter`. Wo SVG kein getreues Gegenstück hat, sagt die Zuordnung das, statt
stillschweigend zu approximieren:

| Aufruf | SVG |
|---|---|
| `FillRect`, `PutPixel` | `<rect>` |
| `FillRoundRect` | `<rect rx>`; die Ecke wird anders geglättet als beim Pixel-Painter |
| `StrokeRect`, `StrokeRoundRect` | ein `evenodd`-Rahmen, Ringe nach innen gezeichnet wie beim Pixel-Painter |
| `Text` | Glyphenumrisse (siehe unten) |
| `PushClip` / `PushTranslate` | `<clipPath>` plus ein verschachteltes `<g>`; die Verschiebung wird auf die Koordinaten angewendet |
| `FillPath`, `StrokePath` | `<path>`; eine Strichbreite steht in Benutzereinheiten und skaliert daher mit der viewBox, und ein Teilpfad der Länge null malt in SVG einen Punkt, wo der Pixel-Painter nichts malt |
| `DrawImage`, `DrawMask` | `<image>` mit einer PNG-Daten-URI |

**Text wird zu Umrissen, nicht zu `<text>`.** Die eingebaute 5x7-Schrift ist ein
Pixelraster, daher wird jede Zeichenkette zu einem `<path>` aus zusammengefassten
Läufen, eingefasst in ein `<g>`, dessen `<title>` den Text für Hilfstechnologien
enthält. Eine eingebettete Teilmengen-Schrift bräuchte einen Font-Writer und einen
Betrachter, der sie beachtet; ein Betrachter, der das nicht tut, würde auf eine
andere Schrift ausweichen und das Layout verschieben. Der Preis: Der Text ist nicht
auswählbar. In einer TrueType-Schrift gesetzter Text wird noch nicht als Umriss
aufgezeichnet; `FacePainter` wird nicht angeboten, er kommt also als Raster-
`DrawMask` an.

Painter-Methoden geben keinen Fehler zurück, daher wird ein abgelehnter Aufruf
(ungültiges UTF-8, ein Zeichen, das XML nicht tragen kann, eine Koordinate jenseits
von 1<<24, ein Aufruf oder Dokument über seinen `Limits`) verworfen, und der erste
Fehler wird von `Err()` und `SVG()` gemeldet. Die Fuzz-Targets parsen jede Ausgabe
mit `encoding/xml` zurück und schlagen bei jedem Element oder Attribut fehl, das
der Writer nicht selbst erzeugt.

Gegenüber dem Pixel-Painter, gerastert mit dem reinen Go-Paket `go-gfx/gfx/svg`,
stimmen die Beispiel-Widgets, Rechtecke, Bitmap-Text, Clips und Verschiebung
pixelgenau überein; Pfade weichen um höchstens 1 in einem Kanal ab; geglättete
runde Ecken um bis zu 61 in einem Kanal bei 7,5 % der Pixel der schlechtesten
Szene.

## Ausprobieren {#trying-it}

Die Demos des Repositorys zeichnen dieselben drei Widgets über beide Painter:

```sh
go run ./cmd/wui-demo --out demo.png     # pixels, written as a PNG
go run ./cmd/tui-demo --theme dark       # cells, written as ANSI to stdout
```

und [go-widgets.github.io/painter](https://go-widgets.github.io/painter/) zeigt
den Pixel-Painter in einem Browser.
