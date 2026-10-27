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

En v0.329.0, elle compte **160 types de widgets exportés** en environ **69 000 lignes** de
code de widgets. Ces deux nombres sont mesurés à cette étiquette, pas reportés : le
README de la boîte à outils donne les commandes, et il vaut mieux les relancer que
répéter les chiffres.

## Les familles {#the-families}

| Famille | Widgets |
|---|---|
| Action | `Button`, `ToggleButton`, `CheckButton`, `RadioButton` + `RadioGroup`, `Switch`, `SplitButton`, `IconButton`, `CycleButton`, `Chip`, `SegmentedBar` |
| Saisie | `Entry`, `TextView` (sélection, aperçu IME, plages de syntaxe, numéros de ligne), `SpinButton`, `Scale`, `RangeSlider`, `SearchEntry`, `TagField`, `ComboBox`, `FormField` avec `Validate`/`Rule` |
| Sélection | `ListBox`, `TreeView`, `DropDown` |
| Conteneurs | `Container` avec un `Layout` interchangeable (`FitLayout`, `BoxLayout`, `BorderLayout`, `CardLayout`, `FlowLayout`) ; `HBox`, `VBox`, `Grid`, `Frame`, `Dock`, `Border`, `Stack`, `Overlay`, `Paned`, `Splitter`, `Expander`, `Accordion` ; un constructeur déclaratif `Node` |
| Navigation | `Menu`, `MenuBar` (avec `Alt`+lettre), `ContextMenu`, `Popover`, `CommandPalette`, `Dialog`, `MessageDialog`, `Wizard`, `Notebook`, `ViewSwitcher`, `Carousel` |
| Retour | `ProgressBar`, `ProgressCircle`, `LevelBar`, `Spinner`, `Tooltip`, `Notification`, `Toast`, `Banner`, `Alert`, `Badge`, `Skeleton` |
| Données | `Table` (édition de cellules, colonnes figées, lignes de groupe, agrégats, lignes dépliables), `TreeTable`, `PropertyGrid`, `Kanban`, `Gantt`, `Agenda` (semaine, mois, trimestre, année ; plusieurs calendriers) |
| Graphiques | `LineChart`, `BarChart`, `PieChart`, `AreaChart`, `ScatterChart`, `RadarChart`, `Gauge`, `Sparkline` |
| Composites | `FileChooser`, `ColorChooser`, `ColorPicker`, `FontChooser`, `Calendar`, `DatePicker`, `DateRangePicker`, `TimePicker`, `MarkdownView`, `MarkdownEditor`, `TerminalView` |
| Shell | `Window` avec décorations côté client, `HeaderBar`, `Toolbar`, `Ribbon`, `Statusbar`, `StatusIcon`, `StatusArea`, `Wallpaper`, `Thumbnail` |

