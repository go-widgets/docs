---
title: "终端"
linkTitle: "终端"
weight: 40
description: "go-widgets/tui 通过单元格绘制器渲染控件：一个快照渲染器、一个交互式运行器，以及为字符网格设计尺寸的控件。"
tags: [surfaces, tui, terminal]
---

[`go-widgets/tui`](https://github.com/go-widgets/tui) 通过
[单元格绘制器]({{< relref "/rendering/painter.md" >}})把控件树变成终端帧。
仅使用标准库——没有 `x/term`，没有 cgo。

## 一帧 {#one-frame}

```go
_ = tui.RenderOnce(os.Stdout, widgets, nil) // nil theme: light
```

`RenderOnceSized(w, cols, rows, widgets, theme)` 显式指定大小；`RenderOnce` 读取 `COLUMNS` 和 `LINES`，
并回退到 80×24。读取环境变量而不是调用 `TIOCGWINSZ`，可以让该包不含任何平台相关代码。

## 交互式程序 {#an-interactive-program}

```go
app := tui.NewApp()
app.Root = buildScene()
app.Keys["q"] = func(a *tui.App) { a.Quit() }
app.Keys["Ctrl+C"] = func(a *tui.App) { a.Quit() }
os.Exit(app.Run())
```

`Run` 进入备用屏幕和原始模式，把输入分发给 `Root`（全局按键处理函数优先），跟随 `SIGWINCH`，
并且总会恢复终端——发生 panic 时也一样。按键处理函数调用 `Consume` 可以不让某个事件到达 `Root`；
`InputTarget` 把每个未被消费的事件送给某一个控件，命令面板打开期间吞掉键入内容就是这样实现的。
`NewFocusRing` 让表单支持 Tab 和 Shift+Tab 遍历。

## 为单元格设计尺寸的控件 {#widgets-sized-for-cells}

大多数工具包控件是按像素设计的：它们的内边距常量在单元格绘制器之下按单元格计数，因而会膨胀。
所以 `tui` 有自己**原生面向单元格**的一组控件——`TextEditor`、`Entry`、`Button`、`CheckButton`、`RadioButton`、
`Scale`、`ProgressBar`、`Sparkline`、`ListBox`、`TreeView`、`Table`、`MenuBar`、`Notebook`、`HSplit`、`VSplit`、
`Dialog`、`Dropdown`、`Spinner` 等——其中一个字形就是一个单元格。每个控件仍然是 `toolkit.Widget`，
因此同一个实例也能绘制成像素。

`TextEditor` 是可读写的代码编辑器：行号栏、撤销和重做、查找和替换、选择、行移动，以及对 Go（通过 `go/scanner`）、
JavaScript 和 TypeScript、Python、Ruby、shell、C 和 C++、Rust、JSON、YAML、HCL、TOML、LaTeX 和 Markdown 的语法高亮，
且没有外部依赖。

```sh
go run ./cmd/tui-widgets | less -R     # every cell-native widget, one per slot
go run ./cmd/tui-widget-explorer       # the same, interactive
go run ./cmd/tui-explorer              # a file browser
go run ./cmd/tui-editor --file=x.go    # a modal editor
```

这两个演示程序在真实的 pty 中做了端到端测试：输入真实的按键字节，并对渲染出的帧做断言。
