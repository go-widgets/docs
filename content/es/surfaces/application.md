---
title: "Una aplicación y su bandeja"
linkTitle: "Aplicación y bandeja"
weight: 20
description: "go-widgets/application gestiona el ciclo de vida por encima de una ventana; go-widgets/tray coloca un icono y un menú en la barra de menús o en el área de notificación."
tags: [superficies, aplicación, bandeja]
---

## La aplicación {#the-application}

[`go-widgets/application`](https://github.com/go-widgets/application) es la
capa situada por encima de [`window`]({{< relref "/surfaces/native-window.md" >}}):
abre la ventana, hace girar su bucle, coloca un icono en la bandeja en cuanto
el primer fotograma está listo, pasa a la aplicación la apariencia del sistema
en vivo (oscura o clara, color de acento, fuente del sistema), publica el árbol
de accesibilidad y llama a `onReady` en cuanto el primer fotograma está en
pantalla.

```go
spec := application.Spec{
	Name:       "News Reader",
	Identifier: "com.example.reader",
	Version:    "1.0.0",
	Tray: func() *tray.Menu {
		return tray.NewMenu().Add(
			tray.Item("Refresh", refresh),
			tray.Item("Quit", quit),
		)
	},
}
cfg := application.Config{Title: "News Reader", Width: 1200, Height: 800}
err := application.Run(spec, cfg, handler, func() { log.Println("on screen") })
```

`handler` implementa `Handler` —`Frame`, `Resize`, eventos de ratón, de
desplazamiento y de teclado— y puede implementar cualquiera de las capacidades
opcionales: apariencia, atajos, clics y teclas con modificadores, clic
secundario, menús contextuales, portapapeles, accesibilidad, controles nativos.
El bucle nunca envuelve el manejador; hace una aserción de tipo para cada
capacidad y respeta las que encuentra. Una prueba recorre el paquete en busca de
esas aserciones y falla si la documentación del paquete no las nombra todas.

Una bandeja que no puede colocarse es un error de `Run`, no un silencio:
`errors.Is(err, tray.ErrNoBackend)` dice por qué. Una aplicación que quiere su
ventana pase lo que pase con la bandeja deja `Spec.Tray` a nil y ejecuta una
por su cuenta.

`Run` cierra su ventana cuando el bucle termina, de modo que el «Open» de una
aplicación de bandeja puede volver a llamar a `Run` para obtener una ventana
nueva. Para un host que no es una ventana —un shell de escritorio, una pestaña
del navegador—, `Bind` y `BindScaled` devuelven un `*toolkit.Surface`
conectado al mismo manejador.

## Accesibilidad de los widgets dibujados {#painted-accessibility}

Un manejador que dibuja sus propios widgets del
[toolkit]({{< relref "/rendering/toolkit.md#accessibility" >}}) es un único
rectángulo opaco para la capa de accesibilidad de la plataforma, salvo que
implemente `Accessible`. `application.A11yElementsOf(root)` construye esa
descripción a partir del árbol de widgets (rol, etiqueta, valor y límites de
cada widget, como máximo `MaxA11yElements` = 1000):

```go
func (v *View) A11yElements() []application.A11yElement {
	return application.A11yElementsOf(v.root)
}
```

En macOS, los botones dibujados aparecen entonces en el árbol AX como
`AXButton` con su etiqueta, y `AXPress` sobre uno equivale a un clic; los
controles nativos que el manejador incrusta siguen en el árbol a su lado. Los
back-ends de Windows (UI Automation) y Linux (AT-SPI) consumen la misma
descripción. Un widget deshabilitado se publica como
`A11yElement.Disabled`: `AXEnabled` es false en macOS, `IsEnabled` es false en
UI Automation, y el objeto AT-SPI pierde los estados `ENABLED` y `SENSITIVE`
(una señal `StateChanged` anuncia el cambio). El puente de Windows sigue
rechazando los rectángulos de elementos.

## La bandeja {#the-tray}

Un icono de bandeja es integración con el sistema operativo, no un widget
pintado, así que [`go-widgets/tray`](https://github.com/go-widgets/tray) usa la
API propia de cada plataforma, siempre con `CGO_ENABLED=0`:

| Plataforma | API | A través de |
|---|---|---|
| macOS | `NSStatusItem`, `NSMenu` | purego y el runtime de Objective-C |
| Windows | `Shell_NotifyIcon`, `TrackPopupMenu` | `golang.org/x/sys/windows` |
| Linux | `StatusNotifierItem`, `com.canonical.dbusmenu` | DBus, en Go |

```go
menu := tray.NewMenu().Add(
	tray.Item("Open", open),
	tray.Checkbox("Notifications", true, setNotify),
	tray.SubMenu("Recent", tray.NewMenu().Add(tray.Item("file.txt", nil))),
	tray.Separator(),
	tray.Item("Quit", func() { t.Quit() }),
)
t := tray.New(iconPNG).SetTooltip("My App").SetMenu(menu)
t.Run() // blocks on the platform's loop until Quit
```

Un programa que ya ejecuta el bucle de una ventana no puede ceder a `Run` su
hilo principal: `Attach` coloca el mismo icono y retorna, y `Quit` lo retira.

Los backends nativos están activados por defecto; no hay ninguna etiqueta de
compilación que recordar. En una plataforma sin ninguno, `Run` devuelve
`ErrNoBackend`: la diferencia entre «aquí no hay bandeja» y una bandeja que en
silencio no hace nada. Una prueba o un servicio sin interfaz pasa
`WithBackend(tray.NewHeadless())`.

### Notificaciones

`(*Tray).Notify(tray.Notification{Title, Body, OnClick})` lanza una notificación nativa del sistema: UNUserNotificationCenter en macOS, `org.freedesktop.Notifications` en Linux, un globo de Shell_NotifyIcon en Windows. El título tiene como máximo 63 runas y el cuerpo 255 (`MaxNotificationTitle`, `MaxNotificationBody`); un título vacío da `ErrEmptyTitle`. Los errores nombran la causa: `ErrNoNotifier` (sin back-end), `ErrNotRunning`, `ErrNotBundled` (macOS solo entrega para un paquete de aplicación) y `ErrNotAuthorized` (el usuario la rechazó). `tray.NewHeadless()` registra lo enviado en `Headless.Notified`, para las pruebas.

## Abrir un enlace

`application.OpenURL(rawURL)` abre una URL `http` o `https` en el navegador del usuario: NSWorkspace en macOS, ShellExecuteW en Windows, el portal OpenURI y luego `xdg-open` en Linux. La URL necesita un host, un puerto (si lo hay) entre 1 y 65535 y como máximo 8 KiB; cualquier otra cosa se rechaza con `ErrURLScheme` o `ErrURLInvalid`, y una plataforma sin lanzador devuelve `ErrOpenURLUnsupported`.
