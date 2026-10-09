---
title: "Une application et son icône de zone de notification"
linkTitle: "Application et zone de notification"
weight: 20
description: "go-widgets/application gère le cycle de vie au-dessus d'une fenêtre ; go-widgets/tray place une icône et un menu dans la barre des menus ou la zone de notification."
tags: [surfaces, application, zone de notification]
---

## L'application {#the-application}

[`go-widgets/application`](https://github.com/go-widgets/application) est la
couche au-dessus de [`window`]({{< relref "/surfaces/native-window.md" >}}) : elle ouvre
la fenêtre, pilote sa boucle, attache une icône de zone de notification une fois la première image affichée,
transmet à l'application l'apparence du système en direct (sombre ou claire, couleur d'accent, police système),
publie l'arbre d'accessibilité, et appelle `onReady`
une fois la première image à l'écran.

```go
spec := application.Spec{
	Name:       "News Reader",
	Identifier: "com.example.reader",
	Version:    "1.0.0",
	Tray: func() *tray.Menu {
		return tray.NewMenu().Add(
			tray.Item("Refresh", refresh),
			tray.Item("Quit", quit),
		)
	},
}
cfg := application.Config{Title: "News Reader", Width: 1200, Height: 800}
err := application.Run(spec, cfg, handler, func() { log.Println("on screen") })
```

`handler` implémente `Handler` — `Frame`, `Resize`, les événements de souris, de défilement et de
clavier — et peut implémenter n'importe laquelle des capacités optionnelles : apparence,
raccourcis, clics et touches avec modificateurs, clic secondaire, menus contextuels,
presse-papiers, accessibilité, contrôles natifs. La boucle n'enveloppe jamais le handler ;
elle fait une assertion de type pour chaque capacité et honore celles qu'elle trouve. Un test parcourt
le paquet à la recherche de ces assertions et échoue si la documentation du paquet ne
les nomme pas toutes.

Une icône de zone de notification qui ne peut pas être attachée est une erreur de `Run`, pas un silence :
`errors.Is(err, tray.ErrNoBackend)` dit pourquoi. Une application qui veut sa
fenêtre quoi qu'il arrive à l'icône laisse `Spec.Tray` à nil et en lance une
elle-même.

`Run` ferme sa fenêtre quand la boucle se termine, si bien que l'« Open » d'une application
de zone de notification peut rappeler `Run` pour une nouvelle fenêtre. Pour un hôte qui n'est pas une
fenêtre — un shell de bureau, un onglet de navigateur — `Bind` et `BindScaled` renvoient une
`*toolkit.Surface` branchée sur le même handler.

## La zone de notification {#the-tray}

Une icône de zone de notification relève de l'intégration au système, pas d'un widget peint ;
[`go-widgets/tray`](https://github.com/go-widgets/tray) pilote donc l'API propre à
chaque plateforme, toujours avec `CGO_ENABLED=0` :

| Plateforme | API | Par |
|---|---|---|
| macOS | `NSStatusItem`, `NSMenu` | purego et le runtime Objective-C |
| Windows | `Shell_NotifyIcon`, `TrackPopupMenu` | `golang.org/x/sys/windows` |
| Linux | `StatusNotifierItem`, `com.canonical.dbusmenu` | DBus, en Go |

```go
menu := tray.NewMenu().Add(
	tray.Item("Open", open),
	tray.Checkbox("Notifications", true, setNotify),
	tray.SubMenu("Recent", tray.NewMenu().Add(tray.Item("file.txt", nil))),
	tray.Separator(),
	tray.Item("Quit", func() { t.Quit() }),
)
t := tray.New(iconPNG).SetTooltip("My App").SetMenu(menu)
t.Run() // blocks on the platform's loop until Quit
```

Un programme qui fait déjà tourner la boucle d'une fenêtre ne peut pas céder son thread principal à `Run` :
`Attach` affiche la même icône et rend la main, et `Quit` la retire.

Les back-ends natifs sont activés par défaut ; il n'y a pas d'étiquette de compilation à retenir.
Sur une plateforme qui n'en a aucun, `Run` renvoie `ErrNoBackend` — toute la différence
entre « il n'y a pas de zone de notification ici » et une icône qui, en silence, ne fait rien. Un test
ou un service headless passe `WithBackend(tray.NewHeadless())`.
