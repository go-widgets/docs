---
title: "MVVM: observables, comandos, binders"
linkTitle: "MVVM"
weight: 10
description: "go-widgets/mvvm, sus binders para el toolkit y el terminal, las funciones de una sola llamada de mvvmtk y una pila de deshacer."
tags: [gestión del estado, mvvm, mvvmtk, deshacer]
---

[`go-widgets/mvvm`](https://github.com/go-widgets/mvvm) usa solo genéricos, sin
reflexión. Su paquete central no importa ningún paquete de widgets, de modo que
un mismo view-model controla tanto el [toolkit]({{< relref "/rendering/toolkit.md" >}})
de píxeles como los [widgets de terminal]({{< relref "/surfaces/terminal.md" >}}).

| Primitiva | Papel |
|---|---|
| `Observable[T]` | una propiedad: `Get`, `Set`, `Subscribe`. Asignar un valor igual no hace nada. |
| `Command` | una acción, con `CanExecute` y `RaiseCanExecuteChanged` |
| `ObservableList[T]` | una colección que notifica cada inserción, eliminación, sustitución, desplazamiento y reinicio |

## Un formulario {#a-form}

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

## Enlace {#binding}

Los binders genéricos llegan a un widget mediante **un puntero a su campo de
valor y un puntero a su ranura de callback**: `BindField` (bidireccional),
`OneWay`, `BindCommand`, `BindList`. Un widget dispara su callback desde su
manejador de eventos y deja en silencio una escritura directa del campo, y
`Observable.Set` ignora un valor igual: así, un enlace bidireccional puede
hacer eco sin entrar en recursión.

El toolkit está pasando sus widgets a exponer su estado **como** un observable
(`Label.Text()` devuelve un `*mvvm.Observable[string]`). Entonces no hay ningún
campo que inicializar ni ningún callback que componer, solo dos propiedades que
deben coincidir: `BindTwoWay(src, dst, invalidate)`.

Los widgets cuyo callback recibe varios argumentos tienen un pequeño adaptador
con nombre en un paquete por backend, los únicos paquetes que importan un
backend:

| Paquete | Adaptadores |
|---|---|
| `mvvm/tkbind` | `BindRange` (el `RangeSlider` de dos tiradores), `BindContainer` y `BindCardActive` (los hijos de un contenedor y un card layout a partir de una lista o de un índice), `BindStore` y `BindTable` (un `Table` sobre un [almacén de datos]({{< relref "/state/data.md" >}}): ordenación, agrupación y ediciones en línea validadas) |
| `mvvm/tuibind` | `BindDropdown`, `BindTableSelection` |

## mvvmtk: una llamada por widget {#mvvmtk-one-call-per-widget}

[`go-widgets/mvvmtk`](https://github.com/go-widgets/mvvmtk) rellena los nombres
reales del campo y del callback de cada widget del toolkit, de modo que la vista
nunca los nombra:

```go
unbind := mvvmtk.BindText(search, vm.Query, win.Invalidate) // search.Text ⇄ vm.Query
defer unbind()
```

| Función | Sentido |
|---|---|
| `BindText` (`SearchEntry`), `BindEntryText`, `BindChecked`, `BindSelectedIndex`, `BindSpin`, `BindListSelection`, `BindViewSwitcher` | bidireccional |
| `BindLabel`, `BindProgress` | view-model → widget |
| `BindListItems`, `BindDropDownOptions`, `BindViews`, `BindTree` | lista → widget |
| `BindCommand` | el clic de un botón, su estado `Disabled` y su atenuado siguen al comando |

Cada función devuelve un `unbind` que desconecta el enlace y restaura el
callback que había antes. Un botón enlazado a un comando que no puede ejecutarse
no acepta ningún clic, ni Intro ni Espacio, ni el foco del teclado.

## Deshacer y rehacer {#undo-and-redo}

`mvvm/undo` es una pila de comandos construida únicamente sobre las primitivas
centrales, así que funciona bajo cualquier vista:

```go
s := undo.New() // unlimited, coalescing on; undo.WithLimit(n), undo.WithCoalescing(false)
s.Push(undo.NewCommand("Write hello", do, undoIt))
s.Undo()
s.Redo()

mvvm.BindCommand(s.UndoCommand(), &undoBtn.OnClick, setEnabled)
mvvm.OneWay(s.UndoTextBinding(), &undoBtn.Text, repaint) // "Undo Write hello"
```

Una serie de pulsaciones de teclas se fusiona en un solo paso, y un push tras un
deshacer descarta la cola de rehacer.
