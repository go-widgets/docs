---
title: "Ein Terminal"
linkTitle: "Terminal"
weight: 40
description: "go-widgets/tui rendert Widgets über den Zellen-Painter: ein Snapshot-Renderer, ein interaktiver Runner und Widgets, die für ein Zeichenraster bemessen sind."
tags: [oberflächen, tui, terminal]
---

[`go-widgets/tui`](https://github.com/go-widgets/tui) macht aus einem Widget-Baum
einen Terminal-Frame, über den [Zellen-Painter]({{< relref "/rendering/painter.md" >}}).
Nur Standardbibliothek – kein `x/term`, kein cgo.

## Ein Frame {#one-frame}

```go
_ = tui.RenderOnce(os.Stdout, widgets, nil) // nil theme: light
```

`RenderOnceSized(w, cols, rows, widgets, theme)` nimmt die Größe explizit entgegen;
`RenderOnce` liest `COLUMNS` und `LINES` und fällt auf 80×24 zurück. Die
Umgebung zu lesen, statt `TIOCGWINSZ` aufzurufen, hält das Paket frei von
plattformspezifischem Code.

## Ein interaktives Programm {#an-interactive-program}

```go
app := tui.NewApp()
app.Root = buildScene()
app.Keys["q"] = func(a *tui.App) { a.Quit() }
app.Keys["Ctrl+C"] = func(a *tui.App) { a.Quit() }
os.Exit(app.Run())
```

`Run` wechselt in den alternativen Bildschirm und den Raw-Modus, leitet Eingaben an `Root` weiter
(globale Tasten-Handler zuerst), folgt `SIGWINCH` und stellt das Terminal immer wieder her –
auch bei einer Panic. Ein Tasten-Handler ruft `Consume` auf, um ein Ereignis von
`Root` fernzuhalten; `InputTarget` schickt jedes nicht konsumierte Ereignis an ein einzelnes Widget, und so
verschluckt eine Befehlspalette Tastatureingaben, solange sie offen ist. `NewFocusRing` gibt einem
Formular Navigation mit Tab und Shift+Tab.

## Widgets, für Zellen bemessen {#widgets-sized-for-cells}

Die meisten Toolkit-Widgets sind in Pixeln entworfen: Ihre Padding-Konstanten zählen unter dem
Zellen-Painter als Zellen und blähen sich auf. Deshalb hat `tui` einen eigenen **zellennativen**
Satz – `TextEditor`, `Entry`, `Button`, `CheckButton`, `RadioButton`, `Scale`,
`ProgressBar`, `Sparkline`, `ListBox`, `TreeView`, `Table`, `MenuBar`,
`Notebook`, `HSplit`, `VSplit`, `Dialog`, `Dropdown`, `Spinner` und weitere –,
in dem eine Glyphe eine Zelle ist. Jedes davon ist weiterhin ein `toolkit.Widget`, sodass dieselbe
Instanz auch in Pixel zeichnet.

`TextEditor` ist ein Code-Editor zum Lesen und Schreiben: Randspalte, Rückgängig und Wiederherstellen, Suchen und
Ersetzen, Auswahl, Zeilen verschieben und Syntaxhervorhebung für Go (über
`go/scanner`), JavaScript und TypeScript, Python, Ruby, Shell, C und C++,
Rust, JSON, YAML, HCL, TOML, LaTeX und Markdown, ohne externe Abhängigkeit.

```sh
go run ./cmd/tui-widgets | less -R     # every cell-native widget, one per slot
go run ./cmd/tui-widget-explorer       # the same, interactive
go run ./cmd/tui-explorer              # a file browser
go run ./cmd/tui-editor --file=x.go    # a modal editor
```

Die beiden Demoprogramme werden Ende-zu-Ende in einem echten pty getestet: echte Tasten-Bytes hinein,
der gerenderte Frame geprüft.
