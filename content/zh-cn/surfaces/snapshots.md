---
title: "快照：把控件导出为 SVG 或 PNG"
linkTitle: "快照"
weight: 60
description: "go-widgets/svg 把任何 RGBA 渲染结果无损地封装进自包含的 SVG，或将其写成 PNG。"
tags: [surfaces, svg, png]
---

[`go-widgets/svg`](https://github.com/go-widgets/svg) 把 RGBA 缓冲区变成自包含的 SVG：
`viewBox` 固定为渲染结果的像素大小，像素则以 base-64 PNG 的形式放在 `image-rendering: pixelated` 之下。
所显示的每个像素就是控件绘制的那个像素——不重新矢量化，不替换字体——并且在任何缩放比例下都保持清晰。

```go
// any RGBA producer
svg.Snapshot(f, surface, w, h, "my image")

// a toolkit widget
btn := toolkit.NewButton("Click me", nil)
btn.SetBounds(toolkit.Rect{W: 200, H: 40})
widget.Snapshot(f, btn, 200, 40, toolkit.DefaultLight(), "widget: button")
widget.PNG(p, btn, 200, 40, toolkit.DefaultLight())
```

标签会成为 SVG 的 `<title>` 和 `aria-label`，并经过转义。根包不依赖任何东西；`widget` 子包是独立的，
这样拥有自己像素生产者的使用方就不会把工具包拉进它的模块图。

`svg/cmd/gallery-render` 以浅色和深色渲染控件目录：
[go-widgets.github.io/svg](https://go-widgets.github.io/svg/)。
