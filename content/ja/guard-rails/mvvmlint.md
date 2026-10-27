---
title: "mvvmlint: 状態はビューモデルを通す"
linkTitle: "mvvmlint"
weight: 10
description: "ツールキットのウィジェットの状態フィールドを直接代入する、またはウィジェットがあるのにビューモデルがないプルリクエストを失敗させます。"
tags: [ci, lint, mvvm]
---

[`go-widgets/mvvmlint`](https://github.com/go-widgets/mvvmlint) が発動するのは、ツールキットをインポートしているパッケージだけです。

1. **ウィジェットの状態フィールドの手作業による代入。** `entry.Text = "x"` のように、`entry` の静的な型がツールキットのウィジェットで、`Text` が状態フィールドである場合に指摘されます。[バインダ]({{< relref "/state/mvvm.md" >}}) が受け取る `&entry.Text` は代入ではないため、指摘されることはありません。複合リテラル（`toolkit.Entry{Text: "x"}`）や、`OnChange` のようなコールバックの合成も同様です。
2. **ウィジェットがあるのにビューモデルがない。** ツールキットをインポートしているのに `go-widgets/mvvm` をインポートしていないパッケージには、診断が 1 件出ます。ビューモデルを持たないのが正当なウィジェットライブラリでは、`-requirevm=false` でこのチェックを無効にできます。

状態フィールドとは、ツールキットのウィジェットが持つもの（`Text`、`Items`、`Rows`、`Root`、`Selected`、`Value`、`Checked` など）です。`-statefields=` でリストを置き換えられます。名前が `*_binding.go` のファイル、または `//mvvmlint:allow` を含むファイルは、1 つ目のルールの対象外です。

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

[`app-template`]({{< relref "/getting-started.md" >}}) には、このジョブが最初から含まれています。
