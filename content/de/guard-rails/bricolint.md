---
title: "bricolint: keine handgezeichnete UI"
linkTitle: "bricolint"
weight: 20
description: "Lässt einen Pull Request scheitern, der im Anwendungscode Bedienelemente mit Painter-Primitiven zeichnet oder ein Widget in jedem Frame neu baut."
tags: [ci, lint, painter]
---

*Bricolage* – improvisiertes, selbstgestricktes Zeichnen – ist der Weg, auf dem eine Oberfläche still und leise
ihr Feedback für Drücken, Hover und Fokus verliert, ihr Theming, ihre HiDPI-Skalierung
und ihre Barrierefreiheit. [`go-widgets/bricolint`](https://github.com/go-widgets/bricolint)
verhindert, dass eine Anwendung, die auf das Toolkit umgestiegen ist, wieder zurückdriftet.

```go
p.FillRect(bar, theme.Bg)              // flagged: hand-drawn chrome
toolkit.NewBackdrop(theme.Bg).Draw(p)  // a toolkit widget

func (v *view) Draw(p painter.Painter) {
	b := toolkit.NewButton("ok")       // flagged: a new widget every frame
	b.Draw(p)
}
```

1. **Ein Painter-Primitiv im Anwendungscode**: eine Zeichenmethode (`FillRect`,
   `StrokeRect`, `FillPath`, `DrawImage`, `Text`, `PutPixel`, …), aufgerufen auf einem
   Empfänger, dessen statischer Typ ein Painter-Typ ist. Die Oberfläche abzufragen oder zu beschneiden
   (`Size`, `PushClip`, `PushTranslate`) ist kein Zeichnen und wird nicht
   gemeldet. `-primitives=` ersetzt die Liste.
2. **Ein Widget, das in jedem Frame weggeworfen wird**: ein `toolkit.New…`-Aufruf in einer Methode
   namens `Draw`, `Paint` oder `Render`. Ein bei jedem Zeichnen neu gebautes Widget behält keinen
   Interaktionszustand. Bauen Sie es einmal, halten Sie es in einem Feld, steuern Sie es über eine
   Bindung. `-checkthrow=false` schaltet das ab.

Mancher Code ist ein echtes Blatt – ein Spiel-Framebuffer, ein Painter-Back-End, die
Interna des Toolkits selbst – und sagt das explizit:

```go
p.FillRect(bg, c) //bricolint:allow engine SVG raster blit — a genuine leaf
```

```go
//bricolint:allowfile painter back-end — this file IS the leaf
package pdfsurface
```

**Die Begründung ist Pflicht**: Eine Direktive ohne Begründung wird ignoriert, und der
Befund schlägt weiter an, bis jemand die Rechtfertigung aufschreibt.

```yaml
jobs:
  bricolint:
    uses: go-widgets/bricolint/.github/workflows/bricolint.yml@main
```
