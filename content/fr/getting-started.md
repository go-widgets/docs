---
title: "Une première application"
weight: 5
description: "Partir d'app-template, déjà conforme à MVVM et au vert, ou ouvrir une simple fenêtre et y ajouter des widgets."
tags: [premiers pas, app-template, window]
---

## Depuis le modèle {#from-the-template}

[`go-widgets/app-template`](https://github.com/go-widgets/app-template) est un
dépôt modèle : une petite mais vraie application de navigateur, avec un champ de
recherche, un filtre par catégorie, la liste des lignes qui correspondent et une ligne d'état.
Tout ce dont une application plus grande a besoin est déjà branché : un view-model qui ne
contient aucun widget, une vue liée à celui-ci, un hôte wasm, une barrière de couverture à 100 %, et la
[barrière MVVM]({{< relref "/guard-rails/mvvmlint.md" >}}).

```sh
git clone https://github.com/go-widgets/app-template my-app
cd my-app
go mod edit -module github.com/you/my-app
./build.sh          # dist/app.wasm, dist/wasm_exec.js, dist/index.html
```

Servez `dist/` en HTTP : une page ouverte depuis `file://` ne peut pas instancier de wasm.

| Fichier | Ce qu'il contient |
|---|---|
| `viewmodel.go` | tout l'état, sous forme d'observables et de commandes [`mvvm`]({{< relref "/state/mvvm.md" >}}). Aucun widget. Testable sans canvas. |
| `scene.go` | les widgets, chacun lié au view-model par `mvvmtk`. Il n'affecte jamais lui-même un champ d'état d'un widget. |
| `main.go` | le seul fichier avec une étiquette de compilation (`js && wasm`) : il transmet les entrées à la boîte à outils et copie l'image dans un `<canvas>`. |

Modifiez les deux premiers et faites passer l'état par les binders : la tâche
`mvvm` de la CI fait échouer une pull request qui affecte directement un champ d'un widget.

## Depuis une simple fenêtre {#from-a-bare-window}

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

Une largeur et une hauteur laissées à zéro demandent au back-end une taille lisible par défaut ; sous
macOS, c'est une fraction de l'écran principal. `CGO_ENABLED=0 go build` produit
le binaire pour chaque plateforme, depuis n'importe quelle machine.

## Quel module pour quoi {#which-module-for-what}

| Vous voulez | Commencez par |
|---|---|
| une fenêtre de bureau | [`window`]({{< relref "/surfaces/native-window.md" >}}) |
| une application de bureau avec une icône de zone de notification et l'apparence du système | [`application`]({{< relref "/surfaces/application.md" >}}) |
| une page dans un navigateur | [`webcanvas`]({{< relref "/surfaces/browser.md" >}}), ou le modèle ci-dessus |
| un programme en terminal | [`tui`]({{< relref "/surfaces/terminal.md" >}}) |
| une application Android | [`android`]({{< relref "/surfaces/android.md" >}}) |
| une image d'un widget pour un README | [`svg`]({{< relref "/surfaces/snapshots.md" >}}) |

Pour voir le jeu de widgets avant d'écrire quoi que ce soit, ouvrez la
[galerie](https://go-widgets.github.io/gallery/) : chaque famille de widgets, en direct,
dans un canvas de navigateur.
