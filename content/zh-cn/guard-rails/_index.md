---
title: "CI 强制执行的规则"
linkTitle: "CI 规则"
weight: 40
description: "应用作为必需状态检查运行的两个 go/analysis 检查：状态经由 MVVM，以及禁止手绘界面。"
tags: [ci, lint]
---

有两项约定让应用的界面保持规范，如果只是写在文档里，它们会在一次次图省事的捷径中逐渐瓦解。因此每一项都做成了 [`go/analysis`](https://pkg.go.dev/golang.org/x/tools/go/analysis) 分析器，通过 `go vet` 运行，并配有可复用的工作流，应用可将其设为**必需**检查。

| | 拒绝的内容 |
|---|---|
| [`mvvmlint`]({{< relref "/guard-rails/mvvmlint.md" >}}) | 手工给控件的状态字段赋值，以及包含控件却没有视图模型的包 |
| [`bricolint`]({{< relref "/guard-rails/bricolint.md" >}}) | 应用代码中的绘制器图元，以及每一帧都重新构建的控件 |

两者在设计上都很保守：只有当接收者的**静态类型**经由 `go/types` 解析为工具包或绘制器类型时，规则才会触发。无关类型上同名的字段或方法绝不会被标记。有意的例外是代码中的显式指令，因此每一处例外都写在它出现的地方。
