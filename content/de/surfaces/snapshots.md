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
