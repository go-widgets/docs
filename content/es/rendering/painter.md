---
title: "La costura del painter"
linkTitle: "Painter"
weight: 10
description: "Las primitivas con las que se dibuja cada widget, las capacidades opcionales que un painter puede añadir, y los painters de píxeles y de celdas."
tags: [renderizado, painter]
---

[`go-widgets/painter`](https://github.com/go-widgets/painter) define la única
interfaz a través de la cual dibuja cada widget:

```go
type Painter interface {
	FillRect(r Rect, c RGBA)
	StrokeRect(r Rect, c RGBA, lineW int)
	FillRoundRect(r Rect, radius int, c RGBA)
	StrokeRoundRect(r Rect, radius int, c RGBA, lineW int)
	PutPixel(x, y int, c RGBA)
	Text(x, y int, s string, ink RGBA)
	Size() (w, h int)
}
```

El conjunto es pequeño a propósito: cada backend debe implementarlo entero, y
una cuadrícula de terminal no puede redondear una esquina ni variar el grosor
de un trazo. Por eso el contrato dice qué ocurre cuando no puede: el grosor de
`StrokeRect` es una indicación que una cuadrícula de celdas ignora, y
`FillRoundRect` recurre allí a un relleno cuadrado.

## Capacidades opcionales {#optional-capabilities}

Lo que no todas las superficies pueden hacer no está en la interfaz base. Un
widget que lo necesita hace una aserción de tipo, y dibuja algo razonable
cuando la respuesta es no:

| Interfaz | Métodos | Se usa para |
|---|---|---|
| `Clipper` | `PushClip`, `PopClip` | mantener un hijo desplazado dentro de los límites de su padre |
| `Translator` | `PushTranslate`, `PopTranslate` | dibujar un hijo en sus propias coordenadas |
| `PathPainter` | `FillPath`, `StrokePath` | curvas, gráficos, formas con antialiasing |
| `ImagePainter` | `DrawImage` | iconos, miniaturas, imágenes |
| `MaskPainter` | `DrawMask` | glifos y otras máscaras de cobertura en un solo color |
| `FacePainter` | `TextFace` | texto en una tipografía y un tamaño concretos |

```go
if c, ok := p.(painter.Clipper); ok {
	c.PushClip(bounds)
	defer c.PopClip()
}
```

## Los dos painters {#the-two-painters}

| | Escribe en | Lo presenta |
|---|---|---|
| `NewPixelPainter(buf, w, h)` | un `[]byte` RGBA propiedad del llamante (`NewPixelPainterBGRA` para superficies BGRA) | una ventana nativa, un canvas de navegador, una superficie Android, un PNG o un SVG |
| `NewCellPainter(w, h)` | una cuadrícula de celdas (runa, primer plano, fondo) con un serializador ANSI de 24 bits | un terminal, a través de [`tui`]({{< relref "/surfaces/terminal.md" >}}) |

El búfer sigue perteneciendo al llamante. Un `PixelPainter` solo traduce
primitivas en escrituras, de modo que el mismo `[]byte` puede entregarse a un
backend de ventana, a un `<canvas>` o a un codificador de imágenes sin ninguna
copia intermedia.

## Probarlo {#trying-it}

Las demos del propio repositorio dibujan los mismos tres widgets a través de
ambos painters:

```sh
go run ./cmd/wui-demo --out demo.png     # pixels, written as a PNG
go run ./cmd/tui-demo --theme dark       # cells, written as ANSI to stdout
```

y [go-widgets.github.io/painter](https://go-widgets.github.io/painter/) muestra
el painter de píxeles en un navegador.
