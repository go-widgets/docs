---
title: "Módulos"
weight: 50
description: "Los diecinueve módulos de go-widgets, para qué sirve cada uno y dónde están su código fuente y su referencia de API."
tags: [módulos]
---

La referencia de API la genera a partir del código fuente
[pkg.go.dev](https://pkg.go.dev/), que sigue las etiquetas publicadas; estas
páginas explican, y no la copian. La etiqueta es la más reciente en el momento
de escribir; la autoridad es la lista de etiquetas del propio repositorio.

| Módulo | Etiqueta | Qué es | Referencia |
|---|---|---|---|
| [`painter`](https://github.com/go-widgets/painter) | v0.15.0 | [la costura de dibujo]({{< relref "/rendering/painter.md" >}}): painters de píxeles y de celdas | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/painter) |
| [`toolkit`](https://github.com/go-widgets/toolkit) | v0.328.0 | [el conjunto de widgets]({{< relref "/rendering/toolkit.md" >}}), temas, layouts, texto | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/toolkit) |
| [`skin`](https://github.com/go-widgets/skin) | v0.2.0 | [widgets descritos como datos]({{< relref "/rendering/skin.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/skin) |
| [`isoicons`](https://github.com/go-widgets/isoicons) | v0.3.0 | paquetes de iconos isométricos (cloud-native, AWS) para los diagramas isométricos del toolkit | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/isoicons) |
| [`mvvm`](https://github.com/go-widgets/mvvm) | v0.13.0 | [observables, comandos, binders, deshacer]({{< relref "/state/mvvm.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvm) |
| [`mvvmtk`](https://github.com/go-widgets/mvvmtk) | v0.14.1 | [binders de una sola llamada para los widgets del toolkit]({{< relref "/state/mvvm.md#mvvmtk-one-call-per-widget" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmtk) |
| [`data`](https://github.com/go-widgets/data) | v0.3.0 | [registros tipados, consultas, un almacén local o remoto]({{< relref "/state/data.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/data) |
| [`window`](https://github.com/go-widgets/window) | v0.86.1 | [una ventana nativa]({{< relref "/surfaces/native-window.md" >}}) sobre ocho backends | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/window) |
| [`application`](https://github.com/go-widgets/application) | v0.7.0 | [el ciclo de vida de la aplicación]({{< relref "/surfaces/application.md" >}}) por encima de una ventana | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/application) |
| [`tray`](https://github.com/go-widgets/tray) | v0.14.0 | [un icono de bandeja y su menú]({{< relref "/surfaces/application.md#the-tray" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tray) |
| [`webcanvas`](https://github.com/go-widgets/webcanvas) | v0.2.0 | [una escena en un `<canvas>` de navegador]({{< relref "/surfaces/browser.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/webcanvas) |
| [`tui`](https://github.com/go-widgets/tui) | v0.61.0 | [renderizado en terminal y widgets nativos de celdas]({{< relref "/surfaces/terminal.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tui) |
| [`android`](https://github.com/go-widgets/android) | v0.15.0 | [un APK pintado por un proceso sin CGO]({{< relref "/surfaces/android.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/android) |
| [`svg`](https://github.com/go-widgets/svg) | v0.6.0 | [un renderizado como SVG o PNG]({{< relref "/surfaces/snapshots.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/svg) |
| [`mvvmlint`](https://github.com/go-widgets/mvvmlint) | v0.4.0 | [la comprobación MVVM]({{< relref "/guard-rails/mvvmlint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmlint) |
| [`bricolint`](https://github.com/go-widgets/bricolint) | v0.4.0 | [la comprobación de que no haya interfaz dibujada a mano]({{< relref "/guard-rails/bricolint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/bricolint) |
| [`app-template`](https://github.com/go-widgets/app-template) | v0.4.0 | [el punto de partida de una aplicación]({{< relref "/getting-started.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/app-template) |
| [`gallery`](https://github.com/go-widgets/gallery) | v0.6.0 | la [demo en vivo](https://go-widgets.github.io/gallery/) de todas las familias de widgets | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/gallery) |
| [`desktop`](https://github.com/go-widgets/desktop) | v0.19.0 | un shell de escritorio que combina go-freedesktop y go-widgets, nativo y en wasmdesk | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/desktop) |

## Lo que se exige a cada módulo {#what-every-module-is-held-to}

- `CGO_ENABLED=0`: nada de cgo, ni de invocar una herramienta de línea de
  comandos en lugar de una biblioteca.
- Compilación cruzada para amd64, arm64, riscv64, loong64, ppc64le y s390x
  —esta última big-endian, lo que mantiene honesta cada codificación binaria— y,
  cuando procede, `js/wasm`. Las diecisiete bibliotecas y herramientas lo hacen
  desde el 2026-10-10; los otros dos son aplicaciones de navegador ([Estado]({{< relref "/status.md#what-ci-does-not-cover-yet" >}})).
- Una barrera de cobertura de sentencias en la CI: 100 %, en los diecinueve.
  Cuando un archivo no puede ejecutarse en una prueba (un bucle de
  navegador, un bucle de ejecución nativo), el módulo indica qué archivo queda
  fuera y por qué. Consulte [Estado]({{< relref "/status.md" >}}) para las
  excepciones.
- BSD-3-Clause.
