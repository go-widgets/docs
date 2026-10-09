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

## Une fenêtre sur le bureau wasmdesk {#a-window-on-the-wasmdesk-desktop}

Sur `js/wasm`, [`window.Open`]({{< relref "/surfaces/native-window.md" >}})
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
