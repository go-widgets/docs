---
title: "Wie ein Widget auf den Bildschirm kommt"
linkTitle: "Rendering"
weight: 10
description: "Ein Widget zeichnet über einen Painter; der Painter entscheidet, ob daraus Pixel oder Terminalzellen werden; ein Back-End stellt das Ergebnis dar."
tags: [rendering, painter, toolkit]
---

Drei Schichten, und jede kennt nur die unter ihr:

```text
toolkit widget tree          Button, Table, Agenda, VBox ...  (or a skin.Object)
        │  Draw(p painter.Painter, theme *Theme)
        ▼
painter.Painter              FillRect, StrokeRect, Text, PutPixel, ...
        │
        ├─ PixelPainter  → an RGBA []byte   → window, browser canvas, PNG, Android
        └─ CellPainter   → a grid of cells  → a terminal, as 24-bit ANSI
```

Ein Widget erfährt nie, welchen Painter es bekommen hat. Ein Back-End erfährt nie,
welche Widgets gezeichnet haben. Genau deshalb kann ein Widget-Baum in einem
nativen Fenster, einem Browser und einem Terminal ohne eine Zeile bedingten Codes angezeigt werden.

| | |
|---|---|
| [Die Painter-Schnittstelle]({{< relref "/rendering/painter.md" >}}) | die Primitive, die optionalen Fähigkeiten, die beiden Painter |
| [Das Toolkit]({{< relref "/rendering/toolkit.md" >}}) | der Widget-Satz, Themes, Layouts, Text und Barrierefreiheit |
| [Skins]({{< relref "/rendering/skin.md" >}}) | die Teile und Zustände eines Widgets als Daten statt in Go beschreiben |
