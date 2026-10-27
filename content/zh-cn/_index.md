---
title: "go-widgets"
linkTitle: "首页"
type: docs
cascade:
  type: docs
description: "纯 Go 实现的控件（widget）工具包，以及它能绘制的所有绘制表面：X11、Wayland、macOS 和 Windows 上的原生窗口，浏览器画布，终端，Android APK。"
---

**一棵控件树，由 Go 绘制，覆盖 Go 能到达的每一个绘制表面（surface）。** X11、Wayland、macOS 和 Windows 上的原生窗口；浏览器标签页中的 `<canvas>`；终端；Android APK。无需 C 工具链，无需 webview，也不依赖系统控件集：每个模块都以 `CGO_ENABLED=0` 构建，每一个像素都由工具包自己绘制。

```go
w, err := window.Open(window.Config{Title: "Hello"})
if err != nil {
	return err
}
defer w.Close()

box := toolkit.NewVBox()
box.Append(toolkit.NewLabel("Hello from go-widgets"))
box.Append(toolkit.NewButton("Click me", func() { /* ... */ }))
return w.Run(box) // an X11 or Wayland window, an NSWindow, a Win32 window
```

以上是
[`cmd/windowdemo`](https://github.com/go-widgets/window/blob/main/cmd/windowdemo/main.go)
的精简版。同一个 `box` 画到浏览器、终端或 PNG 中，代码完全相同，只是底下的绘制表面不同。

## 为什么由一个工具包绘制一切 {#why-one-toolkit-draws-everything}

调用平台自带按钮的控件，其可移植性只能与它所支持的最弱平台相当，而且在每个平台上外观各异。go-widgets 走的是另一条路：控件的 `Draw` 只与 [`painter.Painter`]({{< relref "/rendering/painter.md" >}}) 对话，它只提供少量图元（填充矩形、描边矩形、设置像素、绘制文本）。这些调用最终变成 RGBA 像素还是终端字符单元，是后端的事，与控件无关。

代价也直说：默认没有原生外观，也没有现成的无障碍支持——工具包自行向各平台发布它的[无障碍树]({{< relref "/rendering/toolkit.md#accessibility" >}})。换来的是一套只需测试一次、在任何地方行为都一致的控件集，以及一个可以在任意机器上交叉编译到所有这些平台的二进制文件。

## 接下来去哪里 {#where-to-go-next}

| | |
|---|---|
| [第一个应用]({{< relref "/getting-started.md" >}}) | 从 `app-template` 开始，或从一个空白窗口开始 |
| [控件如何到达屏幕]({{< relref "/rendering/_index.md" >}}) | 绘制器接缝、控件集、皮肤 |
| [绘制器接缝]({{< relref "/rendering/painter.md" >}}) | 所有控件赖以绘制的图元 |
| [工具包]({{< relref "/rendering/toolkit.md" >}}) | 160 种控件类型、主题、布局、文本、无障碍 |
| [皮肤]({{< relref "/rendering/skin.md" >}}) | 把控件的部件和状态写进数据文件 |
| [状态存放在视图模型中]({{< relref "/state/_index.md" >}}) | 为什么不手工给控件字段赋值 |
| [MVVM]({{< relref "/state/mvvm.md" >}}) | 可观察对象、命令、绑定器、撤销 |
| [数据主干]({{< relref "/state/data.md" >}}) | 带类型的记录、查询、本地或远程存储 |
| [绘制到哪里]({{< relref "/surfaces/_index.md" >}}) | 所有后端及其选择方式 |
| [原生窗口]({{< relref "/surfaces/native-window.md" >}}) | X11、Wayland、Cocoa、Win32、GTK4、Android、wasmbox、浏览器标签页 |
| [应用及其托盘]({{< relref "/surfaces/application.md" >}}) | 生命周期、外观、菜单栏图标 |
| [浏览器标签页]({{< relref "/surfaces/browser.md" >}}) | 普通的 `<canvas>`，或 wasmdesk 窗口 |
| [终端]({{< relref "/surfaces/terminal.md" >}}) | 原生字符单元控件和交互式运行器 |
| [Android]({{< relref "/surfaces/android.md" >}}) | 由无 CGO 进程绘制的真实 APK |
| [快照]({{< relref "/surfaces/snapshots.md" >}}) | 把控件输出为 SVG 或 PNG |
| [CI 强制执行的规则]({{< relref "/guard-rails/_index.md" >}}) | `mvvmlint` 和 `bricolint` |
| [模块]({{< relref "/modules.md" >}}) | 全部十九个模块，附源码和参考文档 |
| [状态]({{< relref "/status.md" >}}) | 哪些已在哪里验证过，哪些尚未完成 |

## 许可证 {#licence}

BSD-3-Clause。
