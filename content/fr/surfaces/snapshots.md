---
title: "Instantanés : un widget en SVG ou en PNG"
linkTitle: "Instantanés"
weight: 60
description: "go-widgets/svg enveloppe n'importe quel rendu RGBA dans un SVG autonome, exact au bit près, ou l'écrit en PNG."
tags: [surfaces, svg, png]
---

[`go-widgets/svg`](https://github.com/go-widgets/svg) transforme un tampon RGBA en
SVG autonome : une `viewBox` calée sur la taille en pixels du rendu, et les
pixels sous forme de PNG en base 64 avec `image-rendering: pixelated`. Chaque pixel affiché
est le pixel que le widget a dessiné — pas de revectorisation, pas de substitution de police — et
il reste net à tout niveau de zoom.

```go
// any RGBA producer
svg.Snapshot(f, surface, w, h, "my image")

// a toolkit widget
btn := toolkit.NewButton("Click me", nil)
btn.SetBounds(toolkit.Rect{W: 200, H: 40})
widget.Snapshot(f, btn, 200, 40, toolkit.DefaultLight(), "widget: button")
widget.PNG(p, btn, 200, 40, toolkit.DefaultLight())
```

Le libellé devient le `<title>` et l'`aria-label` du SVG, échappés. Le paquet
racine ne dépend de rien ; le sous-paquet `widget` est séparé pour qu'un consommateur
doté de son propre producteur de pixels n'entraîne pas la boîte à outils dans son graphe de modules.

`svg/cmd/gallery-render` rend le catalogue de widgets, en clair et en sombre :
[go-widgets.github.io/svg](https://go-widgets.github.io/svg/).

## SVG animé

`svg.Animate` assemble une liste d'images SVG datées (par exemple issues de
l'enregistreur [`svgrec`](../../rendering/painter/) du peintre) en un seul SVG
animé. Chaque sous-arbre qui ne change pas d'une image à l'autre est stocké une
fois dans `<defs>` et réutilisé par `<use>` ; les images successives identiques
fusionnent, et le minutage est du pur CSS (`@keyframes`, aucun script),
interrompu par `prefers-reduced-motion`.

```go
out, err := svg.Animate([]svg.Frame{
    {At: 0, SVG: first},
    {At: 100 * time.Millisecond, SVG: second},
}, svg.Options{End: time.Second, Title: "demo"})
```

Sur 60 images de 1000×700, un écran avec une zone vivante passe de 1,49 Mo
d'images à 65 Ko (22,9 fois plus petit) ; un écran statique à 24 Ko (60 fois).
L'entrée est analysée par liste blanche et bornée (images, octets, durée,
taille), et chaque refus est une erreur typée. `Document.Video` pilote un
rastériseur et un encodeur vidéo que vous fournissez via deux petites
interfaces ; aucun encodeur WebM n'est inclus.
