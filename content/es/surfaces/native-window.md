---
title: "Una ventana nativa"
linkTitle: "Ventana nativa"
weight: 10
description: "go-widgets/window: una sola API Open/Run sobre X11, Wayland, GTK4, Cocoa, Win32, Android, wasmbox y una pestaña del navegador, todo con CGO_ENABLED=0."
tags: [superficies, ventana, x11, wayland, macos, windows]
---

[`go-widgets/window`](https://github.com/go-widgets/window) abre una ventana
real y ejecuta en ella un árbol de widgets: layout, dibujo, presentación,
traducción de la entrada a `toolkit.Event`, y vuelta a empezar hasta que la
ventana se cierra.

```go
w, err := window.Open(window.Config{Title: "Demo"})
if err != nil {
	return err
}
defer w.Close()
return w.Run(root) // root is any toolkit.Widget
```

`Config` recibe además el tamaño inicial en **puntos lógicos** (cero pide un
tamaño por defecto legible), la instancia y la clase de `WM_CLASS`, un `Theme`
y un `RenderScale`.

## Cómo elige Open {#how-open-chooses}

| Plataforma y entorno | Backend | Cómo llega a la pantalla |
|---|---|---|
| Linux, `$GO_WIDGETS_GTK` definida | GTK4 | GTK posee la ventana; el framebuffer es un `GtkPicture`, y los controles nativos son widgets GTK reales por encima de él. Necesita el runtime de libgtk-4. |
| Linux, `$WAYLAND_DISPLAY` definida | Wayland | xdg-shell sobre el socket unix del compositor |
| Linux, en otro caso `$DISPLAY` | X11 | el protocolo central de X11 sobre el socket unix, con MIT-SHM |
| macOS | Cocoa | `NSWindow` y `NSView` a través de [go-macos/objc](https://github.com/go-macos/objc) (purego) |
| Windows | Win32 | un `HWND` de nivel superior mediante llamadas al sistema de user32 y gdi32, `StretchDIBits` |
| Android, `$GW_ANDROID_SOCKET` definida | Host Android | un protocolo por tramas hacia [el host Java]({{< relref "/surfaces/android.md" >}}), píxeles en un memfd compartido |
| Android, en otro caso | Wayland o X11 | un shell bajo Termux sigue teniendo un servidor gráfico al que conectarse |
| `js/wasm`, dentro de wasmdesk | wasmbox | un cliente del [compositor wasmdesk]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}}) |
| `js/wasm`, una página corriente | pestaña del navegador | un `<canvas>` de la página (`Config.Canvas`, por defecto `"screen"`) a través de `webcanvas`: sin compositor, sin `SharedArrayBuffer`, sin aislamiento entre orígenes |
| cualquier otra cosa | — | `window.ErrUnsupported`, para que una compilación cruzada siga compilando |

Todos los backends son `CGO_ENABLED=0`. El de X11 es el protocolo central
escrito desde cero sobre el socket —sin Xlib, sin XCB—, con ambos órdenes de
bytes, la cookie de Xauthority, el mapeo de keysyms, `PutImage` troceado por
debajo del tamaño máximo de petición del servidor y la vía rápida MIT-SHM, con
el segmento compartido pasado por `SCM_RIGHTS`. A Windows se llega a través de
las propias DLL del proceso con `syscall.NewLazyDLL` y un procedimiento de
ventana `syscall.NewCallback`; a macOS, a través del runtime de Objective-C con
purego.

La parte de cada backend que no depende de la plataforma —mapeo de eventos,
cálculo de coordenadas, empaquetado de píxeles, rectángulos dañados— vive en un
códec cubierto al 100 %, en todos los sistemas operativos. La fina capa de
unión con la plataforma se demuestra en vivo: consulte
[Estado]({{< relref "/status.md" >}}).

## Solo lo que ha cambiado {#only-what-changed}

Una raíz que implementa `DamageRenderer` (como hace `toolkit/scene.HostRoot`)
informa de los rectángulos que ha repintado, y todos los backends salvo GTK4
presentan solo esos: X11 mediante `ShmPutImage` de MIT-SHM, Wayland mediante
el daño de `wl_shm`, Cocoa, Win32, Android, wasmbox y la pestaña del navegador.
El primer fotograma, un cambio de tamaño y un `Expose` de X11 siguen
presentando toda la superficie.

## HiDPI {#hidpi}

Por defecto hay un píxel de framebuffer por punto lógico: la interfaz se
organiza y se pinta a un tamaño legible, y el compositor la sobremuestrea. En
Windows, la ventana declara compatibilidad con DPI por monitor y el sistema
operativo escala el fotograma lógico al área cliente física. `RenderScale` pide
un framebuffer a la resolución real del panel, lo cual solo es correcto para una
raíz que organiza su layout en píxeles del dispositivo.

## Pantallas, y mostrar y ocultar {#screens-and-showing-and-hiding}

`window.Screens()` enumera las pantallas conectadas, la principal primero, en
puntos lógicos y descontando los paneles del escritorio, y puede llamarse antes
de `Open`. El nombre de una pantalla es el del propio panel (`"DELL U2720Q"`,
tomado de su EDID), y recurre al conector (`"HDMI-1"`) cuando el panel no
publica ninguno, y también cuando dos paneles conectados publican el mismo,
porque un nombre que no permite distinguirlos no es un nombre.

`window.Show`, `Hide` y `Raise` retiran de la pantalla una ventana abierta sin
cerrarla y la traen de vuelta: lo que necesita una
[aplicación de bandeja]({{< relref "/surfaces/application.md#the-tray" >}}).

| Backend | Show | Hide | Raise |
|---|---|---|---|
| X11 | `MapWindow` | retirada ICCCM, para que salga de la barra de tareas | mapear, elevar, `_NET_ACTIVE_WINDOW` |
| Wayland | volver a hacer commit | búfer nulo y commit | `ErrNotSupported`: xdg-shell no tiene esa petición |
| macOS | `orderFront:` | `orderOut:` | activar la aplicación, `makeKeyAndOrderFront:` |
| Windows | `SW_SHOWNA` | `SW_HIDE` | `SW_RESTORE` y `SetForegroundWindow` |
| GTK, Android, wasmbox, pestaña del navegador | `ErrNotSupported` | `ErrNotSupported` | `ErrNotSupported` |

Conceder el foco corresponde a la plataforma: con la prevención del robo de
foco, `Raise` puede limitarse a marcar la ventana como necesitada de atención,
y no puede saberlo.
