---
title: "Un terminal"
linkTitle: "Terminal"
weight: 40
description: "go-widgets/tui rend les widgets à travers le painter en cellules : un moteur de rendu d'instantanés, un exécuteur interactif, et des widgets dimensionnés pour une grille de caractères."
tags: [surfaces, tui, terminal]
---

[`go-widgets/tui`](https://github.com/go-widgets/tui) transforme un arbre de widgets en
image de terminal à travers le [painter en cellules]({{< relref "/rendering/painter.md" >}}).
Bibliothèque standard uniquement — pas de `x/term`, pas de cgo.

## Une image {#one-frame}

```go
_ = tui.RenderOnce(os.Stdout, widgets, nil) // nil theme: light
```

`RenderOnceSized(w, cols, rows, widgets, theme)` prend la taille explicitement ;
`RenderOnce` lit `COLUMNS` et `LINES` et se replie sur 80×24. Lire
l'environnement plutôt qu'appeler `TIOCGWINSZ` garde le paquet exempt de
code propre à chaque plateforme.

## Un programme interactif {#an-interactive-program}

```go
app := tui.NewApp()
app.Root = buildScene()
app.Keys["q"] = func(a *tui.App) { a.Quit() }
app.Keys["Ctrl+C"] = func(a *tui.App) { a.Quit() }
os.Exit(app.Run())
```

`Run` passe sur l'écran alternatif et en mode brut, distribue les entrées à `Root`
(les gestionnaires de touches globaux d'abord), suit `SIGWINCH`, et restaure toujours le
terminal — y compris lors d'une panique. Un gestionnaire de touche appelle `Consume` pour soustraire un événement à
`Root` ; `InputTarget` envoie chaque événement non consommé à un seul widget, ce qui permet
à une palette de commandes d'avaler la frappe tant qu'elle est ouverte. `NewFocusRing` donne à un
formulaire le parcours par Tab et Maj+Tab.

## Des widgets dimensionnés pour les cellules {#widgets-sized-for-cells}

La plupart des widgets de la boîte à outils sont conçus en pixels : leurs constantes de marge comptent
des cellules sous le painter en cellules et gonflent. `tui` a donc son propre jeu **natif en cellules**
— `TextEditor`, `Entry`, `Button`, `CheckButton`, `RadioButton`, `Scale`,
`ProgressBar`, `Sparkline`, `ListBox`, `TreeView`, `Table`, `MenuBar`,
`Notebook`, `HSplit`, `VSplit`, `Dialog`, `Dropdown`, `Spinner` et d'autres —
où un glyphe occupe une cellule. Chacun reste un `toolkit.Widget`, si bien que la même
instance dessine aussi en pixels.

`TextEditor` est un éditeur de code en lecture-écriture : gouttière, annuler et rétablir, rechercher et
remplacer, sélection, déplacement de lignes, et coloration syntaxique pour Go (via
`go/scanner`), JavaScript et TypeScript, Python, Ruby, shell, C et C++,
Rust, JSON, YAML, HCL, TOML, LaTeX et Markdown, sans aucune dépendance externe.

```sh
go run ./cmd/tui-widgets | less -R     # every cell-native widget, one per slot
go run ./cmd/tui-widget-explorer       # the same, interactive
go run ./cmd/tui-explorer              # a file browser
go run ./cmd/tui-editor --file=x.go    # a modal editor
```

Les deux programmes de démonstration sont testés de bout en bout dans un vrai pty : de vrais octets de touches en entrée,
l'image rendue vérifiée.
