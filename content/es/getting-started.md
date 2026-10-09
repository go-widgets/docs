---
title: "Una primera aplicación"
weight: 5
description: "Partir de app-template, que ya cumple MVVM y está en verde, o abrir una ventana vacía y añadirle widgets."
tags: [primeros pasos, app-template, ventana]
---

## Desde la plantilla {#from-the-template}

[`go-widgets/app-template`](https://github.com/go-widgets/app-template) es un
repositorio plantilla: una aplicación de navegador pequeña pero real, con un
campo de búsqueda, un filtro por categoría, una lista de las filas que coinciden
y una línea de estado. Todo lo que necesita una aplicación más grande ya está
conectado: un view-model sin ningún widget dentro, una vista enlazada a él, un
host wasm, una barrera de cobertura del 100 % y la
[barrera MVVM]({{< relref "/guard-rails/mvvmlint.md" >}}).

```sh
git clone https://github.com/go-widgets/app-template my-app
cd my-app
go mod edit -module github.com/you/my-app
./build.sh          # dist/app.wasm, dist/wasm_exec.js, dist/index.html
```

Sirva `dist/` por HTTP: una página abierta desde `file://` no puede instanciar wasm.

| Archivo | Qué contiene |
|---|---|
| `viewmodel.go` | todo el estado, como observables y comandos de [`mvvm`]({{< relref "/state/mvvm.md" >}}). Ningún widget. Se puede probar sin canvas. |
| `scene.go` | los widgets, cada uno enlazado al view-model mediante `mvvmtk`. Nunca asigna por sí mismo un campo de estado de un widget. |
| `main.go` | el único archivo con una etiqueta de compilación (`js && wasm`): reenvía la entrada al toolkit y copia el fotograma en un `<canvas>`. |

Edite los dos primeros y haga que el estado siga pasando por los binders: el
trabajo `mvvm` de la CI rechaza una pull request que asigne directamente un
campo de un widget.

## Desde una ventana vacía {#from-a-bare-window}

```go
package main

import (
	"github.com/go-widgets/toolkit"
	"github.com/go-widgets/window"
)

func main() {
	w, err := window.Open(window.Config{Title: "Demo"})
	if err == window.ErrUnsupported {
		return // no native back-end on this platform
	}
	if err != nil {
		panic(err)
	}
	defer w.Close()

	box := toolkit.NewVBox()
	box.Append(toolkit.NewLabel("Hello"))
	box.Append(toolkit.NewButton("Click me", func() {}))
	w.Run(box) // layout, draw, present, dispatch input, until closed
}
```

Un ancho y un alto dejados a cero piden al backend un tamaño por defecto
legible; en macOS es una fracción de la pantalla principal. `CGO_ENABLED=0 go build`
produce el binario para cada plataforma, desde cualquier máquina.

## Qué módulo para qué {#which-module-for-what}

| Quiere | Empiece por |
|---|---|
| una ventana de escritorio | [`window`]({{< relref "/surfaces/native-window.md" >}}) |
| una aplicación de escritorio con un icono en la bandeja y la apariencia del sistema | [`application`]({{< relref "/surfaces/application.md" >}}) |
| una página en un navegador | [`webcanvas`]({{< relref "/surfaces/browser.md" >}}), o la plantilla anterior |
| un programa de terminal | [`tui`]({{< relref "/surfaces/terminal.md" >}}) |
| una aplicación Android | [`android`]({{< relref "/surfaces/android.md" >}}) |
| una imagen de un widget para un README | [`svg`]({{< relref "/surfaces/snapshots.md" >}}) |

Para ver el conjunto de widgets antes de escribir nada, abra la
[galería](https://go-widgets.github.io/gallery/): todas las familias de widgets,
en vivo, en un canvas de navegador.
