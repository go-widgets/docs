---
title: "State lives in a view-model"
linkTitle: "State"
weight: 20
description: "Why application state is held in observables and bound to widgets, rather than written into widget fields."
tags: [state, mvvm]
---

A widget's fields — an entry's text, a list's items, a table's rows — are its
**view** of the state, not the state. An application that assigns them from
wherever it happens to be scatters its state across call sites, and two of
those call sites eventually disagree about what is on screen.

So state lives in a **view-model**: plain Go values wrapped in observables,
with no widget in sight, testable without a window. The view binds each widget
to the view-model once, and from then on the binding — not the application —
keeps them in step, in both directions.

| | |
|---|---|
| [MVVM]({{< relref "/state/mvvm.md" >}}) | `Observable`, `Command`, `ObservableList`; binders for the toolkit and the terminal widgets; undo and redo |
| [The data spine]({{< relref "/state/data.md" >}}) | typed records with validation, queries, and a store that runs the same pipeline locally or over gRPC |
| [`mvvmlint`]({{< relref "/guard-rails/mvvmlint.md" >}}) | the CI check that fails a pull request assigning a widget's state field by hand |
