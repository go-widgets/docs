---
title: "Eine Anwendung und ihr Tray"
linkTitle: "Anwendung und Tray"
weight: 20
description: "go-widgets/application besitzt den Lebenszyklus oberhalb eines Fensters; go-widgets/tray setzt ein Symbol und ein Menü in die Menüleiste oder den Infobereich."
tags: [oberflächen, anwendung, tray]
---

## Die Anwendung {#the-application}

[`go-widgets/application`](https://github.com/go-widgets/application) ist die
Schicht oberhalb von [`window`]({{< relref "/surfaces/native-window.md" >}}): Sie öffnet
das Fenster, treibt seine Schleife, hängt ein Tray-Symbol an, sobald der erste Frame steht,
reicht das aktuelle Erscheinungsbild des Systems (dunkel oder hell, Akzentfarbe, Systemschrift)
an die Anwendung weiter, veröffentlicht den Barrierefreiheitsbaum und ruft `onReady` auf,
sobald der erste Frame auf dem Bildschirm ist.

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

`handler` implementiert `Handler` – `Frame`, `Resize`, Maus-, Scroll- und Tastatur-
Ereignisse – und kann jede der optionalen Fähigkeiten implementieren: Erscheinungsbild,
Tastenkürzel, Klicks und Tasten mit Modifikatoren, Sekundärklick, Kontextmenüs,
Zwischenablage, Barrierefreiheit, native Bedienelemente. Die Schleife umhüllt den Handler nie;
sie macht für jede Fähigkeit eine Typzusicherung und beachtet die, die sie findet. Ein Test durchläuft
das Paket nach diesen Zusicherungen und schlägt fehl, wenn die Paketdokumentation
nicht jede einzelne nennt.

Ein Tray, das nicht angehängt werden kann, ist ein Fehler von `Run`, kein Schweigen:
`errors.Is(err, tray.ErrNoBackend)` sagt, warum. Eine Anwendung, die ihr
Fenster will, egal was mit dem Tray geschieht, lässt `Spec.Tray` auf nil und betreibt selbst
eines.

`Run` schließt sein Fenster, wenn die Schleife zurückkehrt, sodass das „Open" einer Tray-Anwendung
`Run` erneut aufrufen kann, um ein frisches Fenster zu bekommen. Für einen Host, der kein
Fenster ist – eine Desktop-Shell, ein Browser-Tab –, geben `Bind` und `BindScaled` eine
`*toolkit.Surface` zurück, die mit demselben Handler verdrahtet ist.

## Die Menüleiste {#menu-bar}

`application.Config.Menu` nimmt dasselbe `tray.Menu` wie ein Tray-Symbol und
installiert es als Menüleiste der Anwendung; `application.SetMenu` ändert sie
später, aus jeder Goroutine, und installiert sie nur neu, wenn sich das Menü
geändert hat. `tray.EditMenu()` ist das Standardmenü Bearbeiten (Widerrufen,
Ausschneiden, Kopieren, Einsetzen, Alles auswählen).

- macOS: das Menü am oberen Bildschirmrand, das Anwendungsmenü zuerst.
- Windows: die Menüleiste des Fensters; Tastenkürzel erscheinen als Text.
- X11, Wayland, GTK, wasm und Android: `window.ErrNotSupported`, von `Run`
  ignoriert. GTK ist eine Lücke: die GTK-Anbindung hat noch kein Menümodell.

## Barrierefreiheit gezeichneter Widgets {#painted-accessibility}

Ein Handler, der eigene [Toolkit]({{< relref "/rendering/toolkit.md#accessibility" >}})-Widgets
zeichnet, ist für die Zugänglichkeitsschicht der Plattform ein einziges
undurchsichtiges Rechteck, sofern er nicht `Accessible` implementiert.
`application.A11yElementsOf(root)` erzeugt diese Beschreibung aus dem
Widget-Baum (Rolle, Beschriftung, Wert und Grenzen jedes Widgets, höchstens
`MaxA11yElements` = 1000):

```go
func (v *View) A11yElements() []application.A11yElement {
	return application.A11yElementsOf(v.root)
}
```

Unter macOS erscheinen die gezeichneten Schaltflächen dann im AX-Baum als
`AXButton` mit ihrer Beschriftung, und `AXPress` darauf ist ein Klick; die
nativen Steuerelemente, die der Handler einbettet, bleiben daneben im Baum. Die
Back-Ends für Windows (UI Automation) und Linux (AT-SPI) verwenden dieselbe
Beschreibung. Ein deaktiviertes Widget wird als
`A11yElement.Disabled` veröffentlicht: `AXEnabled` ist unter macOS false,
`IsEnabled` ist in UI Automation false, und das AT-SPI-Objekt verliert die
Zustände `ENABLED` und `SENSITIVE` (ein `StateChanged`-Signal meldet die
Änderung). Die Windows-Brücke lehnt weiterhin Element-Rechtecke ab.

## Das Tray {#the-tray}

Ein Tray-Symbol ist Betriebssystemintegration, kein gezeichnetes Widget, deshalb
steuert [`go-widgets/tray`](https://github.com/go-widgets/tray) die eigene API jeder Plattform,
weiterhin mit `CGO_ENABLED=0`:

| Plattform | API | Über |
|---|---|---|
| macOS | `NSStatusItem`, `NSMenu` | purego und die Objective-C-Runtime |
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

Ein Programm, das bereits die Schleife eines Fensters betreibt, kann `Run` seinen Hauptthread nicht überlassen:
`Attach` zeigt dasselbe Symbol an und kehrt zurück, und `Quit` entfernt es wieder.

Die nativen Back-Ends sind standardmäßig aktiv; es gibt kein Build-Tag, an das man denken müsste.
Auf einer Plattform ohne Back-End gibt `Run` `ErrNoBackend` zurück – der Unterschied
zwischen „hier gibt es kein Tray" und einem Tray, das stillschweigend nichts tut. Ein Test
oder ein Headless-Dienst übergibt `WithBackend(tray.NewHeadless())`.

### Benachrichtigungen

`(*Tray).Notify(tray.Notification{Title, Body, OnClick})` löst eine native Systembenachrichtigung aus: UNUserNotificationCenter unter macOS, `org.freedesktop.Notifications` unter Linux, ein Shell_NotifyIcon-Ballon unter Windows. Der Titel hat höchstens 63 Runen und der Text 255 (`MaxNotificationTitle`, `MaxNotificationBody`); ein leerer Titel ergibt `ErrEmptyTitle`. Die Fehler benennen die Ursache: `ErrNoNotifier` (kein Back-End), `ErrNotRunning`, `ErrNotBundled` (macOS liefert nur für ein App-Bundle) und `ErrNotAuthorized` (der Benutzer hat abgelehnt). `tray.NewHeadless()` hält in `Headless.Notified` fest, was gesendet wurde, für Tests.

## Einen Link öffnen

`application.OpenURL(rawURL)` öffnet eine `http`- oder `https`-URL im Browser des Benutzers: NSWorkspace unter macOS, ShellExecuteW unter Windows, das OpenURI-Portal und danach `xdg-open` unter Linux. Die URL braucht einen Host, einen Port (falls vorhanden) zwischen 1 und 65535 und höchstens 8 KiB; alles andere wird mit `ErrURLScheme` oder `ErrURLInvalid` abgelehnt, und eine Plattform ohne Launcher liefert `ErrOpenURLUnsupported`.
