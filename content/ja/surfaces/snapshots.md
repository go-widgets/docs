---
title: "スナップショット：SVG または PNG としてのウィジェット"
linkTitle: "スナップショット"
weight: 60
description: "go-widgets/svg は、任意の RGBA レンダリングを自己完結した SVG にビット単位で正確に包むか、PNG として書き出します。"
tags: [surfaces, svg, png]
---

[`go-widgets/svg`](https://github.com/go-widgets/svg) は、RGBA バッファを自己完結した SVG に変換します。
`viewBox` はレンダリングのピクセルサイズに固定され、ピクセルは `image-rendering: pixelated` の下で base-64 の
PNG として埋め込まれます。表示されるすべてのピクセルは、ウィジェットが描いたそのままのピクセルです。再ベクター化も
フォントの代替もなく、どんなズームでもくっきり表示されます。

```go
// any RGBA producer
svg.Snapshot(f, surface, w, h, "my image")

// a toolkit widget
btn := toolkit.NewButton("Click me", nil)
btn.SetBounds(toolkit.Rect{W: 200, H: 40})
widget.Snapshot(f, btn, 200, 40, toolkit.DefaultLight(), "widget: button")
widget.PNG(p, btn, 200, 40, toolkit.DefaultLight())
```

ラベルは SVG の `<title>` と `aria-label` になり、エスケープされます。ルートパッケージは何にも依存しません。
`widget` サブパッケージは別になっているので、独自のピクセル生成元を持つ利用者が、ツールキットを自分のモジュール
グラフに引き込むことはありません。

`svg/cmd/gallery-render` は、ウィジェットのカタログをライトとダークでレンダリングします：
[go-widgets.github.io/svg](https://go-widgets.github.io/svg/)。
