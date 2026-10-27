---
title: "mvvmlint：状态经由视图模型"
linkTitle: "mvvmlint"
weight: 10
description: "当拉取请求直接给工具包控件的状态字段赋值，或者有控件却没有视图模型时，使其失败。"
tags: [ci, lint, mvvm]
---

[`go-widgets/mvvmlint`](https://github.com/go-widgets/mvvmlint) 只在导入了工具包的包中触发。

1. **手工给控件的状态字段赋值。** `entry.Text = "x"`，其中 `entry` 的静态类型是工具包控件，`Text` 是状态字段，会被标记。`&entry.Text`——[绑定器]({{< relref "/state/mvvm.md" >}})接受的形式——不是赋值，绝不会被标记；复合字面量（`toolkit.Entry{Text: "x"}`）以及组装 `OnChange` 之类的回调也不会。
2. **有控件却没有视图模型。** 导入了工具包却没有导入 `go-widgets/mvvm` 的包，会得到一条诊断。对于确实没有视图模型的控件库，`-requirevm=false` 可关闭此项。

状态字段指的是工具包各控件的那些字段（`Text`、`Items`、`Rows`、`Root`、`Selected`、`Value`、`Checked`，……）；`-statefields=` 可替换该列表。名为 `*_binding.go` 的文件，或包含 `//mvvmlint:allow` 的文件，不受第一条规则约束。

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

[`app-template`]({{< relref "/getting-started.md" >}}) 自带这个任务。
