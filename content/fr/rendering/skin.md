---
title: "Skins : les parties et les états d'un widget sous forme de données"
linkTitle: "Skins"
weight: 30
description: "go-widgets/skin décrit les parties d'un widget, son apparence par état et ses transitions pilotées par des signaux dans un fichier .skin.json, rendu par un seul moteur."
tags: [rendu, skin, thème]
---

Les widgets de la boîte à outils sont écrits en Go : les parties d'un `Button`, ses apparences
au survol, appuyé et désactivé, et son retour au clic vivent tous dans `button.go`.
C'est rapide et exact, et chaque changement visuel est un changement de code.

[`go-widgets/skin`](https://github.com/go-widgets/skin) fait passer cette description
dans des données — l'idée d'Edje, d'Enlightenment. Un document `.skin.json` nomme les
**parties** d'un widget, leur aspect dans chaque **état**, et les **programmes** qui
les font passer d'un état à l'autre quand un signal arrive. Changer de skin, ou construire un nouveau
contrôle, ne demande alors aucune recompilation : le programme charge d'autres octets.

```go
theme, err := skin.Load(jsonBytes) // parse and validate every collection
if err != nil {
	return err
}
obj, _ := theme.New("button")
obj.Bind(skin.NewMVVMContext().Set("$.label", labelObs))
obj.SetBounds(toolkit.Rect{W: 96, H: 28})

// each frame
obj.OnEvent(ev)               // pointer events become signals, signals states
busy := obj.Tick(time.Now())  // advance the transitions
obj.Draw(p, theme.Palette())
```

Un `skin.Object` est un `toolkit.Widget` ; il prend donc place dans une `VBox` à côté de
widgets écrits à la main. Les mises en page placent les widgets ; une skin agence ce qui est
à l'intérieur de l'un d'eux.

## Le format {#the-format}

```jsonc
{
  "collections": {
    "button": {
      "min": { "w": 96, "h": 28 },
      "parts": [
        { "name": "bg", "type": "rect",
          "states": {
            "default": { "color": "@surface",     "border": "@border", "radius": 6 },
            "hover":   { "color": "@surface_alt", "border": "@border", "radius": 6 },
            "pressed": { "color": "@accent",      "border": "@border", "radius": 6 } } },
        { "name": "label", "type": "text", "text_from": "$.label", "align": [0.5, 0.5],
          "states": { "default": { "ink": "@on_surface" } } }
      ],
      "programs": [
        { "on": "mouse,in",    "target": ["bg", "label"], "to": "hover", "in": 0.12, "ease": "ease_out_cubic" },
        { "on": "mouse,click", "emit": "clicked" }
      ]
    }
  }
}
```

- Une **partie** est un `rect`, un `text` ou une `image`, placée par deux coins relatifs
  (`rel1`, `rel2`) par rapport à l'objet ou à une partie antérieure, avec des décalages en pixels et
  relatifs à la police (`offset_em`), si bien qu'une partie texte suit la police.
- Un **état** fixe les couleurs, la bordure, le rayon et la visibilité, et peut déplacer ou
  redimensionner la partie.
- Un **programme** réagit à un signal, fait passer des parties dans un état sur une durée donnée
  avec une courbe d'accélération nommée, et peut émettre (`emit`) un signal vers l'extérieur — vers une
  [`mvvm.Command`]({{< relref "/state/mvvm.md" >}}), par exemple.
- Les couleurs sont des `@tokens` résolus d'après le `Theme` de la boîte à outils (y compris chaque
  entrée GTK `@define-color` sous la forme `@extra:<name>`), de l'hexadécimal, ou des tableaux RGBA.

Le JSON est décodé par la bibliothèque standard : aucune dépendance tierce.

## Éprouvé face aux widgets écrits à la main {#proven-against-the-hand-written-widgets}

Le bouton, la case à cocher, l'interrupteur, la puce et la carte de la boîte à outils sont chacun réécrits
sous forme de collection de skin, et les tests vérifient que les deux rendus sont **identiques au pixel
près** dans chaque état. Un `Draw` avec skin est comparé au
`Draw` écrit à la main dans un benchmark.
