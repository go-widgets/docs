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

## 动画 SVG

`svg.Animate` 把一组带时间的 SVG 帧（例如来自 painter 的
[`svgrec`](../../rendering/painter/) 录制器）拼成一个动画 SVG。帧与帧之间没有
变化的子树只在 `<defs>` 中保存一次，并用 `<use>` 复用；相邻的相同帧会合并，
时序是纯 CSS（`@keyframes`，没有脚本），可被 `prefers-reduced-motion` 停止。

```go
out, err := svg.Animate([]svg.Frame{
    {At: 0, SVG: first},
    {At: 100 * time.Millisecond, SVG: second},
}, svg.Options{End: time.Second, Title: "demo"})
```

对 60 帧 1000×700 的画面，只有一个区域在变化的屏幕从 1.49 MB 缩小到 65 KB
（缩小 22.9 倍），静态屏幕缩小到 24 KB（60 倍）。输入用允许列表解析并有上限
（帧数、字节数、时长、尺寸），每次拒绝都是带类型的错误。`Document.Video` 驱动由您
通过两个小接口提供的光栅化器和视频编码器；不附带 WebM 编码器。
