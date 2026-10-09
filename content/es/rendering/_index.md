---
title: "Cómo llega un widget a la pantalla"
linkTitle: "Renderizado"
weight: 10
description: "Un widget dibuja a través de un painter; el painter decide si eso se convierte en píxeles o en celdas de terminal; un backend presenta el resultado."
tags: [renderizado, painter, toolkit]
---

Tres capas, y cada una solo conoce la que tiene debajo:

```text
toolkit widget tree          Button, Table, Agenda, VBox ...  (or a skin.Object)
        │  Draw(p painter.Painter, theme *Theme)
        ▼
painter.Painter              FillRect, StrokeRect, Text, PutPixel, ...
        │
        ├─ PixelPainter  → an RGBA []byte   → window, browser canvas, PNG, Android
        └─ CellPainter   → a grid of cells  → a terminal, as 24-bit ANSI
```

Un widget nunca sabe qué painter ha recibido. Un backend nunca sabe qué
widgets han dibujado. Esa es toda la razón por la que un mismo árbol de widgets
puede mostrarse en una ventana nativa, un navegador y un terminal sin una sola
línea de código condicional.

| | |
|---|---|
| [La costura del painter]({{< relref "/rendering/painter.md" >}}) | las primitivas, las capacidades opcionales, los dos painters |
| [El toolkit]({{< relref "/rendering/toolkit.md" >}}) | el conjunto de widgets, temas, layouts, texto y accesibilidad |
| [Skins]({{< relref "/rendering/skin.md" >}}) | describir las partes y los estados de un widget como datos en lugar de en Go |
