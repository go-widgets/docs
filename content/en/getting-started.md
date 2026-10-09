---
title: "A first application"
weight: 5
description: "Start from app-template, already MVVM-compliant and green, or open a bare window and add widgets."
tags: [getting-started, app-template, window]
---

## From the template

[`go-widgets/app-template`](https://github.com/go-widgets/app-template) is a
template repository: a small but real browser application with a search
field, a category filter, a list of the rows that match and a status line.
Everything a larger application needs is already wired: a view-model with no
widget in it, a view bound to it, a wasm host, a 100% coverage gate, and the
[MVVM gate]({{< relref "/guard-rails/mvvmlint.md" >}}).

```sh
git clone https://github.com/go-widgets/app-template my-app
cd my-app
go mod edit -module github.com/you/my-app
./build.sh          # dist/app.wasm, dist/wasm_exec.js, dist/index.html
```

Serve `dist/` over HTTP: a page opened from `file://` cannot instantiate wasm.

| File | What it holds |
|---|---|
| `viewmodel.go` | every piece of state, as [`mvvm`]({{< relref "/state/mvvm.md" >}}) observables and commands. No widget. Testable with no canvas. |
| `scene.go` | the widgets, each bound to the view-model through `mvvmtk`. It never assigns a widget's state field itself. |
| `main.go` | the only file with a build tag (`js && wasm`): forwards input to the toolkit and blits the frame into a `<canvas>`. |

Edit the first two and keep state flowing through the binders: the
`mvvm` job in CI fails a pull request that assigns a widget's field directly.

## From a bare window

```go
package main

import (
	"github.com/go-widgets/toolkit"
	"github.com/go-widgets/window"
)

func main() {
	w, err := window.Open(window.Config{Title: "Demo"})
	if err == window.ErrUnsupported {
		return // no native back-end on this platform
	}
	if err != nil {
		panic(err)
	}
	defer w.Close()

	box := toolkit.NewVBox()
	box.Append(toolkit.NewLabel("Hello"))
	box.Append(toolkit.NewButton("Click me", func() {}))
	w.Run(box) // layout, draw, present, dispatch input, until closed
}
```

Width and height left at zero ask the back-end for a readable default; on
macOS that is a fraction of the main screen. `CGO_ENABLED=0 go build` produces
the binary for each platform, from any machine.

## Which module for what

| You want | Start with |
|---|---|
| a desktop window | [`window`]({{< relref "/surfaces/native-window.md" >}}) |
| a desktop application with a tray icon and the system appearance | [`application`]({{< relref "/surfaces/application.md" >}}) |
| a page in a browser | [`webcanvas`]({{< relref "/surfaces/browser.md" >}}), or the template above |
| a terminal program | [`tui`]({{< relref "/surfaces/terminal.md" >}}) |
| an Android app | [`android`]({{< relref "/surfaces/android.md" >}}) |
| a picture of a widget for a README | [`svg`]({{< relref "/surfaces/snapshots.md" >}}) |

To see the widget set before writing anything, open the
[gallery](https://go-widgets.github.io/gallery/): every widget family, live,
in a browser canvas.
