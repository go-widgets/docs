---
title: "绘制器接缝"
linkTitle: "绘制器"
weight: 10
description: "所有控件赖以绘制的图元、绘制器可以额外提供的可选能力，以及像素绘制器和字符单元绘制器。"
tags: [rendering, painter]
---

[`go-widgets/painter`](https://github.com/go-widgets/painter) 定义了所有控件都通过它来绘制的唯一接口：

```go
type Painter interface {
	FillRect(r Rect, c RGBA)
	StrokeRect(r Rect, c RGBA, lineW int)
	FillRoundRect(r Rect, radius int, c RGBA)
	StrokeRoundRect(r Rect, radius int, c RGBA, lineW int)
	PutPixel(x, y int, c RGBA)
	Text(x, y int, s string, ink RGBA)
	Size() (w, h int)
}
```

这个集合刻意保持小巧：每个后端都必须全部实现，而终端字符网格无法画出圆角，也无法改变描边宽度。因此契约规定了无法实现时的行为——`StrokeRect` 的宽度只是一个提示，字符网格会忽略它，而 `FillRoundRect` 在那里退化为方形填充。

## 可选能力 {#optional-capabilities}

并非每个绘制表面都能做到的事，不在基础接口里。需要它的控件通过类型断言来判断，若答案为否，则绘制出合理的替代效果：

| 接口 | 方法 | 用途 |
|---|---|---|
| `Clipper` | `PushClip`, `PopClip` | 让滚动的子控件保持在其父控件的边界之内 |
| `Translator` | `PushTranslate`, `PopTranslate` | 在子控件自己的坐标系中绘制它 |
| `PathPainter` | `FillPath`, `StrokePath` | 曲线、图表、抗锯齿图形 |
| `ImagePainter` | `DrawImage` | 图标、缩略图、图片 |
| `MaskPainter` | `DrawMask` | 单一颜色的字形及其他覆盖率蒙版 |
| `FacePainter` | `TextFace` | 使用指定字体字面和字号的文本 |

```go
if c, ok := p.(painter.Clipper); ok {
	c.PushClip(bounds)
	defer c.PopClip()
}
```

## 两种绘制器 {#the-two-painters}

| | 写入 | 由谁呈现 |
|---|---|---|
| `NewPixelPainter(buf, w, h)` | 调用方拥有的 RGBA `[]byte`（BGRA 绘制表面用 `NewPixelPainterBGRA`） | 原生窗口、浏览器画布、Android 绘制表面、PNG 或 SVG |
| `NewCellPainter(w, h)` | 由字符单元（字符、前景色、背景色）组成的网格，带 24 位 ANSI 序列化器 | 终端，经由 [`tui`]({{< relref "/surfaces/terminal.md" >}}) |

缓冲区始终归调用方所有。`PixelPainter` 只是把图元翻译成写入操作，所以同一个 `[]byte` 可以直接交给窗口后端、`<canvas>` 或图像编码器，中间无需拷贝。

## 录制为 SVG {#recording-to-svg}

`svgrec.Recorder`（包 `github.com/go-widgets/painter/svgrec`）是第三种绘制器：它把同样的调用转换成**原生 SVG 文档**，由 `<rect>`、`<path>` 和 `<clipPath>` 元素组成，只有在确实绘制了光栅图的地方才使用 `<image>`。结果无损、可缩放，并且是确定性的：相同的调用得到相同的字节。

```go
r, _ := svgrec.New(1000, 700)
widget.Draw(r, painter.LightTheme())
doc, err := r.SVG() // err is the first call that was refused, if any
```

它实现了 `Clipper`、`Translator`、`PathPainter`、`ImagePainter` 和 `MaskPainter`。在 SVG 没有忠实对应物的地方，映射表会明说，而不是悄悄地近似：

| 调用 | SVG |
|---|---|
| `FillRect`、`PutPixel` | `<rect>` |
| `FillRoundRect` | `<rect rx>`；圆角的抗锯齿方式与像素绘制器不同 |
| `StrokeRect`、`StrokeRoundRect` | 一个 `evenodd` 边框，环向内绘制，与像素绘制器一致 |
| `Text` | 字形轮廓（见下文） |
| `PushClip` / `PushTranslate` | `<clipPath>` 加一个嵌套的 `<g>`；平移直接作用于坐标 |
| `FillPath`、`StrokePath` | `<path>`；描边宽度以用户单位计，因此随 viewBox 缩放；长度为零的子路径在 SVG 中会画出一个点，而像素绘制器什么也不画 |
| `DrawImage`、`DrawMask` | 带 PNG 数据 URI 的 `<image>` |

**文本变成轮廓，而不是 `<text>`。** 内置的 5x7 字体是像素网格，因此每个字符串是一个由合并后的连续段组成的 `<path>`，外面包一个 `<g>`，其 `<title>` 为辅助技术保存文本。嵌入子集字体需要字体写入器和遵循它的查看器；不遵循的查看器会回退到另一种字体，使版式错位。代价是：文本无法选中。用 TrueType 字体排版的文本目前还不会记录为轮廓；`FacePainter` 未提供，因此它以光栅 `DrawMask` 的形式出现。

绘制器的方法不返回错误，因此被拒绝的调用（无效的 UTF-8、XML 无法承载的字符、超过 1<<24 的坐标、超出其 `Limits` 的调用或文档）会被丢弃，第一个错误由 `Err()` 和 `SVG()` 报告。模糊测试目标会用 `encoding/xml` 把每个输出重新解析，只要出现写入器自己不产生的元素或属性就判失败。

与像素绘制器相比（用纯 Go 的 `go-gfx/gfx/svg` 光栅化），示例控件、矩形、位图文本、裁剪和平移与之逐像素一致；路径在单个通道上最多相差 1；抗锯齿圆角在最差场景中，对 7.5 % 的像素，单个通道最多相差 61。

## 试一试 {#trying-it}

仓库自带的演示通过两种绘制器绘制同样的三个控件：

```sh
go run ./cmd/wui-demo --out demo.png     # pixels, written as a PNG
go run ./cmd/tui-demo --theme dark       # cells, written as ANSI to stdout
```

此外，[go-widgets.github.io/painter](https://go-widgets.github.io/painter/) 在浏览器中展示了像素绘制器。
