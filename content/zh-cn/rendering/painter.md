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

## 试一试 {#trying-it}

仓库自带的演示通过两种绘制器绘制同样的三个控件：

```sh
go run ./cmd/wui-demo --out demo.png     # pixels, written as a PNG
go run ./cmd/tui-demo --theme dark       # cells, written as ANSI to stdout
```

此外，[go-widgets.github.io/painter](https://go-widgets.github.io/painter/) 在浏览器中展示了像素绘制器。
