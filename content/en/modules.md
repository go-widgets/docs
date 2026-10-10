---
title: "Modules"
weight: 50
description: "The nineteen modules of go-widgets, what each one is for, and where its source and API reference live."
tags: [modules]
---

The API reference is generated from the source by
[pkg.go.dev](https://pkg.go.dev/), which follows the released tags; these
pages explain, and do not copy it. The tag is the newest one at the time of
writing — the repository's own tag list is the authority.

| Module | Tag | What it is | Reference |
|---|---|---|---|
| [`painter`](https://github.com/go-widgets/painter) | v0.15.0 | [the drawing seam]({{< relref "/rendering/painter.md" >}}): pixel and cell painters | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/painter) |
| [`toolkit`](https://github.com/go-widgets/toolkit) | v0.328.0 | [the widget set]({{< relref "/rendering/toolkit.md" >}}), themes, layouts, text | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/toolkit) |
| [`skin`](https://github.com/go-widgets/skin) | v0.2.0 | [widgets described as data]({{< relref "/rendering/skin.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/skin) |
| [`isoicons`](https://github.com/go-widgets/isoicons) | v0.3.0 | isometric icon packs (cloud-native, AWS) for the toolkit's isometric diagrams | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/isoicons) |
| [`mvvm`](https://github.com/go-widgets/mvvm) | v0.13.0 | [observables, commands, binders, undo]({{< relref "/state/mvvm.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvm) |
| [`mvvmtk`](https://github.com/go-widgets/mvvmtk) | v0.14.1 | [one-call binders for toolkit widgets]({{< relref "/state/mvvm.md#mvvmtk-one-call-per-widget" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmtk) |
| [`data`](https://github.com/go-widgets/data) | v0.3.0 | [typed records, queries, a local or remote store]({{< relref "/state/data.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/data) |
| [`window`](https://github.com/go-widgets/window) | v0.86.1 | [a native window]({{< relref "/surfaces/native-window.md" >}}) on eight back-ends | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/window) |
| [`application`](https://github.com/go-widgets/application) | v0.7.0 | [the application lifecycle]({{< relref "/surfaces/application.md" >}}) above a window | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/application) |
| [`tray`](https://github.com/go-widgets/tray) | v0.14.0 | [a tray icon and its menu]({{< relref "/surfaces/application.md#the-tray" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tray) |
| [`webcanvas`](https://github.com/go-widgets/webcanvas) | v0.2.0 | [a scene in a browser `<canvas>`]({{< relref "/surfaces/browser.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/webcanvas) |
| [`tui`](https://github.com/go-widgets/tui) | v0.61.0 | [terminal rendering and cell-native widgets]({{< relref "/surfaces/terminal.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tui) |
| [`android`](https://github.com/go-widgets/android) | v0.15.0 | [an APK painted by a CGO-free process]({{< relref "/surfaces/android.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/android) |
| [`svg`](https://github.com/go-widgets/svg) | v0.6.0 | [a render as SVG or PNG]({{< relref "/surfaces/snapshots.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/svg) |
| [`mvvmlint`](https://github.com/go-widgets/mvvmlint) | v0.4.0 | [the MVVM check]({{< relref "/guard-rails/mvvmlint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmlint) |
| [`bricolint`](https://github.com/go-widgets/bricolint) | v0.4.0 | [the no-hand-drawn-UI check]({{< relref "/guard-rails/bricolint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/bricolint) |
| [`app-template`](https://github.com/go-widgets/app-template) | v0.4.0 | [the starting point for an application]({{< relref "/getting-started.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/app-template) |
| [`gallery`](https://github.com/go-widgets/gallery) | v0.6.0 | the [live demo](https://go-widgets.github.io/gallery/) of every widget family | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/gallery) |
| [`desktop`](https://github.com/go-widgets/desktop) | v0.19.0 | a desktop shell composing go-freedesktop and go-widgets, native and in wasmdesk | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/desktop) |

## What every module is held to

- `CGO_ENABLED=0`: no cgo, and no shelling out to a command-line tool in place
  of a library.
- Cross-compiled for amd64, arm64, riscv64, loong64, ppc64le and s390x — the
  last big-endian, which keeps every wire encoding honest — and, where it
  applies, `js/wasm`. All seventeen libraries and tools do since 2026-10-10;
  the other two are browser applications ([Status]({{< relref "/status.md#what-ci-does-not-cover-yet" >}})).
- A statement-coverage gate in CI: 100%, in all nineteen. Where a
  file cannot run in a test (a browser loop, a native run loop), the module
  says which file is left out and why. See [Status]({{< relref "/status.md" >}})
  for the exceptions.
- BSD-3-Clause.
