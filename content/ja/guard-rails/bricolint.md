---
title: "bricolint: 手描き UI の禁止"
linkTitle: "bricolint"
weight: 20
description: "アプリケーションコード内でペインタのプリミティブを使って UI の外枠を描画したり、毎フレームウィジェットを作り直したりするプルリクエストを失敗させます。"
tags: [ci, lint, painter]
---

*Bricolage*（ブリコラージュ）、つまり即興の手作り描画は、インターフェースから押下、ホバー、フォーカスのフィードバック、テーマ、HiDPI のスケーリング、アクセシビリティを、いつの間にか失わせるやり方です。
[`go-widgets/bricolint`](https://github.com/go-widgets/bricolint)
は、ツールキットに移行済みのアプリケーションが元の状態に戻ってしまうのを防ぎます。

```go
p.FillRect(bar, theme.Bg)              // flagged: hand-drawn chrome
toolkit.NewBackdrop(theme.Bg).Draw(p)  // a toolkit widget

func (v *view) Draw(p painter.Painter) {
	b := toolkit.NewButton("ok")       // flagged: a new widget every frame
	b.Draw(p)
}
```

1. **アプリケーションコード内のペインタプリミティブ**: 静的な型がペインタ型であるレシーバに対する描画メソッド（`FillRect`、`StrokeRect`、`FillPath`、`DrawImage`、`Text`、`PutPixel` など）の呼び出しです。サーフェスの照会やクリップ（`Size`、`PushClip`、`PushTranslate`）は描画ではないため、指摘されません。`-primitives=` でリストを置き換えられます。
2. **毎フレーム捨てられるウィジェット**: `Draw`、`Paint`、`Render` という名前のメソッド内での `toolkit.New…` の呼び出しです。描画のたびに作り直されるウィジェットは、操作の状態を保持できません。1 度だけ作成してフィールドに保持し、バインディングで駆動してください。`-checkthrow=false` でこのチェックを無効にできます。

ゲームのフレームバッファ、ペインタのバックエンド、ツールキット自身の内部のように、正当な末端のコードもあります。そのようなコードは明示的にそう宣言します。

```go
p.FillRect(bg, c) //bricolint:allow engine SVG raster blit — a genuine leaf
```

```go
//bricolint:allowfile painter back-end — this file IS the leaf
package pdfsurface
```

**理由の記載は必須です**。理由のないディレクティブは無視され、誰かが正当化の理由を書き込むまで指摘が出続けます。

```yaml
jobs:
  bricolint:
    uses: go-widgets/bricolint/.github/workflows/bricolint.yml@main
```
