---
title: "go-widgets"
linkTitle: "Startseite"
type: docs
cascade:
  type: docs
description: "Ein Widget-Toolkit in reinem Go und jede Oberfläche, auf die es zeichnet: ein natives Fenster unter X11, Wayland, macOS und Windows, ein Browser-Canvas, ein Terminal, eine Android-APK."
---

**Ein Widget-Baum, von Go gezeichnet, auf jeder Oberfläche, die Go erreicht.** Ein natives
Fenster unter X11, Wayland, macOS und Windows; ein `<canvas>` in einem Browser-Tab; ein
Terminal; eine Android-APK. Keine C-Toolchain, keine Webview, kein System-Widget-Satz:
Jedes Modul baut mit `CGO_ENABLED=0`, und jedes Pixel zeichnet das
Toolkit.

```go
w, err := window.Open(window.Config{Title: "Hello"})
if err != nil {
	return err
}
defer w.Close()

box := toolkit.NewVBox()
box.Append(toolkit.NewLabel("Hello from go-widgets"))
box.Append(toolkit.NewButton("Click me", func() { /* ... */ }))
return w.Run(box) // an X11 or Wayland window, an NSWindow, a Win32 window
```

Das ist
[`cmd/windowdemo`](https://github.com/go-widgets/window/blob/main/cmd/windowdemo/main.go),
gekürzt. Dieselbe `box` in einem Browser, einem Terminal oder einem PNG gezeichnet ist derselbe
Code mit einer anderen Oberfläche darunter.

## Warum ein Toolkit alles zeichnet {#why-one-toolkit-draws-everything}

Ein Widget, das den plattformeigenen Button aufruft, ist so portabel wie die am wenigsten
fähige Plattform, auf die es zielt, und sieht auf jeder anders aus. go-widgets geht den
umgekehrten Weg: Das `Draw` eines Widgets spricht immer nur mit einem
[`painter.Painter`]({{< relref "/rendering/painter.md" >}}), einer Handvoll
Primitive (ein Rechteck füllen, eines umranden, ein Pixel setzen, Text zeichnen). Ob
aus diesen Aufrufen RGBA-Pixel oder Terminalzellen werden, ist Sache des Back-Ends,
nicht des Widgets.

Was das kostet, wird offen gesagt: standardmäßig kein natives Aussehen und keine Barrierefreiheit
geschenkt – das Toolkit veröffentlicht seinen eigenen
[Barrierefreiheitsbaum]({{< relref "/rendering/toolkit.md#accessibility" >}})
an jede Plattform. Was es dafür bekommt, ist ein einziger Widget-Satz, einmal getestet, mit demselben
Verhalten überall, und ein Binary, das sich von jeder Maschine aus für jede dieser
Plattformen cross-kompilieren lässt.

## Wie es weitergeht {#where-to-go-next}

| | |
|---|---|
| [Eine erste Anwendung]({{< relref "/getting-started.md" >}}) | aus `app-template` oder aus einem nackten Fenster |
| [Wie ein Widget auf den Bildschirm kommt]({{< relref "/rendering/_index.md" >}}) | die Painter-Schnittstelle, der Widget-Satz, Skins |
| [Die Painter-Schnittstelle]({{< relref "/rendering/painter.md" >}}) | die Primitive, mit denen jedes Widget gezeichnet wird |
| [Das Toolkit]({{< relref "/rendering/toolkit.md" >}}) | 160 Widget-Typen, Themes, Layouts, Text, Barrierefreiheit |
| [Skins]({{< relref "/rendering/skin.md" >}}) | die Teile und Zustände eines Widgets in einer Datendatei |
| [Zustand lebt in einem View-Model]({{< relref "/state/_index.md" >}}) | warum Widget-Felder nicht von Hand zugewiesen werden |
| [MVVM]({{< relref "/state/mvvm.md" >}}) | Observables, Commands, Binder, Rückgängig |
| [Das Daten-Rückgrat]({{< relref "/state/data.md" >}}) | typisierte Datensätze, Abfragen, ein lokaler oder entfernter Speicher |
| [Wohin es zeichnet]({{< relref "/surfaces/_index.md" >}}) | jedes Back-End und wie es ausgewählt wird |
| [Ein natives Fenster]({{< relref "/surfaces/native-window.md" >}}) | X11, Wayland, Cocoa, Win32, GTK4, Android, wasmbox, ein Browser-Tab |
| [Eine Anwendung und ihr Tray]({{< relref "/surfaces/application.md" >}}) | Lebenszyklus, Erscheinungsbild, ein Menüleisten-Symbol |
| [Ein Browser-Tab]({{< relref "/surfaces/browser.md" >}}) | ein einfaches `<canvas>` oder ein wasmdesk-Fenster |
| [Ein Terminal]({{< relref "/surfaces/terminal.md" >}}) | zellennative Widgets und ein interaktiver Runner |
| [Android]({{< relref "/surfaces/android.md" >}}) | eine echte APK, gezeichnet von einem CGO-freien Prozess |
| [Snapshots]({{< relref "/surfaces/snapshots.md" >}}) | ein Widget als SVG oder PNG |
| [Regeln, die die CI durchsetzt]({{< relref "/guard-rails/_index.md" >}}) | `mvvmlint` und `bricolint` |
| [Module]({{< relref "/modules.md" >}}) | alle neunzehn, mit ihren Quellen und Referenzen |
| [Stand]({{< relref "/status.md" >}}) | was wo geprüft ist und was noch fehlt |

## Lizenz {#licence}

BSD-3-Clause.
