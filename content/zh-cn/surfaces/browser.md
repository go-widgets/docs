---
title: "浏览器标签页"
linkTitle: "浏览器"
weight: 30
description: "用 go-widgets/webcanvas 在普通的 <canvas> 中运行控件树，或作为 wasmdesk 合成器的一个窗口运行。"
tags: [surfaces, wasm, browser, webcanvas, wasmdesk]
---

工具包是纯 Go，不依赖 DOM，因此 `GOOS=js GOARCH=wasm` 像构建其他目标一样构建它。
把结果呈现给别人有两种方式：

## 一个普通的 canvas {#a-plain-canvas}

[`go-widgets/webcanvas`](https://github.com/go-widgets/webcanvas) 把 RGBA 帧缓冲区绘制到 `<canvas>` 中，
并把 DOM 的指针和键盘事件路由回场景。不需要合成器，不需要 `SharedArrayBuffer`，也不需要跨源隔离响应头：
任何静态主机都能提供它。

```go
//go:build js && wasm

package main

import "github.com/go-widgets/webcanvas"

func main() { webcanvas.Run("screen", myScene()) }
```

场景实现 `App`：`Size`、`Draw(buf []byte)`，以及针对每一类事件的一个方法，每个方法都报告是否有任何变化，
这样未变化的帧就不会被重绘。它还可以实现 `Ticker`、`Animator`、`Resizer` 或 `Scroller`。

[画廊（gallery）](https://go-widgets.github.io/gallery/)和
[`app-template`]({{< relref "/getting-started.md" >}}) 就是这样运行的。

或者让 `window` 来选择。自 window v0.87.0 起，为 `js/wasm` 构建、运行在普通页面上的
[`window.Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}})
会返回一个后端，它通过 `webcanvas` 绘制到页面的 `<canvas>`（`Config.Canvas`，默认为 `"screen"`）。
这样，针对 `window` 编写的应用无需改动就能作为原生窗口、在浏览器标签页中以及在 wasmdesk 中运行。

## wasmdesk 桌面上的一个窗口 {#a-window-on-the-wasmdesk-desktop}

在 wasmdesk 工作线程内，[`window.Open`]({{< relref "/surfaces/native-window.md" >}})
返回 [wasmdesk/wasmbox](https://github.com/wasmdesk/wasmbox) 浏览器合成器的一个客户端。
它在 `SharedArrayBuffer` 中分配自己的绘制表面，通过 `MessagePort` 发出 `hello`，等待 `welcome`，
绘制到共享缓冲区中，然后发送 `commit`——整个绘制表面，或只是受损的矩形。合成器的 `input` 消息会变成
`toolkit.Event`，与 X11 上相同。

因此，针对 `window` 编写的应用无需改动，既可作为原生窗口运行，也可作为浏览器标签页中桌面的一个窗口运行。
[桌面外壳演示](https://github.com/go-widgets/desktop)正是如此，而且出自同一份源码。

协议编解码器在每个平台上的覆盖率都是 100%。实时验证在无界面的 Chromium 中针对真实的 wasmdesk 合成器运行：
它启动客户端，读取合成器自己合成出的像素，进行点击，并检查该点击是否到达了控件。
