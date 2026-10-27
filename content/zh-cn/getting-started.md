---
title: "第一个应用"
weight: 5
description: "从已符合 MVVM 且构建通过的 app-template 开始，或者打开一个空白窗口再添加控件。"
tags: [getting-started, app-template, window]
---

## 从模板开始 {#from-the-template}

[`go-widgets/app-template`](https://github.com/go-widgets/app-template) 是一个模板仓库：一个小巧但真实的浏览器应用，包含搜索框、类别过滤器、匹配行的列表和一条状态栏。更大的应用所需的一切都已接好：一个不含任何控件的视图模型、一个与之绑定的视图、一个 wasm 宿主、100% 覆盖率门禁，以及 [MVVM 门禁]({{< relref "/guard-rails/mvvmlint.md" >}})。

```sh
git clone https://github.com/go-widgets/app-template my-app
cd my-app
go mod edit -module github.com/you/my-app
./build.sh          # dist/app.wasm, dist/wasm_exec.js, dist/index.html
```

请通过 HTTP 提供 `dist/`：从 `file://` 打开的页面无法实例化 wasm。

| 文件 | 内容 |
|---|---|
| `viewmodel.go` | 所有状态，以 [`mvvm`]({{< relref "/state/mvvm.md" >}}) 可观察对象和命令的形式存在。不含控件。无需画布即可测试。 |
| `scene.go` | 各个控件，每个都通过 `mvvmtk` 绑定到视图模型。它自己从不给控件的状态字段赋值。 |
| `main.go` | 唯一带构建标签（`js && wasm`）的文件：把输入转发给工具包，并把画面块传送（blit）到 `<canvas>` 中。 |

编辑前两个文件，并让状态始终经由绑定器流动：CI 中的 `mvvm` 任务会让直接给控件字段赋值的拉取请求失败。

## 从空白窗口开始 {#from-a-bare-window}

```go
package main

import (
	"github.com/go-widgets/toolkit"
	"github.com/go-widgets/window"
)

func main() {
	w, err := window.Open(window.Config{Title: "Demo"})
	if err == window.ErrUnsupported {
		return // no native back-end on this platform
	}
	if err != nil {
		panic(err)
	}
	defer w.Close()

	box := toolkit.NewVBox()
	box.Append(toolkit.NewLabel("Hello"))
	box.Append(toolkit.NewButton("Click me", func() {}))
	w.Run(box) // layout, draw, present, dispatch input, until closed
}
```

宽度和高度保持为零，表示请后端给出一个易读的默认值；在 macOS 上，这是主屏幕的一部分。`CGO_ENABLED=0 go build` 可在任意机器上为每个平台生成二进制文件。

## 该用哪个模块 {#which-module-for-what}

| 你想要 | 从这里开始 |
|---|---|
| 一个桌面窗口 | [`window`]({{< relref "/surfaces/native-window.md" >}}) |
| 一个带托盘图标并跟随系统外观的桌面应用 | [`application`]({{< relref "/surfaces/application.md" >}}) |
| 浏览器中的一个页面 | [`webcanvas`]({{< relref "/surfaces/browser.md" >}})，或上文的模板 |
| 一个终端程序 | [`tui`]({{< relref "/surfaces/terminal.md" >}}) |
| 一个 Android 应用 | [`android`]({{< relref "/surfaces/android.md" >}}) |
| 为 README 准备的控件图片 | [`svg`]({{< relref "/surfaces/snapshots.md" >}}) |

若想在动手写代码之前先看看控件集，请打开[画廊](https://go-widgets.github.io/gallery/)：每个控件系列都在浏览器画布中实时展示。
