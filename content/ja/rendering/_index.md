---
title: "ウィジェットが画面に届くまで"
linkTitle: "レンダリング"
weight: 10
description: "ウィジェットはペインタ（painter）を通して描画します。ピクセルになるかターミナルのセルになるかはペインタが決め、バックエンドが結果を表示します。"
tags: [rendering, painter, toolkit]
---

3 つの層があり、それぞれが知っているのは 1 つ下の層だけです。

```text
toolkit widget tree          Button, Table, Agenda, VBox ...  (or a skin.Object)
        │  Draw(p painter.Painter, theme *Theme)
        ▼
painter.Painter              FillRect, StrokeRect, Text, PutPixel, ...
        │
        ├─ PixelPainter  → an RGBA []byte   → window, browser canvas, PNG, Android
        └─ CellPainter   → a grid of cells  → a terminal, as 24-bit ANSI
```

ウィジェットは、どのペインタを渡されたかを知りません。バックエンドは、どのウィジェットが描画したかを知りません。1 つのウィジェットツリーを、条件分岐のコードを 1 行も書かずに、ネイティブウィンドウ、ブラウザ、ターミナルで表示できるのは、まさにこのためです。

| | |
|---|---|
| [ペインタの継ぎ目]({{< relref "/rendering/painter.md" >}}) | プリミティブ、オプションの機能、2 種類のペインタ |
| [ツールキット]({{< relref "/rendering/toolkit.md" >}}) | ウィジェットセット、テーマ、レイアウト、テキスト、アクセシビリティ |
| [スキン]({{< relref "/rendering/skin.md" >}}) | ウィジェットのパーツと状態を、Go ではなくデータで記述 |
