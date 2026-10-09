---
title: "Instantáneas: un widget como SVG o PNG"
linkTitle: "Instantáneas"
weight: 60
description: "go-widgets/svg envuelve cualquier renderizado RGBA en un SVG autocontenido, exacto bit a bit, o lo escribe como PNG."
tags: [superficies, svg, png]
---

[`go-widgets/svg`](https://github.com/go-widgets/svg) convierte un búfer RGBA en
un SVG autocontenido: un `viewBox` fijado al tamaño en píxeles del renderizado,
y los píxeles como un PNG en base 64 bajo `image-rendering: pixelated`. Cada
píxel mostrado es el píxel que dibujó el widget —sin revectorización, sin
sustitución de fuentes— y se mantiene nítido a cualquier nivel de zoom.

```go
// any RGBA producer
svg.Snapshot(f, surface, w, h, "my image")

// a toolkit widget
btn := toolkit.NewButton("Click me", nil)
btn.SetBounds(toolkit.Rect{W: 200, H: 40})
widget.Snapshot(f, btn, 200, 40, toolkit.DefaultLight(), "widget: button")
widget.PNG(p, btn, 200, 40, toolkit.DefaultLight())
```

La etiqueta se convierte en el `<title>` y el `aria-label` del SVG, escapada. El
paquete raíz no depende de nada; el subpaquete `widget` está separado para que
un consumidor con su propio productor de píxeles no arrastre el toolkit a su
grafo de módulos.

`svg/cmd/gallery-render` renderiza el catálogo de widgets, en claro y en oscuro:
[go-widgets.github.io/svg](https://go-widgets.github.io/svg/).
