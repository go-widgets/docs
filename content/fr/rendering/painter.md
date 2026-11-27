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

## Enregistrer en SVG {#recording-to-svg}

`svgrec.Recorder` (paquet `github.com/go-widgets/painter/svgrec`) est un troisième
painter : il transforme les mêmes appels en un **document SVG natif**, des éléments
`<rect>`, `<path>` et `<clipPath>`, et un `<image>` uniquement là où un raster est
réellement dessiné. Le résultat est sans perte et redimensionnable, et déterministe :
les mêmes appels donnent les mêmes octets.

```go
r, _ := svgrec.New(1000, 700)
widget.Draw(r, painter.LightTheme())
doc, err := r.SVG() // err is the first call that was refused, if any
```

Il implémente `Clipper`, `Translator`, `PathPainter`, `ImagePainter` et
`MaskPainter`. Là où SVG n'a pas d'équivalent fidèle, la correspondance le dit
plutôt que d'approximer en silence :

| Appel | SVG |
|---|---|
| `FillRect`, `PutPixel` | `<rect>` |
| `FillRoundRect` | `<rect rx>` ; le coin est anticrénelé différemment de celui du pixel painter |
| `StrokeRect`, `StrokeRoundRect` | un cadre `evenodd`, anneaux dessinés vers l'intérieur comme le fait le pixel painter |
| `Text` | contours de glyphes (ci-dessous) |
| `PushClip` / `PushTranslate` | `<clipPath>` plus un `<g>` imbriqué ; la translation est appliquée aux coordonnées |
| `FillPath`, `StrokePath` | `<path>` ; une largeur de trait est exprimée en unités utilisateur, elle suit donc la viewBox, et un sous-chemin de longueur nulle peint un point en SVG là où le pixel painter ne peint rien |
| `DrawImage`, `DrawMask` | `<image>` avec une URI de données PNG |

**Le texte devient des contours, pas du `<text>`.** La police 5x7 intégrée est une
grille de pixels : chaque chaîne est donc un seul `<path>` de segments fusionnés,
enveloppé dans un `<g>` dont le `<title>` contient le texte pour les technologies
d'assistance. Une police embarquée en sous-ensemble exigerait un écrivain de
polices et un visualiseur qui la respecte, et un visualiseur qui ne le fait pas
se rabattrait sur une autre police et décalerait la mise en page. Le coût : le
texte n'est pas sélectionnable. Le texte composé dans une police TrueType n'est pas
encore enregistré en contours ; `FacePainter` n'est pas proposé, il arrive donc
sous forme de `DrawMask` raster.

Les méthodes du painter ne renvoient aucune erreur : un appel refusé (UTF-8
invalide, un caractère que XML ne peut pas porter, une coordonnée au-delà de
1<<24, un appel ou un document dépassant ses `Limits`) est ignoré et la première
erreur est rapportée par `Err()` et `SVG()`. Les cibles de fuzzing relisent chaque
sortie avec `encoding/xml` et échouent sur tout élément ou attribut que
l'écrivain ne produit pas lui-même.

Comparé au pixel painter, rastérisé avec le `go-gfx/gfx/svg` en Go pur, les
widgets d'exemple, les rectangles, le texte bitmap, les clips et la translation
coïncident au pixel près ; les chemins diffèrent d'au plus 1 dans un canal ; les
coins arrondis anticrénelés, jusqu'à 61 dans un canal sur 7,5 % des pixels de la
pire scène.

## Essayer {#trying-it}

Les démos du dépôt lui-même dessinent les trois mêmes widgets avec les deux painters :

```sh
go run ./cmd/wui-demo --out demo.png     # pixels, written as a PNG
go run ./cmd/tui-demo --theme dark       # cells, written as ANSI to stdout
```

et [go-widgets.github.io/painter](https://go-widgets.github.io/painter/) montre
le painter en pixels dans un navigateur.
