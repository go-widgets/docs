---
title: "La boîte à outils"
linkTitle: "Toolkit"
weight: 20
description: "160 types de widgets dessinés à travers la jonction du painter : thèmes, mises en page, texte, saisie, accessibilité."
tags: [rendu, toolkit, widgets, thème, accessibilité]
---

[`go-widgets/toolkit`](https://github.com/go-widgets/toolkit) est le jeu de
widgets. Chaque widget implémente la même interface — `Bounds`, `SetBounds`,
`Draw(p painter.Painter, theme *Theme)`, `HitTest`, `OnEvent` — si bien qu'un hôte
traite de la même façon un bouton, un tableau et un calendrier d'événements. Bibliothèque standard uniquement,
`CGO_ENABLED=0`, 100 % de couverture des instructions.

En v0.328.0, elle compte **160 types de widgets exportés** en environ **69 000 lignes** de
code de widgets. Ces deux nombres sont mesurés à cette étiquette, pas reportés : le
README de la boîte à outils donne les commandes, et il vaut mieux les relancer que
répéter les chiffres.

## Les familles {#the-families}

| Famille | Widgets |
|---|---|
| Action | `Button`, `ToggleButton`, `CheckButton`, `RadioButton` + `RadioGroup`, `Switch`, `SplitButton`, `IconButton`, `CycleButton`, `Chip`, `SegmentedBar` |
| Saisie | `Entry`, `TextView` (sélection, aperçu IME, plages de syntaxe, numéros de ligne), `SpinButton`, `Scale`, `RangeSlider`, `SearchEntry`, `TagField`, `ComboBox`, `FormField` avec `Validate`/`Rule` |
| Sélection | `ListBox`, `TreeView`, `DropDown` |
| Conteneurs | `Container` avec un `Layout` interchangeable (`FitLayout`, `BoxLayout`, `BorderLayout`, `CardLayout`, `FlowLayout`) ; `HBox`, `VBox`, `Grid`, `Frame`, `Dock`, `Border`, `Stack`, `Overlay`, `Paned`, `Expander`, `Accordion` ; un constructeur déclaratif `Node` |
| Navigation | `Menu`, `MenuBar` (avec `Alt`+lettre), `ContextMenu`, `Popover`, `CommandPalette`, `Dialog`, `MessageDialog`, `Wizard`, `Notebook`, `ViewSwitcher`, `Carousel` |
| Retour | `ProgressBar`, `ProgressCircle`, `LevelBar`, `Spinner`, `Tooltip`, `Notification`, `Toast`, `Banner`, `Alert`, `Badge`, `Skeleton` |
| Données | `Table` (édition de cellules, colonnes figées, lignes de groupe, agrégats, lignes dépliables), `TreeTable`, `PropertyGrid`, `Kanban`, `Gantt`, `Agenda` (semaine, mois, trimestre, année ; plusieurs calendriers) |
| Graphiques | `LineChart`, `BarChart`, `PieChart`, `AreaChart`, `ScatterChart`, `RadarChart`, `Gauge`, `Sparkline` |
| Composites | `FileChooser`, `ColorChooser`, `ColorPicker`, `FontChooser`, `Calendar`, `DatePicker`, `DateRangePicker`, `TimePicker`, `MarkdownView`, `MarkdownEditor`, `TerminalView` |
| Shell | `Window` avec décorations côté client, `HeaderBar`, `Toolbar`, `Statusbar`, `StatusIcon`, `StatusArea`, `Wallpaper`, `Thumbnail` |

Tous, en direct : la [galerie](https://go-widgets.github.io/gallery/).

## Thèmes {#themes}

Une seule valeur `Theme` se propage en cascade dans tout l'arbre : changez une couleur à la
racine et chaque widget se repeint avec elle. `DefaultLight()` et `DefaultDark()`
sont intégrés, et `LoadGTKTheme(css)` lit le bloc `@define-color` de n'importe quel
thème GTK 3 ou libadwaita — Adwaita, WhiteSur, Solarized — dans un `Theme`.

## Texte {#text}

Par défaut, la police compilée est une police bitmap 5×7, pour que les tests au pixel gardent leur
géométrie. Un seul appel fait passer toute l'interface à un texte anticrénelé et mis en forme :

```go
toolkit.UseOpenTypeText() // Atkinson Hyperlegible, bundled; js/wasm-safe
```

La police vient de [go-opentype/fonts](https://github.com/go-opentype/fonts)
et est rastérisée par [go-opentype](https://github.com/go-opentype/opentype),
en Go, sans bibliothèque de polices en C ni recherche de polices système.
`NewTrueTypeFont(ttf, px)` charge une autre police ; `NewFallbackFont` en enchaîne
plusieurs, par exemple pour ajouter une police CJK ou arabe ; le champ `Font` propre à un widget
remplace la police globale pour ce seul widget.

## Saisie {#input}

Souris, clavier, composition IME, tactile (`EventTouchStart`/`Move`/`End`, avec
`GestureRecognizer` pour les tapes, les appuis longs et les balayages), glisser-déposer
(`DragSource`, `DropTarget`). La sélection de plusieurs lignes dans `ListBox`, `Table` et
`TreeView` fonctionne à la souris comme au clavier, et respecte ⌘ aussi bien
que Ctrl, car sous macOS Ctrl-clic est le clic secondaire.

## Accessibilité {#accessibility}

Une boîte à outils qui peint ses propres pixels ne donne rien à lire à un lecteur d'écran
à moins de dire ce qu'elle a dessiné. Les widgets implémentent `Accessible` — un rôle, un nom,
une valeur — et `CollectA11y` les rassemble pour que l'hôte les publie : AT-SPI sous
Linux via [`window`]({{< relref "/surfaces/native-window.md" >}}), et une
hiérarchie de vues virtuelle sur [Android]({{< relref "/surfaces/android.md#accessibility" >}}).
