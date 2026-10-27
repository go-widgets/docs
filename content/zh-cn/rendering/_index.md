---
title: "控件如何到达屏幕"
linkTitle: "渲染"
weight: 10
description: "控件通过绘制器绘制；绘制器决定结果是像素还是终端字符单元；后端负责呈现结果。"
tags: [rendering, painter, toolkit]
---

三个层次，每一层只了解它下面的那一层：

```text
toolkit widget tree          Button, Table, Agenda, VBox ...  (or a skin.Object)
        │  Draw(p painter.Painter, theme *Theme)
        ▼
painter.Painter              FillRect, StrokeRect, Text, PutPixel, ...
        │
        ├─ PixelPainter  → an RGBA []byte   → window, browser canvas, PNG, Android
        └─ CellPainter   → a grid of cells  → a terminal, as 24-bit ANSI
```

控件永远不知道自己拿到的是哪种绘制器。后端永远不知道是哪些控件在绘制。这正是同一棵控件树无需一行条件代码，就能显示在原生窗口、浏览器和终端中的根本原因。

| | |
|---|---|
| [绘制器接缝]({{< relref "/rendering/painter.md" >}}) | 图元、可选能力、两种绘制器 |
| [工具包]({{< relref "/rendering/toolkit.md" >}}) | 控件集、主题、布局、文本和无障碍 |
| [皮肤]({{< relref "/rendering/skin.md" >}}) | 用数据而不是 Go 代码来描述控件的部件和状态 |
