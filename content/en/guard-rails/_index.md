---
title: "Rules CI enforces"
linkTitle: "CI rules"
weight: 40
description: "Two go/analysis checks an application runs as required status checks: state through MVVM, and no hand-drawn UI."
tags: [ci, lint]
---

Two conventions keep an application's UI honest, and both would erode one
convenient shortcut at a time if they were only written down. So each is a
[`go/analysis`](https://pkg.go.dev/golang.org/x/tools/go/analysis) analyzer,
run through `go vet`, with a reusable workflow an application marks as a
**required** check.

| | Refuses |
|---|---|
| [`mvvmlint`]({{< relref "/guard-rails/mvvmlint.md" >}}) | assigning a widget's state field by hand, and a package with widgets but no view-model |
| [`bricolint`]({{< relref "/guard-rails/bricolint.md" >}}) | painter primitives in application code, and a widget built anew on every frame |

Both are conservative by construction: a rule fires only when the **static
type** of the receiver resolves, through `go/types`, to a toolkit or painter
type. A same-named field or method on an unrelated type is never flagged. The
intended exceptions are explicit directives in the code, so each one is
written down where it is made.
