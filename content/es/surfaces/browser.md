---
title: "Una pestaña del navegador"
linkTitle: "Navegador"
weight: 30
description: "Ejecutar un árbol de widgets en un simple <canvas> con go-widgets/webcanvas, o como una ventana del compositor wasmdesk."
tags: [superficies, wasm, navegador, webcanvas, wasmdesk]
---

El toolkit es Go puro sin dependencia del DOM, así que `GOOS=js GOARCH=wasm` lo
compila como cualquier otro destino. Hay dos maneras de poner el resultado
delante de alguien:

## Un simple canvas {#a-plain-canvas}

[`go-widgets/webcanvas`](https://github.com/go-widgets/webcanvas) copia un
framebuffer RGBA en un `<canvas>` y devuelve a la escena los eventos de puntero
y de teclado del DOM. Sin compositor, sin `SharedArrayBuffer`, sin cabeceras de
aislamiento de origen cruzado: cualquier alojamiento estático lo sirve.

```go
//go:build js && wasm

package main

import "github.com/go-widgets/webcanvas"

func main() { webcanvas.Run("screen", myScene()) }
```

La escena implementa `App`: `Size`, `Draw(buf []byte)` y un método por tipo de
evento, cada uno indicando si algo ha cambiado para que un fotograma sin
cambios no se repinte. También puede implementar `Ticker`, `Animator`,
`Resizer` o `Scroller`.

La [galería](https://go-widgets.github.io/gallery/) y
[`app-template`]({{< relref "/getting-started.md" >}}) funcionan así.

O deje que `window` elija. Desde window v0.87.0,
[`window.Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}})
compilado para `js/wasm` en una página corriente devuelve un backend que dibuja
en el `<canvas>` de la página (`Config.Canvas`, por defecto `"screen"`) a través
de `webcanvas`. Una aplicación escrita contra `window` se ejecuta entonces sin
cambios como ventana nativa, en una pestaña del navegador y en wasmdesk.

## Una ventana en el escritorio wasmdesk {#a-window-on-the-wasmdesk-desktop}

Dentro de un worker de wasmdesk, [`window.Open`]({{< relref "/surfaces/native-window.md" >}})
devuelve un cliente del compositor de navegador
[wasmdesk/wasmbox](https://github.com/wasmdesk/wasmbox). Asigna su superficie
en un `SharedArrayBuffer`, dice `hello` por su `MessagePort`, espera `welcome`,
pinta en el búfer compartido y envía `commit`: la superficie entera, o solo los
rectángulos dañados. Los mensajes `input` del compositor se convierten en
`toolkit.Event`, como en X11.

Así, una aplicación escrita contra `window` funciona sin cambios como ventana
nativa y como ventana de un escritorio en una pestaña del navegador. La
[demo del shell de escritorio](https://github.com/go-widgets/desktop) hace
exactamente eso, a partir de un solo código fuente.

El códec del protocolo está cubierto al 100 % en todas las plataformas. La
prueba en vivo ejecuta Chromium headless contra el compositor wasmdesk real:
lanza el cliente, lee los píxeles compuestos por el propio compositor, hace clic
y comprueba que el clic ha llegado al widget.
