---
title: "Snapshots: ein Widget als SVG oder PNG"
linkTitle: "Snapshots"
weight: 60
description: "go-widgets/svg verpackt jedes RGBA-Rendering bitgenau in ein eigenständiges SVG oder schreibt es als PNG."
tags: [oberflächen, svg, png]
---

[`go-widgets/svg`](https://github.com/go-widgets/svg) macht aus einem RGBA-Puffer
ein eigenständiges SVG: eine `viewBox`, die auf die Pixelgröße des Renderings festgelegt ist, und die
Pixel als Base64-PNG unter `image-rendering: pixelated`. Jedes angezeigte Pixel
ist das Pixel, das das Widget gezeichnet hat – keine Re-Vektorisierung, kein Schriftersatz –, und
es bleibt bei jedem Zoom scharf.

```go
// any RGBA producer
svg.Snapshot(f, surface, w, h, "my image")

// a toolkit widget
btn := toolkit.NewButton("Click me", nil)
btn.SetBounds(toolkit.Rect{W: 200, H: 40})
widget.Snapshot(f, btn, 200, 40, toolkit.DefaultLight(), "widget: button")
widget.PNG(p, btn, 200, 40, toolkit.DefaultLight())
```

Die Beschriftung wird, maskiert, zu `<title>` und `aria-label` des SVG. Das Wurzelpaket
hängt von nichts ab; das Unterpaket `widget` ist getrennt, damit ein Nutzer
mit eigenem Pixel-Erzeuger das Toolkit nicht in seinen Modulgraphen zieht.

`svg/cmd/gallery-render` rendert den Widget-Katalog, hell und dunkel:
[go-widgets.github.io/svg](https://go-widgets.github.io/svg/).

## Animiertes SVG

`svg.Animate` fügt eine zeitlich geordnete Liste von SVG-Einzelbildern (zum
Beispiel vom [`svgrec`](../../rendering/painter/)-Rekorder des Painters) zu
einem einzigen animierten SVG zusammen. Jeder Teilbaum, der sich zwischen den
Bildern nicht ändert, wird einmal in `<defs>` abgelegt und per `<use>`
wiederverwendet; aufeinanderfolgende gleiche Bilder werden verschmolzen, und das
Timing ist reines CSS (`@keyframes`, kein Skript), das `prefers-reduced-motion`
anhält.

```go
out, err := svg.Animate([]svg.Frame{
    {At: 0, SVG: first},
    {At: 100 * time.Millisecond, SVG: second},
}, svg.Options{End: time.Second, Title: "demo"})
```

Bei 60 Bildern zu 1000×700 schrumpft ein Bildschirm mit einem lebendigen Bereich
von 1,49 MB auf 65 KB (22,9-mal kleiner); ein statischer auf 24 KB (60-mal). Die
Eingabe wird per Positivliste geprüft und begrenzt (Bilder, Bytes, Dauer,
Größe); jede Ablehnung ist ein typisierter Fehler. `Document.Video` steuert einen
Rasterizer und einen Video-Encoder, die Sie über zwei kleine Schnittstellen
liefern; ein WebM-Encoder ist nicht enthalten.
