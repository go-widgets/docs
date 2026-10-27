---
title: "皮肤：把控件的部件和状态写成数据"
linkTitle: "皮肤"
weight: 30
description: "go-widgets/skin 在 .skin.json 文件中描述控件的部件、各状态下的外观以及由信号驱动的过渡，并由同一个引擎渲染。"
tags: [rendering, skin, theme]
---

工具包的控件是用 Go 编写的：`Button` 的各个部件、悬停、按下和禁用时的外观，以及点击反馈，全都写在 `button.go` 里。这样做快速而精确，但每一次视觉上的改动都是一次代码改动。

[`go-widgets/skin`](https://github.com/go-widgets/skin) 把这份描述搬进了数据——这一思路来自 Enlightenment 的 Edje。一份 `.skin.json` 文档会列出控件的**部件**（part）、它们在每种**状态**（state）下的外观，以及当信号到来时让它们在状态之间切换的**程序**（program）。这样，换肤或构建新的控件就无需重新编译：程序只是加载了不同的字节。

```go
theme, err := skin.Load(jsonBytes) // parse and validate every collection
if err != nil {
	return err
}
obj, _ := theme.New("button")
obj.Bind(skin.NewMVVMContext().Set("$.label", labelObs))
obj.SetBounds(toolkit.Rect{W: 96, H: 28})

// each frame
obj.OnEvent(ev)               // pointer events become signals, signals states
busy := obj.Tick(time.Now())  // advance the transitions
obj.Draw(p, theme.Palette())
```

`skin.Object` 就是一个 `toolkit.Widget`，因此它可以与手写控件并排放进同一个 `VBox`。布局负责安放控件；皮肤负责安排控件内部的内容。

## 格式 {#the-format}

```jsonc
{
  "collections": {
    "button": {
      "min": { "w": 96, "h": 28 },
      "parts": [
        { "name": "bg", "type": "rect",
          "states": {
            "default": { "color": "@surface",     "border": "@border", "radius": 6 },
            "hover":   { "color": "@surface_alt", "border": "@border", "radius": 6 },
            "pressed": { "color": "@accent",      "border": "@border", "radius": 6 } } },
        { "name": "label", "type": "text", "text_from": "$.label", "align": [0.5, 0.5],
          "states": { "default": { "ink": "@on_surface" } } }
      ],
      "programs": [
        { "on": "mouse,in",    "target": ["bg", "label"], "to": "hover", "in": 0.12, "ease": "ease_out_cubic" },
        { "on": "mouse,click", "emit": "clicked" }
      ]
    }
  }
}
```

- **部件**可以是 `rect`、`text` 或 `image`，由两个相对角点（`rel1`、`rel2`）相对于对象或前面的某个部件定位，偏移量可以用像素，也可以用相对字体的（`offset_em`），因此文本部件会跟随字体。
- **状态**设定颜色、边框、圆角和可见性，也可以移动或缩放部件。
- **程序**对信号作出反应，按指定的缓动函数在一段时间内把部件移到某个状态，并且可以向外 `emit` 一个信号——例如发给 [`mvvm.Command`]({{< relref "/state/mvvm.md" >}})。
- 颜色可以是针对工具包 `Theme` 解析的 `@tokens`（包括每个 GTK `@define-color` 条目，写作 `@extra:<name>`）、十六进制值，或 RGBA 数组。

JSON 由标准库解码：没有第三方依赖。

## 与手写控件的对照验证 {#proven-against-the-hand-written-widgets}

工具包的按钮、复选框、开关、芯片和卡片，各自又被重新写成了皮肤集合，测试断言两者在每一种状态下渲染出的结果**逐像素完全一致**。皮肤版 `Draw` 与手写版的对比，则在基准测试中进行测量。
