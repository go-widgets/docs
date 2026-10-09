---
title: "Android"
linkTitle: "Android"
weight: 50
description: "go-widgets/android: un APK instalable cuya interfaz entera la organiza y la pinta un proceso Go con CGO_ENABLED=0."
tags: [superficies, android]
---

Android es la única plataforma en la que un proceso sin CGO no puede poseer una
ventana: todos los caminos hacia una superficie dibujable pasan por JNI, y JNI
necesita cgo. No hay ningún protocolo de red que hablar en su lugar, como sí
tienen X11 y Wayland.

Por eso [`go-widgets/android`](https://github.com/go-widgets/android) son dos
procesos:

| | |
|---|---|
| **Host Java** (`host/`) | posee la `Activity`, el `SurfaceView`, el táctil, las teclas y el ciclo de vida. Copia píxeles; no sabe nada de widgets. |
| **Aplicación Go** (`cmd/gwapp`) | un ejecutable `CGO_ENABLED=0 GOOS=android` corriente. Posee el layout, los widgets, el tema, la detección de impactos y el foco, sin cambios respecto a cualquier otro backend. |

El host Java ocupa el lugar que ocupa el servidor X en Linux. Los píxeles
viajan a través de un **memfd** que la aplicación crea y entrega al host por el
socket; el lado Go escribe RGBA_8888, que es byte a byte el ARGB_8888 de
Android, así que la copia se hace sin conversión y solo del rectángulo dañado.
La entrada, los márgenes de inserción, el texto del IME y el árbol de
accesibilidad atraviesan un protocolo por tramas sobre un `LocalSocket`
abstracto.

```go
c, err := android.Dial("my app", nil) // nil theme: toolkit.DefaultDark()
if errors.Is(err, android.ErrUnsupported) {
	return nil // not under a host
}
defer c.Close()
return c.Run(myWidgetTree())
```

`Client` satisface el `Backend` de `window`, de modo que una aplicación pasa de
Android a los backends de escritorio sin cambiar una línea por encima de la
ventana, y [`window.Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}})
lo elige por sí solo cuando el socket del host está presente.

## Solo arm64 {#arm64-only}

`android/arm64` es el único destino Android que Go enlaza sin cgo; `arm`,
`amd64` y `386` requieren enlazado externo. La CI comprueba ambas mitades —el
que compila y los tres que no—, de modo que el día en que Go levante la
restricción, la compilación lo dirá.

## Accesibilidad {#accessibility}

Un lector de pantalla vería el `SurfaceView` como un único rectángulo opaco. En
su lugar, el host le ofrece una jerarquía de vistas virtual: un nodo por cada
widget accesible, con la clase `android.widget.*` a partir de la cual Android
elige sus anuncios, el texto, los límites y una acción de activación. El árbol
se **extrae** cuando algo lo lee, nunca se envía por iniciativa propia, así que
una aplicación sin ningún servicio de accesibilidad conectado nunca lo
construye. Una activación vuelve como un clic corriente en el centro del
elemento, por el mismo código que recorre un toque.

## Táctil y animación {#touch-and-animation}

Cada muestra táctil llega al árbol como un evento táctil y luego como un evento
de ratón: el primero para los widgets que reconocen gestos, el segundo para los
muchos que solo escuchan clics. Los widgets animados avanzan en un bucle de
fotogramas que empieza cuando algo comienza a animarse y se detiene cuando
nada lo hace.
