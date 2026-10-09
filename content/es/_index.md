---
title: "go-widgets"
linkTitle: "Inicio"
type: docs
cascade:
  type: docs
description: "Un conjunto de widgets en Go puro y todas las superficies sobre las que pinta: una ventana nativa en X11, Wayland, macOS y Windows, un canvas de navegador, un terminal, un APK de Android."
---

**Un único árbol de widgets, pintado por Go, en cada superficie a la que llega
Go.** Una ventana nativa en X11, Wayland, macOS y Windows; un `<canvas>` en una
pestaña del navegador; un terminal; un APK de Android. Sin cadena de herramientas
de C, sin webview, sin conjunto de widgets del sistema: todos los módulos se
compilan con `CGO_ENABLED=0`, y cada píxel lo dibuja el toolkit.

```go
w, err := window.Open(window.Config{Title: "Hello"})
if err != nil {
	return err
}
defer w.Close()

box := toolkit.NewVBox()
box.Append(toolkit.NewLabel("Hello from go-widgets"))
box.Append(toolkit.NewButton("Click me", func() { /* ... */ }))
return w.Run(box) // an X11 or Wayland window, an NSWindow, a Win32 window
```

Eso es
[`cmd/windowdemo`](https://github.com/go-widgets/window/blob/main/cmd/windowdemo/main.go),
abreviado. El mismo `box` dibujado en un navegador, un terminal o un PNG es el
mismo código con otra superficie debajo.

## Por qué un solo toolkit lo dibuja todo {#why-one-toolkit-draws-everything}

Un widget que recurre al botón propio de la plataforma es tan portable como la
plataforma menos capaz a la que se dirige, y tiene un aspecto distinto en cada
una. go-widgets va en sentido contrario: el `Draw` de un widget solo habla con un
[`painter.Painter`]({{< relref "/rendering/painter.md" >}}), un puñado de
primitivas (rellenar un rectángulo, trazar su contorno, fijar un píxel, dibujar
texto). Que esas llamadas se conviertan en píxeles RGBA o en celdas de terminal
es asunto del backend, no del widget.

Lo que cuesta se dice sin rodeos: ningún aspecto nativo por defecto y ninguna
accesibilidad gratuita; el toolkit publica su propio
[árbol de accesibilidad]({{< relref "/rendering/toolkit.md#accessibility" >}})
en cada plataforma. Lo que aporta es un único conjunto de widgets, probado una
sola vez, con el mismo comportamiento en todas partes, y un binario que se
compila de forma cruzada desde cualquier máquina hacia cualquiera de ellas.

## Por dónde seguir {#where-to-go-next}

| | |
|---|---|
| [Una primera aplicación]({{< relref "/getting-started.md" >}}) | desde `app-template`, o desde una ventana vacía |
| [Cómo llega un widget a la pantalla]({{< relref "/rendering/_index.md" >}}) | la costura del painter, el conjunto de widgets, las skins |
| [La costura del painter]({{< relref "/rendering/painter.md" >}}) | las primitivas con las que se dibuja cada widget |
| [El toolkit]({{< relref "/rendering/toolkit.md" >}}) | 160 tipos de widget, temas, layouts, texto, accesibilidad |
| [Skins]({{< relref "/rendering/skin.md" >}}) | las partes y los estados de un widget en un archivo de datos |
| [El estado vive en un view-model]({{< relref "/state/_index.md" >}}) | por qué los campos de los widgets no se asignan a mano |
| [MVVM]({{< relref "/state/mvvm.md" >}}) | observables, comandos, binders, deshacer |
| [La columna vertebral de datos]({{< relref "/state/data.md" >}}) | registros tipados, consultas, un almacén local o remoto |
| [Dónde pinta]({{< relref "/surfaces/_index.md" >}}) | cada backend y cómo se elige |
| [Una ventana nativa]({{< relref "/surfaces/native-window.md" >}}) | X11, Wayland, Cocoa, Win32, GTK4, Android, wasmbox |
| [Una aplicación y su bandeja]({{< relref "/surfaces/application.md" >}}) | ciclo de vida, apariencia, un icono en la barra de menús |
| [Una pestaña del navegador]({{< relref "/surfaces/browser.md" >}}) | un simple `<canvas>`, o una ventana de wasmdesk |
| [Un terminal]({{< relref "/surfaces/terminal.md" >}}) | widgets nativos de celdas y un ejecutor interactivo |
| [Android]({{< relref "/surfaces/android.md" >}}) | un APK real pintado por un proceso sin CGO |
| [Instantáneas]({{< relref "/surfaces/snapshots.md" >}}) | un widget como SVG o PNG |
| [Reglas que impone la CI]({{< relref "/guard-rails/_index.md" >}}) | `mvvmlint` y `bricolint` |
| [Módulos]({{< relref "/modules.md" >}}) | los diecinueve, con sus fuentes y referencias |
| [Estado]({{< relref "/status.md" >}}) | qué está verificado y dónde, y qué no está hecho |

## Licencia {#licence}

BSD-3-Clause.
