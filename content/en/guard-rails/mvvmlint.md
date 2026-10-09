---
title: "mvvmlint: state goes through the view-model"
linkTitle: "mvvmlint"
weight: 10
description: "Fails a pull request that assigns a toolkit widget's state field directly, or that has widgets and no view-model."
tags: [ci, lint, mvvm]
---

[`go-widgets/mvvmlint`](https://github.com/go-widgets/mvvmlint) fires only in
packages that import the toolkit.

1. **A widget's state field assigned by hand.** `entry.Text = "x"`, where the
   static type of `entry` is a toolkit widget and `Text` is a state field, is
   flagged. `&entry.Text` — what the [binders]({{< relref "/state/mvvm.md" >}})
   take — is not an assignment and is never flagged; neither is a composite
   literal (`toolkit.Entry{Text: "x"}`), nor composing a callback such as
   `OnChange`.
2. **Widgets and no view-model.** A package that imports the toolkit but not
   `go-widgets/mvvm` gets one diagnostic. `-requirevm=false` turns this off for
   a widget library that legitimately has no view-model.

The state fields are those of the toolkit's widgets (`Text`, `Items`, `Rows`,
`Root`, `Selected`, `Value`, `Checked`, …); `-statefields=` replaces the list.
A file named `*_binding.go`, or containing `//mvvmlint:allow`, is exempt from
the first rule.

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

[`app-template`]({{< relref "/getting-started.md" >}}) ships with this job.
