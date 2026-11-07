---
title: "Ein Browser-Tab"
linkTitle: "Browser"
weight: 30
description: "Einen Widget-Baum in einem einfachen <canvas> mit go-widgets/webcanvas ausführen oder als Fenster des wasmdesk-Compositors."
tags: [oberflächen, wasm, browser, webcanvas, wasmdesk]
---

Das Toolkit ist reines Go ohne DOM-Abhängigkeit, daher baut `GOOS=js GOARCH=wasm`
es wie jedes andere Ziel. Zwei Wege, das Ergebnis jemandem vorzuführen:

## Ein einfaches Canvas {#a-plain-canvas}

[`go-widgets/webcanvas`](https://github.com/go-widgets/webcanvas) überträgt einen RGBA-
Framebuffer per Blit in ein `<canvas>` und leitet DOM-Zeiger- und Tastaturereignisse zurück
in die Szene. Kein Compositor, kein `SharedArrayBuffer`, keine Header für Cross-Origin-
Isolation: Jeder statische Host kann es ausliefern.

```go
//go:build js && wasm

package main

import "github.com/go-widgets/webcanvas"

func main() { webcanvas.Run("screen", myScene()) }
```

Die Szene implementiert `App`: `Size`, `Draw(buf []byte)` und eine Methode pro
Ereignisart, die jeweils meldet, ob sich etwas geändert hat, damit ein unveränderter Frame
nicht neu gezeichnet wird. Sie kann außerdem `Ticker`, `Animator`, `Resizer` oder
`Scroller` implementieren.

Die [Galerie](https://go-widgets.github.io/gallery/) und
[`app-template`]({{< relref "/getting-started.md" >}}) laufen auf diese Weise.

Oder man lässt `window` wählen. Seit window v0.87.0 gibt
[`window.Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}}),
für `js/wasm` gebaut, auf einer gewöhnlichen Seite ein Back-End zurück, das über
`webcanvas` in das `<canvas>` der Seite zeichnet (`Config.Canvas`, Standard `"screen"`).
Eine gegen `window` geschriebene Anwendung läuft dann unverändert als natives Fenster,
in einem Browser-Tab und in wasmdesk.

## Auf der GPU {#on-the-gpu}

`webcanvas.RunWith(id, app, presenter)` nimmt einen `Presenter` entgegen, und
das Paket `webcanvas/gpu` liefert einen, der Bilder mit WebGPU zeichnet.
- **Wie:** Eine App, die auch `Paint(painter.Painter)` implementiert, wird über
  [gpupainter](https://github.com/go-widgets/gpupainter) in eine
  [go-gfx/gpu2d](https://github.com/go-gfx/gpu2d)-Liste aufgezeichnet. Der
  WebGPU-Renderer zeichnet das Bild, das auf der GPU in die Canvas-Textur
  kopiert wird.
- **Warum ein Presenter:** Ein Canvas nimmt ein Leben lang nur eine
  Kontextart an; der Presenter entscheidet deshalb einmal, vor dem ersten Bild.
- **Exaktheit:** Die Pixel sind die des CPU-Bildes von `Draw`, Byte für Byte
  (ein Test sichert das).
- **Rückfälle:** Ein Browser ohne WebGPU oder eine App ohne `Paint` bleibt beim
  2D-Canvas. Beides ist kein Fehler.

Die [Web-Galerie](https://go-widgets.github.io/gallery/) zeichnet so.

## Ein Fenster auf dem wasmdesk-Desktop {#a-window-on-the-wasmdesk-desktop}

In einem wasmdesk-Worker gibt [`window.Open`]({{< relref "/surfaces/native-window.md" >}})
einen Client des Browser-Compositors [wasmdesk/wasmbox](https://github.com/wasmdesk/wasmbox)
zurück. Er legt seine Oberfläche in einem `SharedArrayBuffer` an, sagt
`hello` über seinen `MessagePort`, wartet auf `welcome`, zeichnet in den gemeinsamen
Puffer und sendet `commit` – die ganze Oberfläche oder nur die geschädigten Rechtecke.
Die `input`-Nachrichten des Compositors werden zu `toolkit.Event`, wie unter X11.

So läuft eine gegen `window` geschriebene Anwendung unverändert als natives Fenster
und als Fenster eines Desktops in einem Browser-Tab. Die
[Desktop-Shell-Demo](https://github.com/go-widgets/desktop) macht genau
das, aus einer einzigen Quelle.

Der Protokoll-Codec ist auf jeder Plattform zu 100 % abgedeckt. Der Live-Nachweis lässt
Headless-Chromium gegen den echten wasmdesk-Compositor laufen: Er startet den Client,
liest die vom Compositor selbst komponierten Pixel, klickt und prüft, dass der
Klick das Widget erreicht hat.
