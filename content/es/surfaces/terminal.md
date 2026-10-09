---
title: "Un terminal"
linkTitle: "Terminal"
weight: 40
description: "go-widgets/tui renderiza widgets a través del painter de celdas: un renderizador de instantáneas, un ejecutor interactivo y widgets dimensionados para una cuadrícula de caracteres."
tags: [superficies, tui, terminal]
---

[`go-widgets/tui`](https://github.com/go-widgets/tui) convierte un árbol de
widgets en un fotograma de terminal a través del
[painter de celdas]({{< relref "/rendering/painter.md" >}}).
Solo la biblioteca estándar: sin `x/term`, sin cgo.

## Un fotograma {#one-frame}

```go
_ = tui.RenderOnce(os.Stdout, widgets, nil) // nil theme: light
```

`RenderOnceSized(w, cols, rows, widgets, theme)` recibe el tamaño de forma
explícita; `RenderOnce` lee `COLUMNS` y `LINES` y, en su defecto, usa 80×24.
Leer el entorno en lugar de llamar a `TIOCGWINSZ` mantiene el paquete libre de
código específico de cada plataforma.

## Un programa interactivo {#an-interactive-program}

```go
app := tui.NewApp()
app.Root = buildScene()
app.Keys["q"] = func(a *tui.App) { a.Quit() }
app.Keys["Ctrl+C"] = func(a *tui.App) { a.Quit() }
os.Exit(app.Run())
```

`Run` entra en la pantalla alternativa y en modo raw, despacha la entrada a
`Root` (primero los manejadores de teclas globales), sigue `SIGWINCH` y siempre
restaura el terminal, también ante un panic. Un manejador de teclas llama a
`Consume` para que un evento no llegue a `Root`; `InputTarget` envía todos los
eventos no consumidos a un único widget, que es como una paleta de comandos se
traga lo que se teclea mientras está abierta. `NewFocusRing` da a un formulario
el recorrido con Tab y Mayús+Tab.

## Widgets dimensionados para celdas {#widgets-sized-for-cells}

La mayoría de los widgets del toolkit están diseñados en píxeles: sus
constantes de relleno cuentan celdas bajo el painter de celdas y se inflan. Por
eso `tui` tiene su propio conjunto **nativo de celdas** —`TextEditor`, `Entry`,
`Button`, `CheckButton`, `RadioButton`, `Scale`, `ProgressBar`, `Sparkline`,
`ListBox`, `TreeView`, `Table`, `MenuBar`, `Notebook`, `HSplit`, `VSplit`,
`Dialog`, `Dropdown`, `Spinner` y más—, donde un glifo es una celda. Cada uno
sigue siendo un `toolkit.Widget`, así que la misma instancia también dibuja en
píxeles.

`TextEditor` es un editor de código de lectura y escritura: margen de números,
deshacer y rehacer, buscar y reemplazar, selección, desplazamiento de líneas y
resaltado de sintaxis para Go (mediante `go/scanner`), JavaScript y TypeScript,
Python, Ruby, shell, C y C++, Rust, JSON, YAML, HCL, TOML, LaTeX y Markdown, sin
ninguna dependencia externa.

```sh
go run ./cmd/tui-widgets | less -R     # every cell-native widget, one per slot
go run ./cmd/tui-widget-explorer       # the same, interactive
go run ./cmd/tui-explorer              # a file browser
go run ./cmd/tui-editor --file=x.go    # a modal editor
```

Los dos programas de demostración se prueban de extremo a extremo en un pty
real: bytes de teclas reales de entrada, y el fotograma renderizado comprobado.
