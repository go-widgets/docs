---
title: "Module"
weight: 50
description: "Die vierundzwanzig Module von go-widgets, wofür jedes da ist und wo seine Quellen und seine API-Referenz liegen."
tags: [module]
---

Die API-Referenz erzeugt
[pkg.go.dev](https://pkg.go.dev/) aus den Quellen und folgt dabei den veröffentlichten Tags; diese
Seiten erklären sie, statt sie zu kopieren. Der Tag ist der neueste zum Zeitpunkt des
Schreibens – maßgeblich ist die Tag-Liste des jeweiligen Repositorys.

| Modul | Tag | Was es ist | Referenz |
|---|---|---|---|
| [`painter`](https://github.com/go-widgets/painter) | v0.17.0 | [die Zeichenschnittstelle]({{< relref "/rendering/painter.md" >}}): Pixel- und Zellen-Painter | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/painter) |
| [`toolkit`](https://github.com/go-widgets/toolkit) | v0.364.0 | [der Widget-Satz]({{< relref "/rendering/toolkit.md" >}}), Themes, Layouts, Text | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/toolkit) |
| [`skin`](https://github.com/go-widgets/skin) | v0.3.0 | [Widgets als Daten beschrieben]({{< relref "/rendering/skin.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/skin) |
| [`gpupainter`](https://github.com/go-widgets/gpupainter) | v0.4.0 | ein `painter.Painter`, der ein Bild für die GPU aufzeichnet (go-gfx/gpu2d): [Metal oder WebGPU]({{< relref "/surfaces/browser.md" >}}), bytegleich mit dem CPU-Painter | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/gpupainter) |
| [`shellterm`](https://github.com/go-widgets/shellterm) | v0.1.0 | ein zeilenorientiertes Terminal: das `Terminal` des Toolkits und ein Zeileneditor vor einer Shell, die Sie mitbringen | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/shellterm) |
| [`isoicons`](https://github.com/go-widgets/isoicons) | v0.4.0 | isometrische Icon-Pakete (Cloud-native, AWS) für die isometrischen Diagramme des Toolkits | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/isoicons) |
| [`mvvm`](https://github.com/go-widgets/mvvm) | v0.15.0 | [Observables, Commands, Binder, Rückgängig]({{< relref "/state/mvvm.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvm) |
| [`mvvmtk`](https://github.com/go-widgets/mvvmtk) | v0.15.0 | [Binder mit einem Aufruf für Toolkit-Widgets]({{< relref "/state/mvvm.md#mvvmtk-one-call-per-widget" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmtk) |
| [`lineedit`](https://github.com/go-widgets/lineedit) | v0.1.1 | ein Zeileneditor als reines Modell: Verlauf, Rückwärtssuche, Tab-Vervollständigung; nur die Standardbibliothek | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/lineedit) |
| [`vt`](https://github.com/go-widgets/vt) | v0.1.0 | ein Terminalemulator als reines Modell: ECMA-48 und die xterm-Teilmenge, Bytes hinein, Zellen heraus; antwortet auf nichts; nur die Standardbibliothek | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/vt) |
| [`vcs`](https://github.com/go-widgets/vcs) | v0.1.0 | welchen Zustand jede Datei eines Arbeitsverzeichnisses hat, und eine Registry für Treiber; nur die Standardbibliothek. Der Git-Treiber ist das verschachtelte Modul `vcs/git` (`git/v0.1.0`) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/vcs) |
| [`data`](https://github.com/go-widgets/data) | v0.4.0 | [typisierte Datensätze, Abfragen, ein lokaler oder entfernter Speicher]({{< relref "/state/data.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/data) |
| [`window`](https://github.com/go-widgets/window) | v0.91.0 | [ein natives Fenster]({{< relref "/surfaces/native-window.md" >}}) auf acht Back-Ends | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/window) |
| [`application`](https://github.com/go-widgets/application) | v0.8.1 | [der Anwendungslebenszyklus]({{< relref "/surfaces/application.md" >}}) oberhalb eines Fensters | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/application) |
| [`tray`](https://github.com/go-widgets/tray) | v0.15.0 | [ein Tray-Symbol und sein Menü]({{< relref "/surfaces/application.md#the-tray" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tray) |
| [`webcanvas`](https://github.com/go-widgets/webcanvas) | v0.9.0 | [eine Szene in einem Browser-`<canvas>`]({{< relref "/surfaces/browser.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/webcanvas) |
| [`tui`](https://github.com/go-widgets/tui) | v0.62.0 | [Terminal-Rendering und zellennative Widgets]({{< relref "/surfaces/terminal.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tui) |
| [`android`](https://github.com/go-widgets/android) | v0.16.0 | [eine APK, gezeichnet von einem CGO-freien Prozess]({{< relref "/surfaces/android.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/android) |
| [`svg`](https://github.com/go-widgets/svg) | v0.7.0 | [ein Rendering als SVG oder PNG]({{< relref "/surfaces/snapshots.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/svg) |
| [`mvvmlint`](https://github.com/go-widgets/mvvmlint) | v0.4.2 | [die MVVM-Prüfung]({{< relref "/guard-rails/mvvmlint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmlint) |
| [`bricolint`](https://github.com/go-widgets/bricolint) | v0.4.1 | [die Prüfung gegen handgezeichnete UI]({{< relref "/guard-rails/bricolint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/bricolint) |
| [`app-template`](https://github.com/go-widgets/app-template) | v0.5.0 | [der Ausgangspunkt für eine Anwendung]({{< relref "/getting-started.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/app-template) |
| [`gallery`](https://github.com/go-widgets/gallery) | v0.7.0 | die [Live-Demo](https://go-widgets.github.io/gallery/) jeder Widget-Familie | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/gallery) |
| [`desktop`](https://github.com/go-widgets/desktop) | v0.20.0 | eine Desktop-Shell, die go-freedesktop und go-widgets kombiniert, nativ und in wasmdesk | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/desktop) |

## Woran sich jedes Modul halten muss {#what-every-module-is-held-to}

- `CGO_ENABLED=0`: kein cgo und kein Aufruf eines Kommandozeilenwerkzeugs anstelle
  einer Bibliothek.
- Cross-kompiliert für amd64, arm64, riscv64, loong64, ppc64le und s390x – letzteres
  Big-Endian, was jede Wire-Kodierung ehrlich hält – und, wo es zutrifft,
  `js/wasm`. Alle zweiundzwanzig Bibliotheken und Werkzeuge tun das, jedes seit seiner Aufnahme oder seit dem 2026-10-10, je nachdem, was später liegt;
  die übrigen zwei sind Browser-Anwendungen ([Stand]({{< relref "/status.md#what-ci-does-not-cover-yet" >}})).
- Eine Schranke für die Anweisungsabdeckung in der CI: 100 %, bei allen vierundzwanzig. Wo eine
  Datei nicht in einem Test laufen kann (eine Browser-Schleife, eine native Run-Loop), sagt das Modul,
  welche Datei ausgenommen ist und warum. Die Ausnahmen stehen unter [Stand]({{< relref "/status.md" >}}).
- BSD-3-Clause.
