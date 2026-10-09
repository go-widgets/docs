---
title: "How a widget reaches the screen"
linkTitle: "Rendering"
weight: 10
description: "A widget draws through a painter; the painter decides whether that becomes pixels or terminal cells; a back-end presents the result."
tags: [rendering, painter, toolkit]
---

Three layers, and each knows only the one below it:

```text
toolkit widget tree          Button, Table, Agenda, VBox ...  (or a skin.Object)
        │  Draw(p painter.Painter, theme *Theme)
        ▼
painter.Painter              FillRect, StrokeRect, Text, PutPixel, ...
        │
        ├─ PixelPainter  → an RGBA []byte   → window, browser canvas, PNG, Android
        └─ CellPainter   → a grid of cells  → a terminal, as 24-bit ANSI
```

A widget never learns which painter it was handed. A back-end never learns
which widgets drew. That is the whole reason one widget tree can be shown in a
native window, a browser and a terminal without a line of conditional code.

| | |
|---|---|
| [The painter seam]({{< relref "/rendering/painter.md" >}}) | the primitives, the optional capabilities, the two painters |
| [The toolkit]({{< relref "/rendering/toolkit.md" >}}) | the widget set, themes, layouts, text and accessibility |
| [Skins]({{< relref "/rendering/skin.md" >}}) | describe a widget's parts and states as data instead of Go |
