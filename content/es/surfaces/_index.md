---
title: "Dónde pinta"
linkTitle: "Superficies"
weight: 30
description: "Todas las superficies en las que se puede mostrar un árbol de go-widgets, y qué decide entre ellas."
tags: [superficies, backends]
---

| Superficie | Módulo | Painter |
|---|---|---|
| [Una ventana nativa]({{< relref "/surfaces/native-window.md" >}}) en X11, Wayland, macOS, Windows | `window` | píxeles |
| [Una aplicación]({{< relref "/surfaces/application.md" >}}) con un icono en la bandeja y la apariencia del sistema | `application`, `tray` | píxeles |
| [Una pestaña del navegador]({{< relref "/surfaces/browser.md" >}}): un simple `<canvas>` | `webcanvas`, o `window` | píxeles |
| [Un escritorio en el navegador]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}}): una ventana del compositor wasmdesk | `window` (wasmbox) | píxeles |
| [Un terminal]({{< relref "/surfaces/terminal.md" >}}) | `tui` | celdas |
| [Un APK de Android]({{< relref "/surfaces/android.md" >}}) | `android`, `window` | píxeles |
| [Una imagen]({{< relref "/surfaces/snapshots.md" >}}): SVG o PNG | `svg` | píxeles |

El árbol de widgets no cambia de una a otra. Lo que cambia es quién posee el
búfer y quién entrega la entrada —un servidor gráfico, un navegador, un
terminal, un host Java—, y cada backend convierte esa entrada en el mismo
`toolkit.Event`.

## Elegido según lo que hay {#chosen-by-what-is-there}

`window.Open` no recibe ningún argumento de backend. Examina el entorno —la
plataforma para la que se compiló, `$WAYLAND_DISPLAY`, `$DISPLAY`, el socket
que exporta un host Android— y elige el que puede funcionar. Un binario
compilado para Linux abre una ventana Wayland en una sesión Wayland y una
ventana X11 bajo X; el mismo binario de Android se conecta a su host dentro de
un APK y abre una ventana Linux corriente bajo Termux. Consulte
[cómo elige `Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}}).
