---
title: "Eine erste Anwendung"
weight: 5
description: "Mit app-template beginnen, das bereits MVVM-konform und grün ist, oder ein nacktes Fenster öffnen und Widgets hinzufügen."
tags: [erste-schritte, app-template, fenster]
---

## Aus dem Template {#from-the-template}

[`go-widgets/app-template`](https://github.com/go-widgets/app-template) ist ein
Template-Repository: eine kleine, aber echte Browser-Anwendung mit einem Suchfeld,
einem Kategoriefilter, einer Liste der passenden Zeilen und einer Statuszeile.
Alles, was eine größere Anwendung braucht, ist bereits verdrahtet: ein View-Model ohne
Widget darin, eine daran gebundene View, ein wasm-Host, eine 100-%-Coverage-Schranke und die
[MVVM-Schranke]({{< relref "/guard-rails/mvvmlint.md" >}}).

```sh
git clone https://github.com/go-widgets/app-template my-app
cd my-app
go mod edit -module github.com/you/my-app
./build.sh          # dist/app.wasm, dist/wasm_exec.js, dist/index.html
```

Stellen Sie `dist/` über HTTP bereit: Eine über `file://` geöffnete Seite kann kein wasm instanziieren.

| Datei | Was sie enthält |
|---|---|
| `viewmodel.go` | jedes Stück Zustand, als [`mvvm`]({{< relref "/state/mvvm.md" >}})-Observables und -Commands. Kein Widget. Ohne Canvas testbar. |
| `scene.go` | die Widgets, jedes über `mvvmtk` an das View-Model gebunden. Es weist nie selbst ein Zustandsfeld eines Widgets zu. |
| `main.go` | die einzige Datei mit einem Build-Tag (`js && wasm`): leitet Eingaben an das Toolkit weiter und überträgt den Frame per Blit in ein `<canvas>`. |

Bearbeiten Sie die ersten beiden und lassen Sie den Zustand weiter durch die Binder fließen: Der
`mvvm`-Job in der CI lässt einen Pull Request scheitern, der ein Feld eines Widgets direkt zuweist.

## Aus einem nackten Fenster {#from-a-bare-window}

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

Breite und Höhe auf null belassen bittet das Back-End um einen lesbaren Standardwert; unter
macOS ist das ein Bruchteil des Hauptbildschirms. `CGO_ENABLED=0 go build` erzeugt
das Binary für jede Plattform, von jeder Maschine aus.

## Welches Modul wofür {#which-module-for-what}

| Sie wollen | Beginnen Sie mit |
|---|---|
| ein Desktop-Fenster | [`window`]({{< relref "/surfaces/native-window.md" >}}) |
| eine Desktop-Anwendung mit Tray-Symbol und dem Erscheinungsbild des Systems | [`application`]({{< relref "/surfaces/application.md" >}}) |
| eine Seite im Browser | [`webcanvas`]({{< relref "/surfaces/browser.md" >}}) oder das Template oben |
| ein Terminalprogramm | [`tui`]({{< relref "/surfaces/terminal.md" >}}) |
| eine Android-App | [`android`]({{< relref "/surfaces/android.md" >}}) |
| ein Bild eines Widgets für ein README | [`svg`]({{< relref "/surfaces/snapshots.md" >}}) |

Um den Widget-Satz zu sehen, bevor Sie etwas schreiben, öffnen Sie die
[Galerie](https://go-widgets.github.io/gallery/): jede Widget-Familie, live,
in einem Browser-Canvas.