Tous, en direct : la [galerie](https://go-widgets.github.io/gallery/).

## Ruban {#ribbon}

`Ribbon` est un ruban de commandes à la manière d'Office : des onglets de
groupes étiquetés contenant de grands boutons (icône au-dessus du texte), de
petits (icône à côté), des bascules, des boutons scindés, des listes
déroulantes et une petite galerie. Il suit les recommandations des rubans
Windows : quand la fenêtre se rétrécit, chaque groupe se réduit selon sa
priorité — complet, compact, icônes seules, puis un seul bouton de menu — de
sorte que le ruban montre la plus grande combinaison qui tient ; il se replie
sur ses onglets (Ctrl+F1, ou le chevron) et un onglet laisse alors entrevoir
son contenu ; Alt affiche les astuces clavier ; les info-bulles donnent le
raccourci et une description. La navigation par onglets et par barre d'outils
suit les motifs WAI-ARIA, et ses contours et son anneau de focus gardent un
contraste de 3:1 en thème clair et sombre.

Chaque contrôle est une vue sur une `Action` — la commande unique que se
partagent un menu, une palette et le ruban — et `NewCommandAction` en fabrique
une à partir d'une `mvvm.Command` du modèle de vue, dont `CanExecute` grise le
contrôle. L'onglet actif, l'état replié, une bascule et le choix d'une liste
sont des `mvvm.Observable`. Placé sous un `PopoverHost`, ses menus se dessinent
au-dessus de la page sans code d'hôte.

`RibbonGrid` est un genre de contrôle dont le menu est une grille de `GridCols`
sur `GridRows` cellules (8 sur 6 si zéro, 16 sur 16 au plus), comme le bouton
Tableau d'un traitement de texte. Le pointeur ou les flèches allument les
cellules du coin jusqu'à celle qui est dessous et une légende dit « 3 × 2 » ;
un clic ou Entrée choisit et appelle `OnGrid(cols, rows)` ; Début et Fin
sautent, Échap ferme. Il se dessine en thème clair et sombre et est signalé aux
technologies d'assistance comme un bouton déplié tant que la grille est ouverte.

```go
&toolkit.RibbonControl{Kind: toolkit.RibbonGrid, Label: "Table",
	GridCols: 8, GridRows: 8,
	OnGrid: insertTable}
```

## Séparateur {#splitter}

`Splitter` dispose N volets côte à côte (ou l'un au-dessus de l'autre avec
`vertical` à vrai) et place entre chaque paire voisine un séparateur
déplaçable à la souris et focalisable, d'après le motif WAI-ARIA « window
splitter ». Un volet est soit **fixe** — sa `Size`, en pixels logiques, est un
`mvvm.Observable[int]` que le séparateur écrit — soit **flexible**, et se
partage ce que les volets fixes laissent selon son `Weight`, lui aussi un
`Observable`. Chaque volet a un `Min` et un `Max`. Un volet fixe `Collapsible`
se ferme (taille 0) quand on tire le séparateur sous la moitié de son `Min`, ou
par double-clic ou Entrée, et se rouvre de la même façon. L'observable
`Visible` d'un volet l'éteint avec son séparateur. Les tailles vivant dans des
observables, un modèle de vue peut les lier, les enregistrer et les restaurer ;
`OnResized` est appelé une fois par geste terminé avec les volets que la
personne a changés, jamais pour une disposition qui a simplement suivi le
redimensionnement de la fenêtre.

```go
files := mvvm.NewObservable(240) // fixed pane, logical px
w := mvvm.NewObservable(1)       // flexible pane, weight

s := toolkit.NewSplitter(false,
	&toolkit.SplitPane{Widget: tree, Name: "Files", Size: files,
		Min: 120, Max: 480, Collapsible: true},
	&toolkit.SplitPane{Widget: editor, Name: "Editor", Weight: w, Min: 200},
)
s.OnResized = func(changed []int) { save(files.Get(), w.Get()) }
```

Un séparateur prend le focus : les flèches (haut et bas en vertical) le
déplacent de `Step` pixels logiques, quatre fois plus avec Maj ; Début et Fin
amènent le volet qui le précède à sa plus petite et sa plus grande taille ;
Entrée ferme ou rétablit le volet fixe qu'il commande ; Tab passe d'un
séparateur à l'autre. Chaque séparateur est un `RoleSeparator` avec
`aria-valuemin`, `aria-valuemax` et `aria-valuenow`, nommé « Resize » suivi du
`Name` du volet. `Cursor(x, y)` renvoie `"col-resize"` ou `"row-resize"` au-dessus
d'un séparateur, et `SplitterColors(theme)` donne le trait et l'accent du
séparateur, qui gardent un contraste de 3:1 contre le fond et la surface dans
les deux thèmes par défaut, comme l'anneau de focus.

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

## Taille du binaire : un programme lie ce qu'il utilise {#binary-size}

La boîte à outils est un grand paquet, et l'éditeur de liens s'en accommode :
l'élimination de code mort de Go ne garde que ce qui est atteignable depuis
`main.main` **et depuis les tâches d'initialisation de chaque paquet importé**.
Les widgets qu'un programme ne touche jamais sont retirés. Ce qui n'est pas
retiré, c'est tout ce qu'atteint un initialiseur de niveau paquet ou le `init`
d'un paquet importé ; depuis la v0.345.0, la boîte à outils garde donc ses
propres initialiseurs sans travail et évite d'importer des paquets dont le
`init` entraîne du code :

- `Material` et `Thumbnail` utilisent
  [`github.com/go-images/images/ops`](https://github.com/go-images/images), sans
  codec, et non `github.com/go-images/images`, dont les décodeurs
  s'enregistrent dans `init` (environ 1,7 Mo de code WebAssembly).
- `CollabText`, `NewCollabText`, `MetaName` et `MetaColor` sont passés dans
  **`github.com/go-widgets/toolkit/collabui`** : ils demandent une session
  `go-crdt/collab` vivante, dont les transports tirent `net/http` et
  `crypto/tls`. `CollabSheet` et `IsoCRDTDocument` ne demandent que
  `go-crdt/crdt` et restent dans `toolkit`.
- La regexp de la `Rule` e-mail, le registre d'icônes iso intégré et la table
  de boutons de `RichEditorToolbar` sont construits à la première utilisation,
  non par des initialiseurs.

⛔ **Le déplacement est une rupture, sans redirection dépréciée.** Changez
l'import des quatre noms ci-dessus vers `collabui` ; rien d'autre ne change.

Tailles pour `GOOS=js GOARCH=wasm` avec `-ldflags="-s -w"`, brut et gzip -9 :

| Programme | avant (brut / gzip) | après (brut / gzip) |
|---|---|---|
| un `Label` (v0.345.0) | 10,49 / 2,88 Mo | 7,48 / 2,13 Mo |
| un `Label` (v0.345.1) | 10,49 / 2,88 Mo | 5,66 / 1,61 Mo |
| go-wasmcontainers/dashboard | 20,42 / 5,17 Mo | 17,72 / 4,58 Mo |
| la carte de flux du relais | 22,62 / 5,44 Mo | 19,95 / 4,85 Mo |

Ce qui reste, c'est surtout le runtime et la pile OpenType dont le rendu du
texte a besoin. Pour voir ce qui retient un symbole dans votre programme,
compilez avec `-ldflags=-dumpdep` et suivez la chaîne de premiers parents du
symbole jusqu'à `main.main` ou `go:main.inittasks`.

La v0.345.0 ajoute aussi `ColorHex` et `ParseColorHex`, pour écrire une couleur
en hexadécimal et la relire.

## Accessibilité {#accessibility}

Une boîte à outils qui peint ses propres pixels ne donne rien à lire à un lecteur d'écran
à moins de dire ce qu'elle a dessiné. Les widgets implémentent `Accessible` — un rôle, un nom,
une valeur — et `CollectA11y` les rassemble pour que l'hôte les publie : AT-SPI sous
Linux via [`window`]({{< relref "/surfaces/native-window.md" >}}), et une
hiérarchie de vues virtuelle sur [Android]({{< relref "/surfaces/android.md#accessibility" >}}).
