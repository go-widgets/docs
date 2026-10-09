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
