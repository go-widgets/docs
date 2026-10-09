---
title: "bricolint: nada de interfaz dibujada a mano"
linkTitle: "bricolint"
weight: 20
description: "Rechaza una pull request que pinta elementos de interfaz con primitivas del painter en el código de la aplicación, o que reconstruye un widget en cada fotograma."
tags: [ci, lint, painter]
---

El *bricolage* —dibujo improvisado, hecho a mano— es la manera en que una
interfaz pierde sin hacer ruido su respuesta visual al pulsar, al pasar el
puntero y al recibir el foco, sus temas, su escalado HiDPI y su accesibilidad.
[`go-widgets/bricolint`](https://github.com/go-widgets/bricolint)
impide que una aplicación que ya se ha pasado al toolkit vuelva atrás.

```go
p.FillRect(bar, theme.Bg)              // flagged: hand-drawn chrome
toolkit.NewBackdrop(theme.Bg).Draw(p)  // a toolkit widget

func (v *view) Draw(p painter.Painter) {
	b := toolkit.NewButton("ok")       // flagged: a new widget every frame
	b.Draw(p)
}
```

1. **Una primitiva del painter en el código de la aplicación**: un método de
   dibujo (`FillRect`, `StrokeRect`, `FillPath`, `DrawImage`, `Text`,
   `PutPixel`, …) invocado sobre un receptor cuyo tipo estático es un tipo de
   painter. Consultar o recortar la superficie (`Size`, `PushClip`,
   `PushTranslate`) no es dibujar y no se señala. `-primitives=` sustituye la lista.
2. **Un widget desechado en cada fotograma**: una llamada `toolkit.New…` dentro
   de un método llamado `Draw`, `Paint` o `Render`. Un widget reconstruido en
   cada pintado no conserva ningún estado de interacción. Constrúyalo una vez,
   guárdelo en un campo y contrólelo mediante un enlace. `-checkthrow=false`
   desactiva esta regla.

Hay código que es una auténtica hoja —un framebuffer de juego, un backend de
painter, las entrañas del propio toolkit— y lo declara explícitamente:

```go
p.FillRect(bg, c) //bricolint:allow engine SVG raster blit — a genuine leaf
```

```go
//bricolint:allowfile painter back-end — this file IS the leaf
package pdfsurface
```

**La razón es obligatoria**: una directiva sin ella se ignora, y el hallazgo
sigue saltando hasta que alguien deja escrita la justificación.

```yaml
jobs:
  bricolint:
    uses: go-widgets/bricolint/.github/workflows/bricolint.yml@main
```
