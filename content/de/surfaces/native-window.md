---
title: "Ein natives Fenster"
linkTitle: "Natives Fenster"
weight: 10
description: "go-widgets/window: eine einzige Open/Run-API über X11, Wayland, Cocoa, Win32, GTK4, Android und wasmbox, alle mit CGO_ENABLED=0."
tags: [oberflächen, fenster, x11, wayland, macos, windows]
---

[`go-widgets/window`](https://github.com/go-widgets/window) öffnet ein echtes Fenster
und führt darin einen Widget-Baum aus: Layout, Zeichnen, Darstellen, Eingaben in
`toolkit.Event` übersetzen, wiederholen, bis das Fenster geschlossen wird.

```go
w, err := window.Open(window.Config{Title: "Demo"})
if err != nil {
	return err
}
defer w.Close()
return w.Run(root) // root is any toolkit.Widget
```

`Config` nimmt außerdem die Anfangsgröße in **logischen Punkten** (null bittet um einen
lesbaren Standardwert), Instanz und Klasse für `WM_CLASS`, ein `Theme` und eine
`RenderScale`.

## Wie Open auswählt {#how-open-chooses}

| Plattform und Umgebung | Back-End | Wie es auf den Bildschirm kommt |
|---|---|---|
| Linux, `$GO_WIDGETS_GTK` gesetzt | GTK4 | GTK besitzt das Fenster; der Framebuffer ist ein `GtkPicture`, und native Bedienelemente sind echte GTK-Widgets darüber. Benötigt die libgtk-4-Laufzeitbibliothek. |
| Linux, `$WAYLAND_DISPLAY` gesetzt | Wayland | xdg-shell über den Unix-Socket des Compositors |
| Linux, sonst `$DISPLAY` | X11 | das X11-Kernprotokoll über den Unix-Socket, mit MIT-SHM |
| macOS | Cocoa | `NSWindow` und `NSView` über [go-macos/objc](https://github.com/go-macos/objc) (purego) |
| Windows | Win32 | ein Top-Level-`HWND` über Syscalls von user32 und gdi32, `StretchDIBits` |
| Android, `$GW_ANDROID_SOCKET` gesetzt | Android-Host | ein gerahmtes Protokoll zum [Java-Host]({{< relref "/surfaces/android.md" >}}), Pixel in einem gemeinsamen memfd |
| Android, sonst | Wayland oder X11 | eine Shell unter Termux hat trotzdem einen Display-Server, mit dem sie sich verbinden kann |
| `js/wasm` | wasmbox | ein Client des [wasmdesk-Compositors]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}}) |
| alles andere | — | `window.ErrUnsupported`, damit ein Cross-Build trotzdem kompiliert |

Jedes Back-End ist `CGO_ENABLED=0`. Das X11-Back-End ist das Kernprotokoll, von Grund auf
über dem Socket geschrieben – kein Xlib, kein XCB –, mit beiden Byte-Reihenfolgen, dem
Xauthority-Cookie, Keysym-Zuordnung, `PutImage` in Kacheln unterhalb der
Anfragegröße des Servers und dem schnellen MIT-SHM-Pfad, wobei das gemeinsame Segment über
`SCM_RIGHTS` übergeben wird. Windows wird über die prozesseigenen DLLs mit
`syscall.NewLazyDLL` und einer Fensterprozedur per `syscall.NewCallback` erreicht; macOS
über die Objective-C-Runtime mit purego.

Der plattformunabhängige Teil jedes Back-Ends – Ereigniszuordnung, Koordinaten-
mathematik, Pixel-Packing, Schadensrechtecke – liegt in einem Codec, der zu 100 % abgedeckt ist,
auf jedem Betriebssystem. Die dünne Plattformanbindung ist live nachgewiesen: siehe
[Stand]({{< relref "/status.md" >}}).

## Nur was sich geändert hat {#only-what-changed}

Eine Wurzel, die `DamageRenderer` implementiert (wie `toolkit/scene.HostRoot`),
meldet die Rechtecke, die sie neu gezeichnet hat, und die Back-Ends für Cocoa, Win32 und wasmbox
stellen nur diese dar.

## HiDPI {#hidpi}

Standard ist ein Framebuffer-Pixel pro logischem Punkt: Die UI wird in einer
lesbaren Größe angeordnet und gezeichnet, und der Compositor skaliert sie hoch. Unter Windows
deklariert das Fenster DPI-Bewusstsein pro Monitor, und das Betriebssystem skaliert den logischen
Frame auf den physischen Client-Bereich. `RenderScale` fordert einen Framebuffer in
der echten Auflösung des Bildschirms an, was nur für eine Wurzel korrekt ist, die in
Gerätepixeln anordnet.

## Bildschirme, Anzeigen und Ausblenden {#screens-and-showing-and-hiding}

`window.Screens()` listet die angeschlossenen Displays auf, das primäre zuerst, in logischen
Punkten und ohne die Leisten des Desktops, und kann vor `Open` aufgerufen werden.
Der Name eines Bildschirms ist der des Panels selbst (`"DELL U2720Q"`, aus seiner EDID), mit
Rückfall auf den Anschluss (`"HDMI-1"`), wenn es keinen veröffentlicht – und auch, wenn zwei
angeschlossene Panels denselben veröffentlichen, denn ein Name, der sie nicht
unterscheiden kann, ist kein Name.

`window.Show`, `Hide` und `Raise` nehmen ein offenes Fenster vom Bildschirm, ohne es
zu schließen, und holen es zurück – was eine [Tray-Anwendung]({{< relref "/surfaces/application.md#the-tray" >}})
braucht.

| Back-End | Show | Hide | Raise |
|---|---|---|---|
| X11 | `MapWindow` | ICCCM-Withdraw, sodass es die Taskleiste verlässt | Map, Raise, `_NET_ACTIVE_WINDOW` |
| Wayland | erneuter Commit | Null-Puffer und Commit | `ErrNotSupported`: xdg-shell hat keine solche Anfrage |
| macOS | `orderFront:` | `orderOut:` | die App aktivieren, `makeKeyAndOrderFront:` |
| Windows | `SW_SHOWNA` | `SW_HIDE` | `SW_RESTORE` und `SetForegroundWindow` |
| GTK, Android, wasmbox | `ErrNotSupported` | `ErrNotSupported` | `ErrNotSupported` |

Den Fokus gewährt die Plattform: Unter einem Schutz gegen Fokusraub kann `Raise`
das Fenster möglicherweise nur als aufmerksamkeitsbedürftig markieren und kann das nicht erkennen.
