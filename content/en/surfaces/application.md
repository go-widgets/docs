---
title: "An application and its tray"
linkTitle: "Application and tray"
weight: 20
description: "go-widgets/application owns the lifecycle above a window; go-widgets/tray puts an icon and a menu in the menu bar or notification area."
tags: [surfaces, application, tray]
---

## The application

[`go-widgets/application`](https://github.com/go-widgets/application) is the
layer above [`window`]({{< relref "/surfaces/native-window.md" >}}): it opens
the window, drives its loop, attaches a tray icon once the first frame is up,
passes the live system appearance (dark or light, accent colour, system font)
to the application, publishes the accessibility tree, and calls `onReady`
once the first frame is on screen.

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

`handler` implements `Handler` — `Frame`, `Resize`, mouse, scroll and key
events — and may implement any of the optional capabilities: appearance,
shortcuts, modified clicks and keys, secondary click, context menus,
clipboard, accessibility, native controls. The loop never wraps the handler;
it type-asserts each capability and honours the ones it finds. A test walks
the package for those assertions and fails if the package documentation does
not name every one.

A tray that cannot be attached is `Run`'s error, not a silence:
`errors.Is(err, tray.ErrNoBackend)` says why. An application that wants its
window whatever happens to the tray leaves `Spec.Tray` nil and runs one
itself.

`Run` closes its window when the loop returns, so a tray application's
"Open" can call `Run` again for a fresh window. For a host that is not a
window — a desktop shell, a browser tab — `Bind` and `BindScaled` return a
`*toolkit.Surface` wired to the same handler.

## The tray

A tray icon is OS integration, not a painted widget, so
[`go-widgets/tray`](https://github.com/go-widgets/tray) drives each platform's
own API, still with `CGO_ENABLED=0`:

| Platform | API | Through |
|---|---|---|
| macOS | `NSStatusItem`, `NSMenu` | purego and the Objective-C runtime |
| Windows | `Shell_NotifyIcon`, `TrackPopupMenu` | `golang.org/x/sys/windows` |
| Linux | `StatusNotifierItem`, `com.canonical.dbusmenu` | DBus, in Go |

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

A program already running a window's loop cannot give `Run` its main thread:
`Attach` puts the same icon up and returns, and `Quit` takes it down.

The native back-ends are on by default; there is no build tag to remember.
On a platform with none, `Run` returns `ErrNoBackend` — the difference
between "there is no tray here" and a tray that silently does nothing. A test
or a headless service passes `WithBackend(tray.NewHeadless())`.
