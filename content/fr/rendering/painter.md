---
title: "La jonction du painter"
linkTitle: "Painter"
weight: 10
description: "Les primitives avec lesquelles chaque widget est dessiné, les capacités optionnelles qu'un painter peut ajouter, et les painters en pixels et en cellules."
tags: [rendu, painter]
---

[`go-widgets/painter`](https://github.com/go-widgets/painter) définit l'unique
interface à travers laquelle chaque widget dessine :

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

L'ensemble est petit à dessein : chaque back-end doit l'implémenter en entier, et une
grille de terminal ne peut ni arrondir un coin ni faire varier l'épaisseur d'un trait. Le contrat
dit donc ce qui se passe quand c'est impossible — l'épaisseur de `StrokeRect` est une indication qu'une
grille de cellules ignore, et `FillRoundRect` s'y replie sur un remplissage carré.

## Capacités optionnelles {#optional-capabilities}

Ce que toutes les surfaces ne savent pas faire n'est pas dans l'interface de base. Un widget qui
en a besoin fait une assertion de type, et dessine quelque chose de raisonnable quand la réponse est non :

| Interface | Méthodes | Sert à |
|---|---|---|
| `Clipper` | `PushClip`, `PopClip` | garder un enfant défilé dans les limites de son parent |
| `Translator` | `PushTranslate`, `PopTranslate` | dessiner un enfant dans ses propres coordonnées |
| `PathPainter` | `FillPath`, `StrokePath` | courbes, graphiques, formes anticrénelées |
| `ImagePainter` | `DrawImage` | icônes, vignettes, images |
| `MaskPainter` | `DrawMask` | glyphes et autres masques de couverture d'une seule couleur |
| `FacePainter` | `TextFace` | du texte dans une police et une taille données |

```go
if c, ok := p.(painter.Clipper); ok {
	c.PushClip(bounds)
	defer c.PopClip()
}
```

## Les deux painters {#the-two-painters}

| | Écrit dans | Présenté par |
|---|---|---|
| `NewPixelPainter(buf, w, h)` | un `[]byte` RGBA appartenant à l'appelant (`NewPixelPainterBGRA` pour les surfaces BGRA) | une fenêtre native, un canvas de navigateur, une surface Android, un PNG ou un SVG |
| `NewCellPainter(w, h)` | une grille de cellules (rune, premier plan, arrière-plan) avec un sérialiseur ANSI 24 bits | un terminal, via [`tui`]({{< relref "/surfaces/terminal.md" >}}) |

Le tampon reste celui de l'appelant. Un `PixelPainter` ne fait que traduire les primitives
en écritures, si bien que le même `[]byte` peut être remis à un back-end de fenêtre, à un
`<canvas>` ou à un encodeur d'images sans copie intermédiaire.

## Essayer {#trying-it}

Les démos du dépôt lui-même dessinent les trois mêmes widgets avec les deux painters :

```sh
go run ./cmd/wui-demo --out demo.png     # pixels, written as a PNG
go run ./cmd/tui-demo --theme dark       # cells, written as ANSI to stdout
```

et [go-widgets.github.io/painter](https://go-widgets.github.io/painter/) montre
le painter en pixels dans un navigateur.
