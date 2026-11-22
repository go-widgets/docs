---
title: "Un onglet de navigateur"
linkTitle: "Navigateur"
weight: 30
description: "Exécuter un arbre de widgets dans un simple <canvas> avec go-widgets/webcanvas, ou comme fenêtre du compositeur wasmdesk."
tags: [surfaces, wasm, navigateur, webcanvas, wasmdesk]
---

La boîte à outils est en Go pur, sans dépendance au DOM ; `GOOS=js GOARCH=wasm` la compile
donc comme n'importe quelle autre cible. Deux façons de mettre le résultat sous les yeux de quelqu'un :

## Un simple canvas {#a-plain-canvas}

[`go-widgets/webcanvas`](https://github.com/go-widgets/webcanvas) copie un framebuffer
RGBA dans un `<canvas>` et renvoie les événements DOM de pointeur et de clavier
vers la scène. Pas de compositeur, pas de `SharedArrayBuffer`, pas d'en-têtes d'isolation
cross-origin : n'importe quel hébergement statique le sert.

```go
//go:build js && wasm

package main

import "github.com/go-widgets/webcanvas"

func main() { webcanvas.Run("screen", myScene()) }
```

La scène implémente `App` : `Size`, `Draw(buf []byte)`, et une méthode par
type d'événement, chacune indiquant si quelque chose a changé, pour qu'une image inchangée
ne soit pas repeinte. Elle peut aussi implémenter `Ticker`, `Animator`, `Resizer` ou
`Scroller`.

La [galerie](https://go-widgets.github.io/gallery/) et
[`app-template`]({{< relref "/getting-started.md" >}}) fonctionnent ainsi.

Ou bien laissez `window` choisir. Depuis window v0.87.0,
[`window.Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}})
compilé pour `js/wasm` sur une page ordinaire renvoie un back-end qui dessine dans le
`<canvas>` de la page (`Config.Canvas`, par défaut `"screen"`) via `webcanvas`. Une
application écrite pour `window` fonctionne alors sans modification comme fenêtre native, dans
un onglet de navigateur et dans wasmdesk.

## Ne repeindre que ce qui a changé {#repainting-only-what-changed}

Sans aide, chaque image est un `Draw` de toute la scène, une copie de tout le
framebuffer vers le navigateur et un `putImageData` de tout le canvas, même quand
seul un curseur a clignoté ou qu'une image d'invité dans un `Layer` est nouvelle.
Une scène qui sait ce qu'elle a modifié implémente `DrawDamaged(buf []byte) []Rect`
(`webcanvas.DamageDrawer`, webcanvas v0.15.0), et l'hôte ne copie et ne dépose
que ces rectangles.

- **Le contrat** est celui que le `DamageRenderer` de [`window`]({{< relref "/surfaces/native-window.md" >}})
  donne à un hôte de bureau, en termes de navigateur : la scène repeint ce qui a
  changé dans un framebuffer qui persiste d'une image à l'autre et renvoie les
  rectangles repeints (`webcanvas.Rect` est `toolkit.Rect`). Un résultat **vide**
  signifie que rien n'a changé et que rien n'est déposé. `Draw` s'exécute encore
  pour la première image et après chaque redimensionnement.
- **Les calques** sont redessinés à chaque image, sans repeindre la scène. Pour un
  calque déplacé, réduit ou disparu, l'ancien rectangle est restauré depuis le
  tampon de la scène ; sous un calque avec `Alpha < 1`, ou non marqué `Opaque`, la
  scène est restaurée à chaque image. `Opaque: true` promet que tous les pixels
  sont opaques, ce qui permet à une image d'invité immobile de ne rien déposer.
- **Ne pas l'utiliser** ne change rien : une scène sans `DrawDamaged` est dessinée
  et déposée en entier, comme avant. Le présentateur `webcanvas/gpu` en a sa
  propre version, plus bas.
- **Mesuré** sous `GOOS=js` dans node, en 1280×800, avec un faux contexte 2D : une
  image qui modifie un widget de 120×40 coûte 0,031 ms au lieu de 2,89 ms, et une
  image sans dégât avec un calque opaque 0,021 ms au lieu de 2,90 ms.

## Sur le GPU {#on-the-gpu}

`webcanvas.RunWith(id, app, presenter)` prend un `Presenter`, et le paquet
`webcanvas/gpu` en fournit un qui dessine les images avec WebGPU.
- **Comment :** une App qui implémente aussi `Paint(painter.Painter)` est
  enregistrée par [gpupainter](https://github.com/go-widgets/gpupainter) dans
  une liste [go-gfx/gpu2d](https://github.com/go-gfx/gpu2d). Son moteur WebGPU
  dessine l'image, qui est copiée dans la texture du canvas sur le GPU.
- **Pourquoi un presenter :** un canvas n'accepte qu'un seul type de contexte
  pour toute sa vie ; le presenter décide donc une fois, avant la première
  image.
- **Exactitude :** les pixels sont ceux de l'image CPU de `Draw`, octet pour
  octet (un test le garantit).
- **Dégâts :** une App qui implémente aussi `PaintDamaged(painter.Painter) []Rect`
  (`gpu.DamagePainted`, webcanvas v0.16.0 et v0.17.0) ne peint dans le painter
  que ce qui a changé, borné aux rectangles qu'elle renvoie, comme le
  `RenderDamaged` de toolkit. Un résultat **vide** veut dire que l'image n'est ni
  enregistrée ni présentée : une image immobile n'envoie aucun tampon de
  commandes au GPU (2 avant, 0 après, comptés dans Chrome, 0 allocation). Une
  image avec dégâts est enregistrée telle que `PaintDamaged` l'a peinte, donc la
  scène doit repeindre le fond sous les dégâts : avec une cellule endommagée sur
  1 200, la liste remise au moteur passe de 2 401 instances à 2. La première
  image, un redimensionnement et une App sans `PaintDamaged` sont enregistrés en
  entier.
- **La texture du canvas ne persiste pas :** après chaque présentation, la
  spécification WebGPU donne à `getCurrentTexture()` une texture noire
  transparente (Chrome fait de même), si bien que la cible propre du moteur
  garde le reste de l'image et que chaque image présentée en copie encore la
  totalité vers le canvas, sur le GPU.
- **Repli :** un navigateur sans WebGPU, ou une App sans `Paint`, reste sur le
  canvas 2D. Ce n'est une erreur ni dans un cas ni dans l'autre.

La [galerie web](https://go-widgets.github.io/gallery/) dessine ainsi.

## Une fenêtre sur le bureau wasmdesk {#a-window-on-the-wasmdesk-desktop}

Dans un worker wasmdesk, [`window.Open`]({{< relref "/surfaces/native-window.md" >}})
renvoie un client du compositeur de navigateur [wasmdesk/wasmbox](https://github.com/wasmdesk/wasmbox).
Il alloue sa surface dans un `SharedArrayBuffer`, dit
`hello` sur son `MessagePort`, attend `welcome`, peint dans le tampon partagé
et poste `commit` — la surface entière, ou seulement les rectangles endommagés.
Les messages `input` du compositeur deviennent des `toolkit.Event`, comme sous X11.

Une application écrite pour `window` s'exécute donc sans modification comme fenêtre native
et comme fenêtre d'un bureau dans un onglet de navigateur. La
[démo du shell de bureau](https://github.com/go-widgets/desktop) fait exactement
cela, à partir d'une seule source.

Le codec du protocole est couvert à 100 % sur chaque plateforme. La preuve en conditions réelles fait tourner
un Chromium headless face au vrai compositeur wasmdesk : elle lance le client,
lit les pixels composités par le compositeur lui-même, clique, et vérifie que le
clic a atteint le widget.
