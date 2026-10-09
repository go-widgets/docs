---
title: "MVVM: observables, commands, binders"
linkTitle: "MVVM"
weight: 10
description: "go-widgets/mvvm, its toolkit and terminal binders, mvvmtk's one-call helpers, and an undo stack."
tags: [state, mvvm, mvvmtk, undo]
---

[`go-widgets/mvvm`](https://github.com/go-widgets/mvvm) is generics only, with
no reflection. Its core package imports no widget package at all, so one
view-model drives the pixel [toolkit]({{< relref "/rendering/toolkit.md" >}})
and the [terminal widgets]({{< relref "/surfaces/terminal.md" >}}) alike.

| Primitive | Role |
|---|---|
| `Observable[T]` | a property: `Get`, `Set`, `Subscribe`. Setting an equal value does nothing. |
| `Command` | an action, with `CanExecute` and `RaiseCanExecuteChanged` |
| `ObservableList[T]` | a collection that reports each insert, remove, replace, move and reset |

## A form

```go
type FormVM struct {
	Name  *mvvm.Observable[string]
	Names *mvvm.ObservableList[string]
	Save  *mvvm.Command
}

vm := &FormVM{Name: mvvm.NewObservable(""), Names: mvvm.NewObservableList[string]()}
vm.Save = mvvm.NewCommand(
	func() { vm.Names.Append(vm.Name.Get()); vm.Name.Set("") },
	func() bool { return vm.Name.Get() != "" }, // CanExecute
)
mvvm.BindCanExecute(vm.Save, vm.Name) // Save greys out while the name is empty
```

## Binding

The generic binders reach a widget through a **pointer to its value field and
a pointer to its callback slot**: `BindField` (two-way), `OneWay`,
`BindCommand`, `BindList`. A widget fires its callback from its event handler
and leaves a direct field write silent, and `Observable.Set` ignores an equal
value — so a two-way binding can echo without recursing.

The toolkit is moving its widgets to exposing their state **as** an
observable (`Label.Text()` returns an `*mvvm.Observable[string]`). There is
then no field to seed and no callback to compose, only two properties that
must agree: `BindTwoWay(src, dst, invalidate)`.

Widgets whose callback takes several arguments get a small named adapter in a
per-back-end package, the only packages that import a back-end:

| Package | Adapters |
|---|---|
| `mvvm/tkbind` | `BindRange` (the two-handle `RangeSlider`), `BindContainer` and `BindCardActive` (a container's children and a card layout from a list or an index), `BindStore` and `BindTable` (a `Table` over a [data store]({{< relref "/state/data.md" >}}): sorting, grouping and validated inline edits) |
| `mvvm/tuibind` | `BindDropdown`, `BindTableSelection` |

## mvvmtk: one call per widget

[`go-widgets/mvvmtk`](https://github.com/go-widgets/mvvmtk) fills in each
toolkit widget's real field and callback names, so the view never names them:

```go
unbind := mvvmtk.BindText(search, vm.Query, win.Invalidate) // search.Text ⇄ vm.Query
defer unbind()
```

| Helper | Direction |
|---|---|
| `BindText` (`SearchEntry`), `BindEntryText`, `BindChecked`, `BindSelectedIndex`, `BindSpin`, `BindListSelection`, `BindViewSwitcher` | two-way |
| `BindLabel`, `BindProgress` | view-model → widget |
| `BindListItems`, `BindDropDownOptions`, `BindViews`, `BindTree` | list → widget |
| `BindCommand` | a button's click, its `Disabled` state and its greying follow the command |

Every helper returns an `unbind` that detaches the binding and restores the
callback that was there before. A button bound to a command that cannot run
takes no click, no Enter or Space, and no keyboard focus.

## Undo and redo

`mvvm/undo` is a command stack built on the core primitives alone, so it
works under any view:

```go
s := undo.New() // unlimited, coalescing on; undo.WithLimit(n), undo.WithCoalescing(false)
s.Push(undo.NewCommand("Write hello", do, undoIt))
s.Undo()
s.Redo()

mvvm.BindCommand(s.UndoCommand(), &undoBtn.OnClick, setEnabled)
mvvm.OneWay(s.UndoTextBinding(), &undoBtn.Text, repaint) // "Undo Write hello"
```

A run of keystrokes coalesces into one step, and a push after an undo drops
the redo tail.
