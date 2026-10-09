---
title: "mvvmlint: Zustand geht über das View-Model"
linkTitle: "mvvmlint"
weight: 10
description: "Lässt einen Pull Request scheitern, der ein Zustandsfeld eines Toolkit-Widgets direkt zuweist oder Widgets, aber kein View-Model hat."
tags: [ci, lint, mvvm]
---

[`go-widgets/mvvmlint`](https://github.com/go-widgets/mvvmlint) schlägt nur in
Paketen an, die das Toolkit importieren.

1. **Ein von Hand zugewiesenes Zustandsfeld eines Widgets.** `entry.Text = "x"`, wobei der
   statische Typ von `entry` ein Toolkit-Widget und `Text` ein Zustandsfeld ist, wird
   gemeldet. `&entry.Text` – was die [Binder]({{< relref "/state/mvvm.md" >}})
   entgegennehmen – ist keine Zuweisung und wird nie gemeldet; ebenso wenig ein zusammengesetztes
   Literal (`toolkit.Entry{Text: "x"}`) oder das Zusammensetzen eines Callbacks wie
   `OnChange`.
2. **Widgets und kein View-Model.** Ein Paket, das das Toolkit importiert, aber nicht
   `go-widgets/mvvm`, erhält eine Diagnose. `-requirevm=false` schaltet das für
   eine Widget-Bibliothek ab, die berechtigterweise kein View-Model hat.

Die Zustandsfelder sind die der Toolkit-Widgets (`Text`, `Items`, `Rows`,
`Root`, `Selected`, `Value`, `Checked`, …); `-statefields=` ersetzt die Liste.
Eine Datei namens `*_binding.go` oder eine, die `//mvvmlint:allow` enthält, ist von
der ersten Regel ausgenommen.

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

[`app-template`]({{< relref "/getting-started.md" >}}) wird mit diesem Job ausgeliefert.
