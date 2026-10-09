---
title: "go-widgets"
linkTitle: "Home"
type: docs
cascade:
  type: docs
description: "A pure-Go widget toolkit and every surface it paints on: a native window on X11, Wayland, macOS and Windows, a browser canvas, a terminal, an Android APK."
---

**One widget tree, painted by Go, on every surface Go reaches.** A native
window on X11, Wayland, macOS and Windows; a `<canvas>` in a browser tab; a
terminal; an Android APK. No C toolchain, no webview, no system widget set:
every module builds with `CGO_ENABLED=0`, and every pixel is drawn by the
toolkit.

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

That is
[`cmd/windowdemo`](https://github.com/go-widgets/window/blob/main/cmd/windowdemo/main.go),
shortened. The same `box` drawn in a browser, a terminal or a PNG is the same
code with a different surface underneath it.

## Why one toolkit draws everything

A widget that calls the platform's own button is as portable as the least
capable platform it targets, and looks different on each. go-widgets goes the
other way: a widget's `Draw` only ever talks to a
[`painter.Painter`]({{< relref "/rendering/painter.md" >}}), a handful of
primitives (fill a rectangle, stroke one, set a pixel, draw text). Whether
those calls become RGBA pixels or terminal cells is the back-end's business,
not the widget's.

What that costs is said plainly: no native look by default, and no free
accessibility — the toolkit publishes its own
[accessibility tree]({{< relref "/rendering/toolkit.md#accessibility" >}})
to each platform. What it buys is one widget set, tested once, with the same
behaviour everywhere, and a binary that cross-compiles from any machine to
any of them.

## Where to go next

| | |
|---|---|
| [A first application]({{< relref "/getting-started.md" >}}) | from `app-template`, or from a bare window |
| [How a widget reaches the screen]({{< relref "/rendering/_index.md" >}}) | the painter seam, the widget set, skins |
| [The painter seam]({{< relref "/rendering/painter.md" >}}) | the primitives every widget is drawn with |
| [The toolkit]({{< relref "/rendering/toolkit.md" >}}) | 160 widget types, themes, layouts, text, accessibility |
| [Skins]({{< relref "/rendering/skin.md" >}}) | a widget's parts and states in a data file |
| [State lives in a view-model]({{< relref "/state/_index.md" >}}) | why widget fields are not assigned by hand |
| [MVVM]({{< relref "/state/mvvm.md" >}}) | observables, commands, binders, undo |
| [The data spine]({{< relref "/state/data.md" >}}) | typed records, queries, a local or remote store |
| [Where it paints]({{< relref "/surfaces/_index.md" >}}) | every back-end and how it is chosen |
| [A native window]({{< relref "/surfaces/native-window.md" >}}) | X11, Wayland, Cocoa, Win32, GTK4, Android, wasmbox |
| [An application and its tray]({{< relref "/surfaces/application.md" >}}) | lifecycle, appearance, a menu-bar icon |
| [A browser tab]({{< relref "/surfaces/browser.md" >}}) | a plain `<canvas>`, or a wasmdesk window |
| [A terminal]({{< relref "/surfaces/terminal.md" >}}) | cell-native widgets and an interactive runner |
| [Android]({{< relref "/surfaces/android.md" >}}) | a real APK painted by a CGO-free process |
| [Snapshots]({{< relref "/surfaces/snapshots.md" >}}) | a widget as an SVG or a PNG |
| [Rules CI enforces]({{< relref "/guard-rails/_index.md" >}}) | `mvvmlint` and `bricolint` |
| [Modules]({{< relref "/modules.md" >}}) | all nineteen, with their sources and references |
| [Status]({{< relref "/status.md" >}}) | what is verified where, and what is not done |

## Licence

BSD-3-Clause.
