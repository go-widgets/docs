---
title: "bricolint：禁止手绘界面"
linkTitle: "bricolint"
weight: 20
description: "当拉取请求在应用代码中用绘制器图元绘制界面外观，或在每一帧重建控件时，使其失败。"
tags: [ci, lint, painter]
---

*Bricolage*（拼凑）——即临时拼凑、手工绘制——会让界面悄悄丧失按下、悬停和焦点反馈，丧失主题、HiDPI 缩放以及无障碍支持。[`go-widgets/bricolint`](https://github.com/go-widgets/bricolint) 防止已经迁移到工具包上的应用再退回去。

```go
p.FillRect(bar, theme.Bg)              // flagged: hand-drawn chrome
toolkit.NewBackdrop(theme.Bg).Draw(p)  // a toolkit widget

func (v *view) Draw(p painter.Painter) {
	b := toolkit.NewButton("ok")       // flagged: a new widget every frame
	b.Draw(p)
}
```

1. **应用代码中的绘制器图元**：在静态类型为绘制器类型的接收者上调用绘图方法（`FillRect`、`StrokeRect`、`FillPath`、`DrawImage`、`Text`、`PutPixel`，……）。查询或裁剪绘制表面（`Size`、`PushClip`、`PushTranslate`）不算绘制，不会被标记。`-primitives=` 可替换该列表。
2. **每一帧都被丢弃的控件**：在名为 `Draw`、`Paint` 或 `Render` 的方法内调用 `toolkit.New…`。每次绘制都重建的控件不会保留交互状态。应只构建一次，放进字段，并通过绑定来驱动。`-checkthrow=false` 可关闭此项。

有些代码确实是叶子——游戏帧缓冲、绘制器后端、工具包自身的内部实现——并会明确声明这一点：

```go
p.FillRect(bg, c) //bricolint:allow engine SVG raster blit — a genuine leaf
```

```go
//bricolint:allowfile painter back-end — this file IS the leaf
package pdfsurface
```

**理由是必填的**：没有理由的指令会被忽略，检查结果会一直触发，直到有人把理由写下来。

```yaml
jobs:
  bricolint:
    uses: go-widgets/bricolint/.github/workflows/bricolint.yml@main
```
