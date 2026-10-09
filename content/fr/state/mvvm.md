---
title: "MVVM : observables, commandes, binders"
linkTitle: "MVVM"
weight: 10
description: "go-widgets/mvvm, ses binders pour la boîte à outils et le terminal, les aides en un appel de mvvmtk, et une pile d'annulation."
tags: [état, mvvm, mvvmtk, annulation]
---

[`go-widgets/mvvm`](https://github.com/go-widgets/mvvm) repose uniquement sur les génériques, sans
réflexion. Son paquet central n'importe aucun paquet de widgets, si bien qu'un même
view-model pilote aussi bien la [boîte à outils]({{< relref "/rendering/toolkit.md" >}}) en pixels
que les [widgets de terminal]({{< relref "/surfaces/terminal.md" >}}).

| Primitive | Rôle |
|---|---|
| `Observable[T]` | une propriété : `Get`, `Set`, `Subscribe`. Affecter une valeur égale ne fait rien. |
| `Command` | une action, avec `CanExecute` et `RaiseCanExecuteChanged` |
| `ObservableList[T]` | une collection qui signale chaque insertion, suppression, remplacement, déplacement et réinitialisation |

## Un formulaire {#a-form}

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

## Liaison {#binding}

Les binders génériques atteignent un widget par un **pointeur vers son champ de valeur et
un pointeur vers son emplacement de callback** : `BindField` (bidirectionnel), `OneWay`,
`BindCommand`, `BindList`. Un widget déclenche son callback depuis son gestionnaire d'événements
et laisse silencieuse une écriture directe dans le champ, et `Observable.Set` ignore une valeur
égale — si bien qu'une liaison bidirectionnelle peut renvoyer l'écho sans récursion.

La boîte à outils fait évoluer ses widgets pour qu'ils exposent leur état **sous forme** d'un
observable (`Label.Text()` renvoie un `*mvvm.Observable[string]`). Il n'y a
alors plus de champ à initialiser ni de callback à composer, seulement deux propriétés qui
doivent s'accorder : `BindTwoWay(src, dst, invalidate)`.

Les widgets dont le callback prend plusieurs arguments reçoivent un petit adaptateur nommé dans un
paquet propre à chaque back-end, les seuls paquets qui importent un back-end :

| Paquet | Adaptateurs |
|---|---|
| `mvvm/tkbind` | `BindRange` (le `RangeSlider` à deux poignées), `BindContainer` et `BindCardActive` (les enfants d'un conteneur et une mise en page en cartes, depuis une liste ou un index), `BindStore` et `BindTable` (une `Table` au-dessus d'un [stockage de données]({{< relref "/state/data.md" >}}) : tri, regroupement et modifications en ligne validées) |
| `mvvm/tuibind` | `BindDropdown`, `BindTableSelection` |

## mvvmtk : un appel par widget {#mvvmtk-one-call-per-widget}

[`go-widgets/mvvmtk`](https://github.com/go-widgets/mvvmtk) renseigne les vrais noms de champ et de
callback de chaque widget de la boîte à outils, si bien que la vue ne les nomme jamais :

```go
unbind := mvvmtk.BindText(search, vm.Query, win.Invalidate) // search.Text ⇄ vm.Query
defer unbind()
```

| Aide | Sens |
|---|---|
| `BindText` (`SearchEntry`), `BindEntryText`, `BindChecked`, `BindSelectedIndex`, `BindSpin`, `BindListSelection`, `BindViewSwitcher` | bidirectionnel |
| `BindLabel`, `BindProgress` | view-model → widget |
| `BindListItems`, `BindDropDownOptions`, `BindViews`, `BindTree` | liste → widget |
| `BindCommand` | le clic d'un bouton, son état `Disabled` et son grisage suivent la commande |

Chaque aide renvoie un `unbind` qui détache la liaison et restaure le
callback qui était là auparavant. Un bouton lié à une commande qui ne peut pas s'exécuter
n'accepte ni clic, ni Entrée ou Espace, ni focus clavier.

## Annuler et rétablir {#undo-and-redo}

`mvvm/undo` est une pile de commandes construite sur les seules primitives centrales ; elle
fonctionne donc sous n'importe quelle vue :

```go
s := undo.New() // unlimited, coalescing on; undo.WithLimit(n), undo.WithCoalescing(false)
s.Push(undo.NewCommand("Write hello", do, undoIt))
s.Undo()
s.Redo()

mvvm.BindCommand(s.UndoCommand(), &undoBtn.OnClick, setEnabled)
mvvm.OneWay(s.UndoTextBinding(), &undoBtn.Text, repaint) // "Undo Write hello"
```

Une suite de frappes se fusionne en une seule étape, et un ajout après une annulation abandonne
la suite des actions à rétablir.
