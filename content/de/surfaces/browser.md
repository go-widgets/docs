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

## Nur neu zeichnen, was sich geändert hat {#repainting-only-what-changed}

Ohne Hilfe ist jedes Bild ein `Draw` der ganzen Szene, eine Kopie des ganzen
Framebuffers in den Browser und ein `putImageData` des ganzen Canvas, selbst wenn
nur ein Cursor geblinkt hat oder nur das Bild eines Gasts in einem `Layer` neu
ist. Eine Szene, die weiß, was sie geändert hat, implementiert
`DrawDamaged(buf []byte) []Rect` (`webcanvas.DamageDrawer`, webcanvas v0.15.0),
und der Host kopiert und setzt nur diese Rechtecke.

- **Der Vertrag** ist der, den der `DamageRenderer` von [`window`]({{< relref "/surfaces/native-window.md" >}})
  einem Desktop-Host gibt, in Browser-Begriffen: Die Szene zeichnet das
  Geänderte in einen Framebuffer, der zwischen den Bildern bestehen bleibt, und
  gibt die neu gezeichneten Rechtecke zurück (`webcanvas.Rect` ist
  `toolkit.Rect`). Ein **leeres** Ergebnis heißt: nichts hat sich geändert, nichts
  wird gesetzt. `Draw` läuft weiterhin für das erste Bild und nach jeder
  Größenänderung.
- **Layer** werden in jedem Bild neu gezeichnet, ohne die Szene neu zu zeichnen.
  Bei einem verschobenen, verkleinerten oder verschwundenen Layer wird das alte
  Rechteck aus dem Szenenpuffer wiederhergestellt; unter einem Layer mit
  `Alpha < 1` oder ohne `Opaque` wird die Szene in jedem Bild wiederhergestellt.
  `Opaque: true` verspricht, dass alle Pixel deckend sind, sodass ein ruhendes
  Gastbild nichts setzt.
- **Ohne** `DrawDamaged` ändert sich nichts: Die Szene wird wie bisher ganz
  gezeichnet und gesetzt. Der Presenter `webcanvas/gpu` hat dafür eine eigene
  Version, weiter unten.
- **Gemessen** unter `GOOS=js` in node, 1280×800, mit einem Fake-2D-Kontext: Ein
  Bild, das ein 120×40-Widget ändert, kostet 0,031 ms statt 2,89 ms, ein Bild
  ohne Änderung mit einem deckenden Layer 0,021 ms statt 2,90 ms.

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
- **Schaden:** Eine App, die auch `PaintDamaged(painter.Painter) []Rect`
  implementiert (`gpu.DamagePainted`, webcanvas v0.16.0 und v0.17.0), malt in den
  Painter nur, was sich geändert hat, begrenzt auf die zurückgegebenen
  Rechtecke, wie `RenderDamaged` des Toolkits. Ein **leeres** Ergebnis heißt: Das
  Bild wird weder aufgezeichnet noch präsentiert, ein ruhendes Bild sendet also
  keinen Command Buffer an die GPU (2 vorher, 0 nachher, in Chrome gezählt, 0
  Allokationen). Ein Bild mit Schaden wird so aufgezeichnet, wie `PaintDamaged`
  es gemalt hat; die Szene muss daher den Untergrund unter dem Schaden neu
  malen: Bei einer beschädigten Zelle von 1.200 sinkt die an den Renderer
  gegebene Liste von 2.401 Instanzen auf 2. Das erste Bild, eine Größenänderung
  und eine App ohne `PaintDamaged` werden ganz aufgezeichnet.
- **Die Canvas-Textur bleibt nicht erhalten:** Nach jeder Präsentation gibt die
  WebGPU-Spezifikation `getCurrentTexture()` eine transparent schwarze Textur
  (Chrome ebenso), sodass das eigene Ziel des Renderers den Rest des Bildes
  behält und jedes präsentierte Bild weiterhin ganz in das Canvas kopiert wird,
  auf der GPU.
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
