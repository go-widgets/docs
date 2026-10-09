---
title: "A terminal"
linkTitle: "Terminal"
weight: 40
description: "go-widgets/tui renders widgets through the cell painter: a snapshot renderer, an interactive runner, and widgets sized for a character grid."
tags: [surfaces, tui, terminal]
---

[`go-widgets/tui`](https://github.com/go-widgets/tui) turns a widget tree into
a terminal frame through the [cell painter]({{< relref "/rendering/painter.md" >}}).
Standard library only — no `x/term`, no cgo.

## One frame

```go
_ = tui.RenderOnce(os.Stdout, widgets, nil) // nil theme: light
```

`RenderOnceSized(w, cols, rows, widgets, theme)` takes the size explicitly;
`RenderOnce` reads `COLUMNS` and `LINES` and falls back to 80×24. Reading the
environment rather than calling `TIOCGWINSZ` keeps the package free of
per-platform code.

## An interactive program

```go
app := tui.NewApp()
app.Root = buildScene()
app.Keys["q"] = func(a *tui.App) { a.Quit() }
app.Keys["Ctrl+C"] = func(a *tui.App) { a.Quit() }
os.Exit(app.Run())
```

`Run` enters the alternate screen and raw mode, dispatches input to `Root`
(global key handlers first), follows `SIGWINCH`, and always restores the
terminal — on a panic too. A key handler calls `Consume` to keep an event from
`Root`; `InputTarget` sends every unconsumed event to one widget, which is how
a command palette swallows typing while it is open. `NewFocusRing` gives a
form Tab and Shift+Tab traversal.

## Widgets sized for cells

Most toolkit widgets are designed in pixels: their padding constants count
cells under the cell painter and inflate. So `tui` has its own **cell-native**
set — `TextEditor`, `Entry`, `Button`, `CheckButton`, `RadioButton`, `Scale`,
`ProgressBar`, `Sparkline`, `ListBox`, `TreeView`, `Table`, `MenuBar`,
`Notebook`, `HSplit`, `VSplit`, `Dialog`, `Dropdown`, `Spinner` and more —
where one glyph is one cell. Each is still a `toolkit.Widget`, so the same
instance also draws into pixels.

`TextEditor` is a read-write code editor: gutter, undo and redo, search and
replace, selection, line moves, and syntax highlighting for Go (through
`go/scanner`), JavaScript and TypeScript, Python, Ruby, shell, C and C++,
Rust, JSON, YAML, HCL, TOML, LaTeX and Markdown, with no external dependency.

```sh
go run ./cmd/tui-widgets | less -R     # every cell-native widget, one per slot
go run ./cmd/tui-widget-explorer       # the same, interactive
go run ./cmd/tui-explorer              # a file browser
go run ./cmd/tui-editor --file=x.go    # a modal editor
```

The two demo programs are tested end to end in a real pty: real key bytes in,
the rendered frame asserted.
