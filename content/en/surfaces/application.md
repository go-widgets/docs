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

## The menu bar {#menu-bar}

`application.Config.Menu` takes the same `tray.Menu` a tray icon uses and
installs it as the application's menu bar; `application.SetMenu` changes it
later, from any goroutine, and reinstalls it only when the menu changed.
`tray.EditMenu()` is the standard Edit menu (Undo, Cut, Copy, Paste, Select
All), so a text field gets its clipboard commands in the menu bar.

- macOS: the menu at the top of the screen, with the application menu first.
- Windows: the window's menu bar; key equivalents are shown as accelerator text.
- X11, Wayland, GTK, wasm and Android: `window.ErrNotSupported`, ignored by
  `Run`. GTK is a gap: the GTK binding has no menu model yet.

## Accessibility of painted widgets {#painted-accessibility}

A handler that paints its own [toolkit]({{< relref "/rendering/toolkit.md#accessibility" >}})
widgets is one opaque rectangle to the platform's accessibility layer, unless
it implements `Accessible`. `application.A11yElementsOf(root)` builds that
description from the widget tree (role, label, value and bounds of each
widget, at most `MaxA11yElements` = 1000):

```go
func (v *View) A11yElements() []application.A11yElement {
	return application.A11yElementsOf(v.root)
}
```

On macOS the painted buttons then appear in the AX tree as `AXButton` with
their label, and `AXPress` on one is a click on it; the native controls the
handler embeds stay in the tree beside them. The Windows (UI Automation) and
Linux (AT-SPI) back-ends consume the same description. An element's disabled
state is not published yet, and the Windows bridge declines element
rectangles.

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

### Notifications

`(*Tray).Notify(tray.Notification{Title, Body, OnClick})` raises a native
system notification: UNUserNotificationCenter on macOS,
`org.freedesktop.Notifications` on Linux, a Shell_NotifyIcon balloon on
Windows. The title is at most 63 runes and the body 255
(`MaxNotificationTitle`, `MaxNotificationBody`); an empty title is
`ErrEmptyTitle`. The errors name the cause: `ErrNoNotifier` (no back-end),
`ErrNotRunning`, `ErrNotBundled` (macOS delivers only for an app bundle) and
`ErrNotAuthorized` (the user refused). `tray.NewHeadless()` records what was
sent in `Headless.Notified`, for tests.

## Opening a link

`application.OpenURL(rawURL)` opens an `http` or `https` URL in the user's
browser: NSWorkspace on macOS, ShellExecuteW on Windows, the OpenURI portal
then `xdg-open` on Linux. The URL needs a host, a port (if any) in 1-65535
and at most 8 KiB; anything else is refused with `ErrURLScheme` or
`ErrURLInvalid`, and a platform without a launcher returns
`ErrOpenURLUnsupported`.
