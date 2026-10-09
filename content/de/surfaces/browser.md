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

## Ein Fenster auf dem wasmdesk-Desktop {#a-window-on-the-wasmdesk-desktop}

Unter `js/wasm` gibt [`window.Open`]({{< relref "/surfaces/native-window.md" >}})
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
