---
title: "MVVM: Observables, Commands, Binder"
linkTitle: "MVVM"
weight: 10
description: "go-widgets/mvvm, seine Binder für Toolkit und Terminal, die Ein-Aufruf-Helfer von mvvmtk und ein Undo-Stack."
tags: [zustand, mvvm, mvvmtk, rückgängig]
---

[`go-widgets/mvvm`](https://github.com/go-widgets/mvvm) besteht nur aus Generics,
ohne Reflection. Sein Kernpaket importiert überhaupt kein Widget-Paket, sodass ein
View-Model das Pixel-[Toolkit]({{< relref "/rendering/toolkit.md" >}})
und die [Terminal-Widgets]({{< relref "/surfaces/terminal.md" >}}) gleichermaßen steuert.

| Primitiv | Rolle |
|---|---|
| `Observable[T]` | eine Eigenschaft: `Get`, `Set`, `Subscribe`. Das Setzen eines gleichen Werts bewirkt nichts. |
| `Command` | eine Aktion, mit `CanExecute` und `RaiseCanExecuteChanged` |
| `ObservableList[T]` | eine Sammlung, die jedes Einfügen, Entfernen, Ersetzen, Verschieben und Zurücksetzen meldet |

## Ein Formular {#a-form}

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

## Bindung {#binding}

Die generischen Binder erreichen ein Widget über einen **Zeiger auf sein Wertfeld und
einen Zeiger auf seinen Callback-Platz**: `BindField` (bidirektional), `OneWay`,
`BindCommand`, `BindList`. Ein Widget löst seinen Callback aus seinem Event-Handler aus
und lässt einen direkten Feldschreibzugriff stumm, und `Observable.Set` ignoriert einen gleichen
Wert – so kann eine bidirektionale Bindung ein Echo erzeugen, ohne in eine Rekursion zu geraten.

Das Toolkit stellt seine Widgets nach und nach darauf um, ihren Zustand **als**
Observable bereitzustellen (`Label.Text()` gibt ein `*mvvm.Observable[string]` zurück). Dann
gibt es kein Feld mehr vorzubelegen und keinen Callback zusammenzusetzen, nur zwei Eigenschaften, die
übereinstimmen müssen: `BindTwoWay(src, dst, invalidate)`.

Widgets, deren Callback mehrere Argumente nimmt, bekommen einen kleinen benannten Adapter in einem
Paket pro Back-End, den einzigen Paketen, die ein Back-End importieren:

| Paket | Adapter |
|---|---|
| `mvvm/tkbind` | `BindRange` (der `RangeSlider` mit zwei Griffen), `BindContainer` und `BindCardActive` (die Kinder eines Containers und ein Card-Layout aus einer Liste oder einem Index), `BindStore` und `BindTable` (eine `Table` über einem [Daten-Store]({{< relref "/state/data.md" >}}): Sortieren, Gruppieren und validierte Inline-Bearbeitung) |
| `mvvm/tuibind` | `BindDropdown`, `BindTableSelection` |

## mvvmtk: ein Aufruf pro Widget {#mvvmtk-one-call-per-widget}

[`go-widgets/mvvmtk`](https://github.com/go-widgets/mvvmtk) setzt die echten Feld- und
Callback-Namen jedes Toolkit-Widgets ein, sodass die View sie nie nennen muss:

```go
unbind := mvvmtk.BindText(search, vm.Query, win.Invalidate) // search.Text ⇄ vm.Query
defer unbind()
```

| Helfer | Richtung |
|---|---|
| `BindText` (`SearchEntry`), `BindEntryText`, `BindChecked`, `BindSelectedIndex`, `BindSpin`, `BindListSelection`, `BindViewSwitcher` | bidirektional |
| `BindLabel`, `BindProgress` | View-Model → Widget |
| `BindListItems`, `BindDropDownOptions`, `BindViews`, `BindTree` | Liste → Widget |
| `BindCommand` | der Klick eines Buttons, sein `Disabled`-Zustand und sein Ausgrauen folgen dem Command |

Jeder Helfer gibt ein `unbind` zurück, das die Bindung löst und den
zuvor vorhandenen Callback wiederherstellt. Ein Button, der an ein Command gebunden ist, das nicht ausgeführt werden kann,
nimmt keinen Klick, kein Enter oder Leerzeichen und keinen Tastaturfokus an.

## Rückgängig und Wiederherstellen {#undo-and-redo}

`mvvm/undo` ist ein Command-Stack, der allein auf den Kern-Primitiven aufbaut, und
funktioniert daher unter jeder View:

```go
s := undo.New() // unlimited, coalescing on; undo.WithLimit(n), undo.WithCoalescing(false)
s.Push(undo.NewCommand("Write hello", do, undoIt))
s.Undo()
s.Redo()

mvvm.BindCommand(s.UndoCommand(), &undoBtn.OnClick, setEnabled)
mvvm.OneWay(s.UndoTextBinding(), &undoBtn.Text, repaint) // "Undo Write hello"
```

Eine Folge von Tastenanschlägen wird zu einem Schritt zusammengefasst, und ein Push nach einem Undo verwirft
den Redo-Rest.
