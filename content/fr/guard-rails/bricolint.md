---
title: "bricolint : pas d'interface dessinée à la main"
linkTitle: "bricolint"
weight: 20
description: "Fait échouer une pull request qui peint des éléments d'interface avec des primitives du painter dans le code applicatif, ou qui reconstruit un widget à chaque image."
tags: [ci, lint, painter]
---

Le *bricolage* — un dessin improvisé, fait main — est la façon dont une interface perd sans bruit
son retour visuel d'appui, de survol et de focus, son thème, sa mise à l'échelle HiDPI
et son accessibilité. [`go-widgets/bricolint`](https://github.com/go-widgets/bricolint)
empêche une application passée à la boîte à outils de revenir en arrière.

```go
p.FillRect(bar, theme.Bg)              // flagged: hand-drawn chrome
toolkit.NewBackdrop(theme.Bg).Draw(p)  // a toolkit widget

func (v *view) Draw(p painter.Painter) {
	b := toolkit.NewButton("ok")       // flagged: a new widget every frame
	b.Draw(p)
}
```

1. **Une primitive du painter dans le code applicatif** : une méthode de dessin (`FillRect`,
   `StrokeRect`, `FillPath`, `DrawImage`, `Text`, `PutPixel`, …) appelée sur un
   récepteur dont le type statique est un type du painter. Interroger ou découper la
   surface (`Size`, `PushClip`, `PushTranslate`) n'est pas dessiner et n'est pas
   signalé. `-primitives=` remplace la liste.
2. **Un widget jeté à chaque image** : un appel `toolkit.New…` dans une méthode
   nommée `Draw`, `Paint` ou `Render`. Un widget reconstruit à chaque peinture ne garde aucun
   état d'interaction. Construisez-le une fois, gardez-le dans un champ, pilotez-le par une
   liaison. `-checkthrow=false` désactive ce contrôle.

Certain code est une véritable feuille — un framebuffer de jeu, un back-end de painter, les
entrailles de la boîte à outils elle-même — et le dit explicitement :

```go
p.FillRect(bg, c) //bricolint:allow engine SVG raster blit — a genuine leaf
```

```go
//bricolint:allowfile painter back-end — this file IS the leaf
package pdfsurface
```

**La raison est obligatoire** : une directive qui n'en donne pas est ignorée, et le
signalement continue de se déclencher jusqu'à ce que quelqu'un écrive la justification.

```yaml
jobs:
  bricolint:
    uses: go-widgets/bricolint/.github/workflows/bricolint.yml@main
```
