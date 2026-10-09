---
title: "mvvmlint: el estado pasa por el view-model"
linkTitle: "mvvmlint"
weight: 10
description: "Rechaza una pull request que asigna directamente un campo de estado de un widget del toolkit, o que tiene widgets y ningún view-model."
tags: [ci, lint, mvvm]
---

[`go-widgets/mvvmlint`](https://github.com/go-widgets/mvvmlint) solo actúa en
los paquetes que importan el toolkit.

1. **Un campo de estado de un widget asignado a mano.** `entry.Text = "x"`,
   donde el tipo estático de `entry` es un widget del toolkit y `Text` es un
   campo de estado, se señala. `&entry.Text` —lo que reciben los
   [binders]({{< relref "/state/mvvm.md" >}})— no es una asignación y nunca se
   señala; tampoco un literal compuesto (`toolkit.Entry{Text: "x"}`), ni la
   composición de un callback como `OnChange`.
2. **Widgets y ningún view-model.** Un paquete que importa el toolkit pero no
   `go-widgets/mvvm` recibe un diagnóstico. `-requirevm=false` lo desactiva para
   una biblioteca de widgets que, con razón, no tiene view-model.

Los campos de estado son los de los widgets del toolkit (`Text`, `Items`,
`Rows`, `Root`, `Selected`, `Value`, `Checked`, …); `-statefields=` sustituye la
lista. Un archivo llamado `*_binding.go`, o que contenga `//mvvmlint:allow`,
queda exento de la primera regla.

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

[`app-template`]({{< relref "/getting-started.md" >}}) incluye este trabajo de serie.
