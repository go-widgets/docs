---
title: "Modules"
weight: 50
description: "Les dix-neuf modules de go-widgets, le rôle de chacun, et où se trouvent sa source et sa référence d'API."
tags: [modules]
---

La référence d'API est générée à partir de la source par
[pkg.go.dev](https://pkg.go.dev/), qui suit les étiquettes publiées ; ces
pages expliquent, et ne la recopient pas. L'étiquette est la plus récente au moment de la
rédaction — la liste des étiquettes du dépôt lui-même fait foi.

| Module | Étiquette | Ce que c'est | Référence |
|---|---|---|---|
| [`painter`](https://github.com/go-widgets/painter) | v0.15.0 | [la jonction de dessin]({{< relref "/rendering/painter.md" >}}) : painters en pixels et en cellules | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/painter) |
| [`toolkit`](https://github.com/go-widgets/toolkit) | v0.328.0 | [le jeu de widgets]({{< relref "/rendering/toolkit.md" >}}), thèmes, mises en page, texte | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/toolkit) |
| [`skin`](https://github.com/go-widgets/skin) | v0.2.0 | [des widgets décrits par des données]({{< relref "/rendering/skin.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/skin) |
| [`isoicons`](https://github.com/go-widgets/isoicons) | v0.3.0 | jeux d'icônes isométriques (cloud-native, AWS) pour les diagrammes isométriques de la boîte à outils | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/isoicons) |
| [`mvvm`](https://github.com/go-widgets/mvvm) | v0.13.0 | [observables, commandes, binders, annulation]({{< relref "/state/mvvm.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvm) |
| [`mvvmtk`](https://github.com/go-widgets/mvvmtk) | v0.14.1 | [des binders en un appel pour les widgets de la boîte à outils]({{< relref "/state/mvvm.md#mvvmtk-one-call-per-widget" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmtk) |
| [`data`](https://github.com/go-widgets/data) | v0.3.0 | [enregistrements typés, requêtes, stockage local ou distant]({{< relref "/state/data.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/data) |
| [`window`](https://github.com/go-widgets/window) | v0.86.1 | [une fenêtre native]({{< relref "/surfaces/native-window.md" >}}) sur huit back-ends | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/window) |
| [`application`](https://github.com/go-widgets/application) | v0.7.0 | [le cycle de vie de l'application]({{< relref "/surfaces/application.md" >}}) au-dessus d'une fenêtre | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/application) |
| [`tray`](https://github.com/go-widgets/tray) | v0.14.0 | [une icône de zone de notification et son menu]({{< relref "/surfaces/application.md#the-tray" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tray) |
| [`webcanvas`](https://github.com/go-widgets/webcanvas) | v0.2.0 | [une scène dans un `<canvas>` de navigateur]({{< relref "/surfaces/browser.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/webcanvas) |
| [`tui`](https://github.com/go-widgets/tui) | v0.61.0 | [rendu en terminal et widgets natifs en cellules]({{< relref "/surfaces/terminal.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tui) |
| [`android`](https://github.com/go-widgets/android) | v0.15.0 | [un APK peint par un processus sans CGO]({{< relref "/surfaces/android.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/android) |
| [`svg`](https://github.com/go-widgets/svg) | v0.6.0 | [un rendu en SVG ou en PNG]({{< relref "/surfaces/snapshots.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/svg) |
| [`mvvmlint`](https://github.com/go-widgets/mvvmlint) | v0.4.0 | [le contrôle MVVM]({{< relref "/guard-rails/mvvmlint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmlint) |
| [`bricolint`](https://github.com/go-widgets/bricolint) | v0.4.0 | [le contrôle « pas d'interface dessinée à la main »]({{< relref "/guard-rails/bricolint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/bricolint) |
| [`app-template`](https://github.com/go-widgets/app-template) | v0.4.0 | [le point de départ d'une application]({{< relref "/getting-started.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/app-template) |
| [`gallery`](https://github.com/go-widgets/gallery) | v0.6.0 | la [démonstration en direct](https://go-widgets.github.io/gallery/) de chaque famille de widgets | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/gallery) |
| [`desktop`](https://github.com/go-widgets/desktop) | v0.19.0 | un shell de bureau qui assemble go-freedesktop et go-widgets, en natif et dans wasmdesk | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/desktop) |

## Ce que chaque module doit respecter {#what-every-module-is-held-to}

- `CGO_ENABLED=0` : pas de cgo, et pas d'appel à un outil en ligne de commande à la place
  d'une bibliothèque.
- Compilé de manière croisée pour amd64, arm64, riscv64, loong64, ppc64le et s390x — ce dernier
  en gros-boutiste, ce qui garde chaque encodage binaire honnête — et, quand cela
  s'applique, `js/wasm`. Les dix-sept bibliothèques et outils le font tous depuis le 2026-10-10 ;
  les deux autres sont des applications de navigateur ([État]({{< relref "/status.md#what-ci-does-not-cover-yet" >}})).
- Une barrière de couverture des instructions dans la CI : 100 %, dans les dix-neuf. Quand un
  fichier ne peut pas s'exécuter dans un test (une boucle de navigateur, une boucle d'exécution native), le module
  indique quel fichier est laissé de côté et pourquoi. Voir [État]({{< relref "/status.md" >}})
  pour les exceptions.
- BSD-3-Clause.
