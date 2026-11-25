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

## La barre de menus {#menu-bar}

`application.Config.Menu` prend le même `tray.Menu` qu'une icône de zone de
notification et l'installe comme barre de menus de l'application ;
`application.SetMenu` la change ensuite, depuis n'importe quelle goroutine, et
ne la réinstalle que si le menu a changé. `tray.EditMenu()` est le menu Édition
standard (Annuler, Couper, Copier, Coller, Tout sélectionner).

- macOS : le menu en haut de l'écran, menu de l'application en premier.
- Windows : la barre de menus de la fenêtre ; les raccourcis sont affichés en texte.
- X11, Wayland, GTK, wasm et Android : `window.ErrNotSupported`, ignoré par
  `Run`. GTK est une lacune : la liaison GTK n'a pas encore de modèle de menu.

## Accessibilité des widgets dessinés {#painted-accessibility}

Un gestionnaire qui dessine ses propres widgets du
[toolkit]({{< relref "/rendering/toolkit.md#accessibility" >}}) n'est qu'un
rectangle opaque pour la couche d'accessibilité de la plateforme, sauf s'il
implémente `Accessible`. `application.A11yElementsOf(root)` construit cette
description à partir de l'arbre de widgets (rôle, libellé, valeur et bornes de
chaque widget, au plus `MaxA11yElements` = 1000) :

```go
func (v *View) A11yElements() []application.A11yElement {
	return application.A11yElementsOf(v.root)
}
```

Sous macOS, les boutons dessinés apparaissent alors dans l'arbre AX comme
`AXButton` avec leur libellé, et `AXPress` sur l'un d'eux équivaut à un clic ;
les contrôles natifs que le gestionnaire embarque restent dans l'arbre à leurs
côtés. Les back-ends Windows (UI Automation) et Linux (AT-SPI) consomment la
même description. Un widget désactivé est publié sous
`A11yElement.Disabled` : `AXEnabled` vaut false sous macOS, `IsEnabled` vaut
false dans UI Automation, et l'objet AT-SPI perd les états `ENABLED` et
`SENSITIVE` (un signal `StateChanged` annonce le changement). Le pont Windows
refuse toujours les rectangles d'éléments.

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

### Notifications

`(*Tray).Notify(tray.Notification{Title, Body, OnClick})` émet une notification système native : UNUserNotificationCenter sous macOS, `org.freedesktop.Notifications` sous Linux, une bulle Shell_NotifyIcon sous Windows. Le titre fait au plus 63 runes et le corps 255 (`MaxNotificationTitle`, `MaxNotificationBody`) ; un titre vide donne `ErrEmptyTitle`. Les erreurs nomment la cause : `ErrNoNotifier` (aucun back-end), `ErrNotRunning`, `ErrNotBundled` (macOS ne délivre que pour un paquet d'application) et `ErrNotAuthorized` (l'utilisateur a refusé). `tray.NewHeadless()` enregistre ce qui a été envoyé dans `Headless.Notified`, pour les tests.

## Boîtes de dialogue natives

`application.OpenFile`, `SaveFile`, `Alert` et `Confirm` affichent les boîtes de dialogue propres à la plateforme : NSOpenPanel, NSSavePanel et NSAlert sous macOS, GtkFileDialog et GtkAlertDialog (GTK 4.10 ou plus) sous GTK, IFileDialog et TaskDialog sous Windows.

```go
err := application.OpenFile(application.OpenOptions{
    Title: "Ouvrir", Multiple: true,
    Filters: []application.Filter{{Name: "Images", Extensions: []string{"png", "jpg"}}},
}, func(r application.OpenResult) { /* r.Paths, ou r.Err == application.ErrCancelled */ })
```

L'appel rend la main aussitôt et peut être fait depuis n'importe quelle goroutine ; la fonction de rappel s'exécute exactement une fois sur sa propre goroutine, jamais sur le fil de l'interface, et n'est pas appelée si l'appel lui-même renvoie une erreur. `ErrNoWindow` est renvoyée avant que `Run` ait ouvert la fenêtre, `ErrNotSupported` sur un back-end sans dialogues natifs (X11, Wayland, wasm), et `ErrInvalidDialog` pour une borne dépassée, un octet NUL ou un texte qui n'est pas de l'UTF-8. Les chemins sont rendus tels que la plateforme les donne, jamais résolus.

## Ouvrir un lien

`application.OpenURL(rawURL)` ouvre une URL `http` ou `https` dans le navigateur de l'utilisateur : NSWorkspace sous macOS, ShellExecuteW sous Windows, le portail OpenURI puis `xdg-open` sous Linux. L'URL doit avoir un hôte, un port (s'il y en a un) entre 1 et 65535 et au plus 8 Kio ; tout le reste est refusé avec `ErrURLScheme` ou `ErrURLInvalid`, et une plateforme sans lanceur renvoie `ErrOpenURLUnsupported`.
