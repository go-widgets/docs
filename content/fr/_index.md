---
title: "go-widgets"
linkTitle: "Accueil"
type: docs
cascade:
  type: docs
description: "Une boîte à outils de widgets en Go pur, et toutes les surfaces sur lesquelles elle peint : une fenêtre native sur X11, Wayland, macOS et Windows, un canvas de navigateur, un terminal, un APK Android."
---

**Un seul arbre de widgets, peint par Go, sur toutes les surfaces que Go atteint.** Une
fenêtre native sur X11, Wayland, macOS et Windows ; un `<canvas>` dans un onglet de
navigateur ; un terminal ; un APK Android. Pas de chaîne de compilation C, pas de webview, pas
de jeu de widgets du système : chaque module se compile avec `CGO_ENABLED=0`, et chaque pixel
est dessiné par la boîte à outils.

```go
w, err := window.Open(window.Config{Title: "Hello"})
if err != nil {
	return err
}
defer w.Close()

box := toolkit.NewVBox()
box.Append(toolkit.NewLabel("Hello from go-widgets"))
box.Append(toolkit.NewButton("Click me", func() { /* ... */ }))
return w.Run(box) // an X11 or Wayland window, an NSWindow, a Win32 window
```

C'est
[`cmd/windowdemo`](https://github.com/go-widgets/window/blob/main/cmd/windowdemo/main.go),
raccourci. Le même `box` dessiné dans un navigateur, un terminal ou un PNG, c'est le même
code avec une autre surface en dessous.

## Pourquoi une seule boîte à outils dessine tout {#why-one-toolkit-draws-everything}

Un widget qui appelle le bouton propre à la plateforme n'est pas plus portable que la
plateforme la moins capable qu'il vise, et son apparence change sur chacune. go-widgets prend
le chemin inverse : la méthode `Draw` d'un widget ne parle jamais qu'à un
[`painter.Painter`]({{< relref "/rendering/painter.md" >}}), une poignée de
primitives (remplir un rectangle, en tracer le contour, poser un pixel, dessiner du texte). Que
ces appels deviennent des pixels RGBA ou des cellules de terminal, c'est l'affaire du back-end,
pas celle du widget.

Ce que cela coûte est dit franchement : pas d'apparence native par défaut, et pas
d'accessibilité gratuite — la boîte à outils publie son propre
[arbre d'accessibilité]({{< relref "/rendering/toolkit.md#accessibility" >}})
vers chaque plateforme. Ce que cela apporte : un seul jeu de widgets, testé une fois, au
comportement identique partout, et un binaire qui se compile de manière croisée depuis
n'importe quelle machine vers chacune d'elles.

## Pour aller plus loin {#where-to-go-next}

| | |
|---|---|
| [Une première application]({{< relref "/getting-started.md" >}}) | depuis `app-template`, ou depuis une simple fenêtre |
| [Comment un widget atteint l'écran]({{< relref "/rendering/_index.md" >}}) | la jonction du painter, le jeu de widgets, les skins |
| [La jonction du painter]({{< relref "/rendering/painter.md" >}}) | les primitives avec lesquelles chaque widget est dessiné |
| [La boîte à outils]({{< relref "/rendering/toolkit.md" >}}) | 160 types de widgets, thèmes, mises en page, texte, accessibilité |
| [Skins]({{< relref "/rendering/skin.md" >}}) | les parties et les états d'un widget dans un fichier de données |
| [L'état vit dans un view-model]({{< relref "/state/_index.md" >}}) | pourquoi on n'affecte pas les champs d'un widget à la main |
| [MVVM]({{< relref "/state/mvvm.md" >}}) | observables, commandes, binders, annulation |
| [La colonne vertébrale des données]({{< relref "/state/data.md" >}}) | enregistrements typés, requêtes, stockage local ou distant |
| [Où elle peint]({{< relref "/surfaces/_index.md" >}}) | chaque back-end et la façon dont il est choisi |
| [Une fenêtre native]({{< relref "/surfaces/native-window.md" >}}) | X11, Wayland, Cocoa, Win32, GTK4, Android, wasmbox, un onglet de navigateur |
| [Une application et son icône de zone de notification]({{< relref "/surfaces/application.md" >}}) | cycle de vie, apparence, une icône dans la barre des menus |
| [Un onglet de navigateur]({{< relref "/surfaces/browser.md" >}}) | un simple `<canvas>`, ou une fenêtre wasmdesk |
| [Un terminal]({{< relref "/surfaces/terminal.md" >}}) | des widgets natifs en cellules et un exécuteur interactif |
| [Android]({{< relref "/surfaces/android.md" >}}) | un vrai APK peint par un processus sans CGO |
| [Instantanés]({{< relref "/surfaces/snapshots.md" >}}) | un widget sous forme de SVG ou de PNG |
| [Règles imposées par la CI]({{< relref "/guard-rails/_index.md" >}}) | `mvvmlint` et `bricolint` |
| [Modules]({{< relref "/modules.md" >}}) | les dix-neuf, avec leurs sources et leurs références |
| [État]({{< relref "/status.md" >}}) | ce qui est vérifié, où, et ce qui n'est pas fait |

## Licence {#licence}

BSD-3-Clause.
