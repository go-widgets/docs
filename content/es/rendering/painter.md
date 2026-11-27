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

## Grabar a SVG {#recording-to-svg}

`svgrec.Recorder` (paquete `github.com/go-widgets/painter/svgrec`) es un tercer
painter: convierte las mismas llamadas en un **documento SVG nativo**, con
elementos `<rect>`, `<path>` y `<clipPath>`, y un `<image>` solo donde de verdad
se dibuja un raster. El resultado no tiene pérdidas, es redimensionable y es
determinista: las mismas llamadas producen los mismos bytes.

```go
r, _ := svgrec.New(1000, 700)
widget.Draw(r, painter.LightTheme())
doc, err := r.SVG() // err is the first call that was refused, if any
```

Implementa `Clipper`, `Translator`, `PathPainter`, `ImagePainter` y
`MaskPainter`. Donde SVG no tiene un equivalente fiel, la correspondencia lo dice
en lugar de aproximar en silencio:

| Llamada | SVG |
|---|---|
| `FillRect`, `PutPixel` | `<rect>` |
| `FillRoundRect` | `<rect rx>`; la esquina se suaviza de forma distinta a la del pixel painter |
| `StrokeRect`, `StrokeRoundRect` | un marco `evenodd`, con los anillos dibujados hacia dentro como hace el pixel painter |
| `Text` | contornos de glifos (abajo) |
| `PushClip` / `PushTranslate` | `<clipPath>` más un `<g>` anidado; la traslación se aplica a las coordenadas |
| `FillPath`, `StrokePath` | `<path>`; el grosor del trazo está en unidades de usuario, así que escala con el viewBox, y un subtrazado de longitud cero pinta un punto en SVG donde el pixel painter no pinta nada |
| `DrawImage`, `DrawMask` | `<image>` con un URI de datos PNG |

**El texto se convierte en contornos, no en `<text>`.** La fuente integrada de 5x7
es una cuadrícula de píxeles, así que cada cadena es un único `<path>` de tramos
fusionados, envuelto en un `<g>` cuyo `<title>` contiene el texto para las
tecnologías de asistencia. Una fuente incrustada con un subconjunto de glifos
necesitaría un escritor de fuentes y un visor que la respete, y un visor que no
lo haga recurriría a otra tipografía y movería el diseño. El coste: el texto no
se puede seleccionar. El texto compuesto con una fuente TrueType aún no se graba
como contornos; `FacePainter` no se ofrece, así que llega como un `DrawMask`
raster.

Los métodos del painter no devuelven error, por lo que una llamada rechazada
(UTF-8 no válido, un carácter que XML no puede transportar, una coordenada por
encima de 1<<24, una llamada o un documento que supere sus `Limits`) se descarta
y el primer error lo informan `Err()` y `SVG()`. Los destinos de fuzzing vuelven a
analizar cada salida con `encoding/xml` y fallan ante cualquier elemento o
atributo que el escritor no genere él mismo.

Frente al pixel painter, rasterizado con el `go-gfx/gfx/svg` en Go puro, los
widgets de ejemplo, los rectángulos, el texto de mapa de bits, los recortes y la
traslación coinciden al píxel; los trazados difieren como máximo en 1 en un
canal; las esquinas redondeadas suavizadas, hasta en 61 en un canal en el 7,5 %
de los píxeles de la peor escena.

## Probarlo {#trying-it}

Las demos del propio repositorio dibujan los mismos tres widgets a través de
ambos painters:

```sh
go run ./cmd/wui-demo --out demo.png     # pixels, written as a PNG
go run ./cmd/tui-demo --theme dark       # cells, written as ANSI to stdout
```

y [go-widgets.github.io/painter](https://go-widgets.github.io/painter/) muestra
el painter de píxeles en un navegador.
