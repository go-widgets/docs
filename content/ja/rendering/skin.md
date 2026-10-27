---
title: "スキン: ウィジェットのパーツと状態をデータで記述"
linkTitle: "スキン"
weight: 30
description: "go-widgets/skin は、ウィジェットのパーツ、状態ごとの見た目、シグナル駆動の遷移を .skin.json ファイルで記述し、1 つのエンジンで描画します。"
tags: [rendering, skin, theme]
---

ツールキットのウィジェットは Go で書かれています。`Button` のパーツ、ホバー、押下、無効の各表示、クリック時のフィードバックは、すべて `button.go` にあります。高速で正確ですが、見た目を変えるたびにコードの変更が必要です。

[`go-widgets/skin`](https://github.com/go-widgets/skin) は、その記述をデータに移します。着想は Enlightenment の Edje です。`.skin.json` ドキュメントには、ウィジェットの **パーツ**、各 **状態** での見た目、シグナルが届いたときにパーツを状態間で動かす **プログラム** を記述します。これにより、スキンの張り替えや新しいコントロールの作成に再コンパイルは不要で、プログラムが別のバイト列を読み込むだけで済みます。

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

`skin.Object` は `toolkit.Widget` なので、手書きのウィジェットと並べて `VBox` に入れられます。レイアウトはウィジェットを配置し、スキンはウィジェットの内部を構成します。

## フォーマット {#the-format}

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

- **パーツ** は `rect`、`text`、`image` のいずれかで、オブジェクトまたは先行するパーツに対する 2 つの相対的な角（`rel1`、`rel2`）によって配置します。オフセットにはピクセル指定と、フォント相対の指定（`offset_em`）があり、テキストのパーツはフォントに追従します。
- **状態** では、色、枠線、角丸、可視性を設定でき、パーツを移動したりリサイズしたりもできます。
- **プログラム** はシグナルに反応し、名前付きのイージングで、指定した時間をかけてパーツを状態へ移動させます。また、シグナルを外部へ `emit` することもでき、たとえば [`mvvm.Command`]({{< relref "/state/mvvm.md" >}}) に送れます。
- 色は、ツールキットの `Theme` に対して解決される `@tokens`（GTK の `@define-color` エントリもすべて `@extra:<name>` として利用できます）、16 進数、または RGBA の配列で指定します。

JSON は標準ライブラリでデコードされるため、サードパーティへの依存はありません。

## 手書きのウィジェットとの照合による検証 {#proven-against-the-hand-written-widgets}

ツールキットのボタン、チェックボックス、スイッチ、チップ、カードは、それぞれスキンのコレクションとして書き直されており、テストでは、あらゆる状態で両者の描画が **ピクセル単位で完全に一致する** ことを検証しています。スキンによる `Draw` は、手書きの `Draw` と比べた速度をベンチマークで計測しています。
