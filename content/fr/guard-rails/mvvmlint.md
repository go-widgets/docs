---
title: "mvvmlint : l'état passe par le view-model"
linkTitle: "mvvmlint"
weight: 10
description: "Fait échouer une pull request qui affecte directement un champ d'état d'un widget de la boîte à outils, ou qui a des widgets et pas de view-model."
tags: [ci, lint, mvvm]
---

[`go-widgets/mvvmlint`](https://github.com/go-widgets/mvvmlint) ne se déclenche que dans
les paquets qui importent la boîte à outils.

1. **Un champ d'état d'un widget affecté à la main.** `entry.Text = "x"`, où le
   type statique de `entry` est un widget de la boîte à outils et `Text` un champ d'état, est
   signalé. `&entry.Text` — ce que prennent les [binders]({{< relref "/state/mvvm.md" >}})
   — n'est pas une affectation et n'est jamais signalé ; pas plus qu'un littéral
   composite (`toolkit.Entry{Text: "x"}`), ni la composition d'un callback tel que
   `OnChange`.
2. **Des widgets et pas de view-model.** Un paquet qui importe la boîte à outils mais pas
   `go-widgets/mvvm` reçoit un diagnostic. `-requirevm=false` désactive ce contrôle pour
   une bibliothèque de widgets qui, légitimement, n'a pas de view-model.

Les champs d'état sont ceux des widgets de la boîte à outils (`Text`, `Items`, `Rows`,
`Root`, `Selected`, `Value`, `Checked`, …) ; `-statefields=` remplace la liste.
Un fichier nommé `*_binding.go`, ou contenant `//mvvmlint:allow`, est exempté de
la première règle.

```sh
go install github.com/go-widgets/mvvmlint/cmd/mvvmlint@latest
go vet -vettool="$(go env GOPATH)/bin/mvvmlint" ./...
```

```yaml
# .github/workflows/mvvm.yml
name: mvvm
on:
  pull_request:
  push:
    branches: [main]
jobs:
  mvvm:
    uses: go-widgets/mvvmlint/.github/workflows/mvvmlint.yml@main
```

[`app-template`]({{< relref "/getting-started.md" >}}) est livré avec cette tâche.
