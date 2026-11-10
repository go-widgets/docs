---
title: "Módulos"
weight: 50
description: "Los veintitrés módulos de go-widgets, para qué sirve cada uno y dónde están su código fuente y su referencia de API."
tags: [módulos]
---

La referencia de API la genera a partir del código fuente
[pkg.go.dev](https://pkg.go.dev/), que sigue las etiquetas publicadas; estas
páginas explican, y no la copian. La etiqueta es la más reciente en el momento
de escribir; la autoridad es la lista de etiquetas del propio repositorio.

| Módulo | Etiqueta | Qué es | Referencia |
|---|---|---|---|
| [`painter`](https://github.com/go-widgets/painter) | v0.17.0 | [la costura de dibujo]({{< relref "/rendering/painter.md" >}}): painters de píxeles y de celdas | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/painter) |
| [`toolkit`](https://github.com/go-widgets/toolkit) | v0.364.0 | [el conjunto de widgets]({{< relref "/rendering/toolkit.md" >}}), temas, layouts, texto | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/toolkit) |
| [`skin`](https://github.com/go-widgets/skin) | v0.3.0 | [widgets descritos como datos]({{< relref "/rendering/skin.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/skin) |
| [`gpupainter`](https://github.com/go-widgets/gpupainter) | v0.4.0 | un `painter.Painter` que graba un fotograma para la GPU (go-gfx/gpu2d): [Metal o WebGPU]({{< relref "/surfaces/browser.md" >}}), idéntico byte a byte al painter de CPU | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/gpupainter) |
| [`shellterm`](https://github.com/go-widgets/shellterm) | v0.1.0 | una terminal orientada a líneas: el `Terminal` del toolkit y un editor de línea delante de un shell que usted aporta | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/shellterm) |
| [`isoicons`](https://github.com/go-widgets/isoicons) | v0.4.0 | paquetes de iconos isométricos (cloud-native, AWS) para los diagramas isométricos del toolkit | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/isoicons) |
| [`mvvm`](https://github.com/go-widgets/mvvm) | v0.15.0 | [observables, comandos, binders, deshacer]({{< relref "/state/mvvm.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvm) |
| [`mvvmtk`](https://github.com/go-widgets/mvvmtk) | v0.15.0 | [binders de una sola llamada para los widgets del toolkit]({{< relref "/state/mvvm.md#mvvmtk-one-call-per-widget" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmtk) |
| [`lineedit`](https://github.com/go-widgets/lineedit) | v0.1.1 | un editor de línea como modelo puro: historial, búsqueda inversa, completado con Tab; solo la biblioteca estándar | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/lineedit) |
| [`vcs`](https://github.com/go-widgets/vcs) | v0.1.0 | en qué estado está cada archivo de un árbol de trabajo, y un registro de controladores; solo la biblioteca estándar. El controlador de git es el módulo anidado `vcs/git` (`git/v0.1.0`) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/vcs) |
| [`data`](https://github.com/go-widgets/data) | v0.4.0 | [registros tipados, consultas, un almacén local o remoto]({{< relref "/state/data.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/data) |
| [`window`](https://github.com/go-widgets/window) | v0.90.0 | [una ventana nativa]({{< relref "/surfaces/native-window.md" >}}) sobre ocho backends | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/window) |
| [`application`](https://github.com/go-widgets/application) | v0.8.1 | [el ciclo de vida de la aplicación]({{< relref "/surfaces/application.md" >}}) por encima de una ventana | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/application) |
| [`tray`](https://github.com/go-widgets/tray) | v0.15.0 | [un icono de bandeja y su menú]({{< relref "/surfaces/application.md#the-tray" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tray) |
| [`webcanvas`](https://github.com/go-widgets/webcanvas) | v0.9.0 | [una escena en un `<canvas>` de navegador]({{< relref "/surfaces/browser.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/webcanvas) |
| [`tui`](https://github.com/go-widgets/tui) | v0.62.0 | [renderizado en terminal y widgets nativos de celdas]({{< relref "/surfaces/terminal.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tui) |
| [`android`](https://github.com/go-widgets/android) | v0.16.0 | [un APK pintado por un proceso sin CGO]({{< relref "/surfaces/android.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/android) |
| [`svg`](https://github.com/go-widgets/svg) | v0.7.0 | [un renderizado como SVG o PNG]({{< relref "/surfaces/snapshots.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/svg) |
| [`mvvmlint`](https://github.com/go-widgets/mvvmlint) | v0.4.2 | [la comprobación MVVM]({{< relref "/guard-rails/mvvmlint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmlint) |
| [`bricolint`](https://github.com/go-widgets/bricolint) | v0.4.1 | [la comprobación de que no haya interfaz dibujada a mano]({{< relref "/guard-rails/bricolint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/bricolint) |
| [`app-template`](https://github.com/go-widgets/app-template) | v0.5.0 | [el punto de partida de una aplicación]({{< relref "/getting-started.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/app-template) |
| [`gallery`](https://github.com/go-widgets/gallery) | v0.7.0 | la [demo en vivo](https://go-widgets.github.io/gallery/) de todas las familias de widgets | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/gallery) |
| [`desktop`](https://github.com/go-widgets/desktop) | v0.20.0 | un shell de escritorio que combina go-freedesktop y go-widgets, nativo y en wasmdesk | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/desktop) |

## Lo que se exige a cada módulo {#what-every-module-is-held-to}

- `CGO_ENABLED=0`: nada de cgo, ni de invocar una herramienta de línea de
  comandos en lugar de una biblioteca.
- Compilación cruzada para amd64, arm64, riscv64, loong64, ppc64le y s390x
  —esta última big-endian, lo que mantiene honesta cada codificación binaria— y,
  cuando procede, `js/wasm`. Las veintiuna bibliotecas y herramientas lo hacen,
  cada una desde que se añadió o desde el 2026-10-10, lo que sea más reciente; los otros dos son aplicaciones de navegador ([Estado]({{< relref "/status.md#what-ci-does-not-cover-yet" >}})).
- Una barrera de cobertura de sentencias en la CI: 100 %, en los veintitrés.
  Cuando un archivo no puede ejecutarse en una prueba (un bucle de
  navegador, un bucle de ejecución nativo), el módulo indica qué archivo queda
  fuera y por qué. Consulte [Estado]({{< relref "/status.md" >}}) para las
  excepciones.
- BSD-3-Clause.
