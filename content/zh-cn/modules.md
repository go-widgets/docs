---
title: "模块"
weight: 50
description: "go-widgets 的十九个模块，各自的用途，以及源码和 API 参考文档所在的位置。"
tags: [modules]
---

API 参考文档由 [pkg.go.dev](https://pkg.go.dev/) 根据源码生成，并跟随已发布的标签；这些页面负责讲解，不复制参考文档。标签为撰写时的最新标签——以仓库自己的标签列表为准。

| 模块 | 标签 | 简介 | 参考文档 |
|---|---|---|---|
| [`painter`](https://github.com/go-widgets/painter) | v0.16.0 | [绘图接缝]({{< relref "/rendering/painter.md" >}})：像素绘制器与字符单元绘制器 | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/painter) |
| [`toolkit`](https://github.com/go-widgets/toolkit) | v0.345.1 | [控件集]({{< relref "/rendering/toolkit.md" >}})、主题、布局、文本 | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/toolkit) |
| [`skin`](https://github.com/go-widgets/skin) | v0.3.0 | [以数据描述的控件]({{< relref "/rendering/skin.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/skin) |
| [`isoicons`](https://github.com/go-widgets/isoicons) | v0.4.0 | 等距图标包（云原生、AWS），供工具包的等距图使用 | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/isoicons) |
| [`mvvm`](https://github.com/go-widgets/mvvm) | v0.15.0 | [可观察对象、命令、绑定器、撤销]({{< relref "/state/mvvm.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvm) |
| [`mvvmtk`](https://github.com/go-widgets/mvvmtk) | v0.15.0 | [面向工具包控件的一行式绑定器]({{< relref "/state/mvvm.md#mvvmtk-one-call-per-widget" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmtk) |
| [`data`](https://github.com/go-widgets/data) | v0.4.0 | [带类型的记录、查询、本地或远程存储]({{< relref "/state/data.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/data) |
| [`window`](https://github.com/go-widgets/window) | v0.89.1 | 基于八种后端的[原生窗口]({{< relref "/surfaces/native-window.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/window) |
| [`application`](https://github.com/go-widgets/application) | v0.8.1 | 位于窗口之上的[应用生命周期]({{< relref "/surfaces/application.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/application) |
| [`tray`](https://github.com/go-widgets/tray) | v0.15.0 | [托盘图标及其菜单]({{< relref "/surfaces/application.md#the-tray" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tray) |
| [`webcanvas`](https://github.com/go-widgets/webcanvas) | v0.4.1 | [浏览器 `<canvas>` 中的场景]({{< relref "/surfaces/browser.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/webcanvas) |
| [`tui`](https://github.com/go-widgets/tui) | v0.62.0 | [终端渲染与原生字符单元控件]({{< relref "/surfaces/terminal.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tui) |
| [`android`](https://github.com/go-widgets/android) | v0.16.0 | [由无 CGO 进程绘制的 APK]({{< relref "/surfaces/android.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/android) |
| [`svg`](https://github.com/go-widgets/svg) | v0.7.0 | [输出为 SVG 或 PNG 的渲染]({{< relref "/surfaces/snapshots.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/svg) |
| [`mvvmlint`](https://github.com/go-widgets/mvvmlint) | v0.4.2 | [MVVM 检查]({{< relref "/guard-rails/mvvmlint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmlint) |
| [`bricolint`](https://github.com/go-widgets/bricolint) | v0.4.1 | [禁止手绘界面的检查]({{< relref "/guard-rails/bricolint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/bricolint) |
| [`app-template`](https://github.com/go-widgets/app-template) | v0.5.0 | [应用的起点]({{< relref "/getting-started.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/app-template) |
| [`gallery`](https://github.com/go-widgets/gallery) | v0.7.0 | 每个控件系列的[在线演示](https://go-widgets.github.io/gallery/) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/gallery) |
| [`desktop`](https://github.com/go-widgets/desktop) | v0.20.0 | 由 go-freedesktop 与 go-widgets 组成的桌面 shell，可原生运行，也可在 wasmdesk 中运行 | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/desktop) |

## 每个模块都必须遵守的约束 {#what-every-module-is-held-to}

- `CGO_ENABLED=0`：不使用 cgo，也不通过调用命令行工具来代替库。
- 交叉编译到 amd64、arm64、riscv64、loong64、ppc64le 和 s390x——最后一个是大端架构，可确保所有线上编码都经得起检验——并在适用时编译到 `js/wasm`。自 2026-10-10 起，全部十七个库和工具都做到了；另外两个是浏览器应用（[状态]({{< relref "/status.md#what-ci-does-not-cover-yet" >}})）。
- CI 中的语句覆盖率门禁：十九个模块全部为 100%。若某个文件无法在测试中运行（浏览器循环、原生运行循环），该模块会说明排除了哪个文件以及原因。例外情况见[状态]({{< relref "/status.md" >}})。
- BSD-3-Clause。
