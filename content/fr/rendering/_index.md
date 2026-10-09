---
title: "Comment un widget atteint l'écran"
linkTitle: "Rendu"
weight: 10
description: "Un widget dessine à travers un painter ; le painter décide si cela devient des pixels ou des cellules de terminal ; un back-end présente le résultat."
tags: [rendu, painter, toolkit]
---

Trois couches, et chacune ne connaît que celle du dessous :

```text
toolkit widget tree          Button, Table, Agenda, VBox ...  (or a skin.Object)
        │  Draw(p painter.Painter, theme *Theme)
        ▼
painter.Painter              FillRect, StrokeRect, Text, PutPixel, ...
        │
        ├─ PixelPainter  → an RGBA []byte   → window, browser canvas, PNG, Android
        └─ CellPainter   → a grid of cells  → a terminal, as 24-bit ANSI
```

Un widget n'apprend jamais quel painter on lui a remis. Un back-end n'apprend jamais
quels widgets ont dessiné. C'est toute la raison pour laquelle un même arbre de widgets peut
s'afficher dans une fenêtre native, un navigateur et un terminal sans une ligne de code conditionnel.

| | |
|---|---|
| [La jonction du painter]({{< relref "/rendering/painter.md" >}}) | les primitives, les capacités optionnelles, les deux painters |
| [La boîte à outils]({{< relref "/rendering/toolkit.md" >}}) | le jeu de widgets, les thèmes, les mises en page, le texte et l'accessibilité |
| [Skins]({{< relref "/rendering/skin.md" >}}) | décrire les parties et les états d'un widget par des données plutôt qu'en Go |
