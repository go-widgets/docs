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

## 只重绘变化的部分 {#repainting-only-what-changed}

如果不加处理，每一帧都是整个场景的 `Draw`、整个帧缓冲向浏览器的复制，以及整个 canvas 的 `putImageData`，
即使只是光标闪烁了一下，或只是 `Layer` 中客户机的图像更新了。知道自己改变了什么的场景实现
`DrawDamaged(buf []byte) []Rect`（`webcanvas.DamageDrawer`，webcanvas v0.15.0），宿主就只复制并放置这些矩形。

- **约定**与 [`window`]({{< relref "/surfaces/native-window.md" >}}) 的 `DamageRenderer` 给桌面宿主的约定相同，
  只是换成浏览器的说法：场景把变化的部分重绘到帧与帧之间保持不变的帧缓冲中，并返回重绘的矩形
  （`webcanvas.Rect` 就是 `toolkit.Rect`）。结果为**空**表示没有任何变化，也不放置任何内容。第一帧以及每次调整大小后，
  仍会执行 `Draw`。
- **图层**每一帧都会重新绘制，但不需要重绘场景。图层移动、缩小或消失时，它原来的矩形会从场景缓冲中恢复；
  在 `Alpha < 1` 或未标记 `Opaque` 的图层之下，每一帧都会恢复场景。`Opaque: true` 承诺所有像素都不透明，
  这样静止的客户机画面就不会放置任何内容。
- **不使用它**则一切照旧：没有 `DrawDamaged` 的场景像以前一样被整体绘制并放置。`webcanvas/gpu` 呈现器有它自己的版本，见下文。
- **测量**：在 node 中以 `GOOS=js`、1280×800 和一个伪造的 2D 上下文测得，改变一个 120×40 部件的帧耗时 0.031 ms，
  而不是 2.89 ms；没有变化且带一个不透明图层的帧耗时 0.021 ms，而不是 2.90 ms。

## 在 GPU 上绘制 {#on-the-gpu}

`webcanvas.RunWith(id, app, presenter)` 接收一个 `Presenter`,`webcanvas/gpu`
包提供了一个用 WebGPU 绘制帧的实现。
- **方式:** 同时实现了 `Paint(painter.Painter)` 的 App,经
  [gpupainter](https://github.com/go-widgets/gpupainter) 记录为
  [go-gfx/gpu2d](https://github.com/go-gfx/gpu2d) 列表。它的 WebGPU 渲染器绘制该帧,
  并在 GPU 上把帧复制到画布纹理中。
- **为何用 Presenter:** 一个画布终生只能取一种上下文,因此由 Presenter 在第一帧之前一次性决定。
- **精确性:** 像素与 `Draw` 的 CPU 帧逐字节相同(有测试保证)。
- **损坏区域:** 同时实现了 `PaintDamaged(painter.Painter) []Rect` 的 App
  (`gpu.DamagePainted`,webcanvas v0.16.0 和 v0.17.0),像 toolkit 的 `RenderDamaged` 一样,
  只把变化的部分绘制到 painter 中,并限制在它返回的矩形内。结果为**空**时,该帧既不记录也不呈现:
  静止的帧不会向 GPU 发送命令缓冲区(之前 2 个,之后 0 个,在 Chrome 中计数,0 次分配)。
  有损坏区域的帧按 `PaintDamaged` 绘制的内容记录,因此场景必须重绘损坏区域之下的底色:
  1,200 个单元格中有一个受损时,交给渲染器的列表从 2,401 个实例降为 2 个。
  第一帧、调整大小,以及没有 `PaintDamaged` 的 App 都会被完整记录。
- **画布纹理不会保留:** 每次呈现之后,WebGPU 规范会给 `getCurrentTexture()` 一个透明黑色纹理
  (Chrome 也一样),因此渲染器自己的目标保留该帧的其余部分,每个呈现的帧仍会把全部内容在 GPU 上复制到画布。
- **回退:** 没有 WebGPU 的浏览器,或没有 `Paint` 的 App,仍使用 2D 画布。两种情况都不算错误。

[Web 画廊](https://go-widgets.github.io/gallery/)就是这样绘制的。

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
