---
title: "ペインタの継ぎ目"
linkTitle: "ペインタ"
weight: 10
description: "すべてのウィジェットが描画に使うプリミティブ、ペインタが追加で持てるオプションの機能、ピクセルペインタとセルペインタ。"
tags: [rendering, painter]
---

[`go-widgets/painter`](https://github.com/go-widgets/painter) は、すべてのウィジェットが描画に使う唯一のインターフェースを定義しています。

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

このセットは意図的に小さくしてあります。すべてのバックエンドがそのすべてを実装しなければならず、ターミナルのセル格子では角を丸めることも線幅を変えることもできないからです。そのため、契約には、できない場合の振る舞いが定められています。`StrokeRect` の線幅はヒントであり、セル格子では無視されます。`FillRoundRect` はセル格子では角を丸めない塗りつぶしにフォールバックします。

## オプションの機能 {#optional-capabilities}

すべてのサーフェスができるわけではないことは、基本インターフェースには含まれていません。それを必要とするウィジェットは型アサーションを行い、答えが「できない」の場合は妥当な代替を描画します。

| インターフェース | メソッド | 用途 |
|---|---|---|
| `Clipper` | `PushClip`, `PopClip` | スクロールされた子を親の境界内に収める |
| `Translator` | `PushTranslate`, `PopTranslate` | 子を子自身の座標系で描画する |
| `PathPainter` | `FillPath`, `StrokePath` | 曲線、グラフ、アンチエイリアスされた図形 |
| `ImagePainter` | `DrawImage` | アイコン、サムネイル、画像 |
| `MaskPainter` | `DrawMask` | グリフなど、単色のカバレッジマスク |
| `FacePainter` | `TextFace` | 特定のフォントフェースとサイズでのテキスト |

```go
if c, ok := p.(painter.Clipper); ok {
	c.PushClip(bounds)
	defer c.PopClip()
}
```

## 2 種類のペインタ {#the-two-painters}

| | 書き込み先 | 表示するもの |
|---|---|---|
| `NewPixelPainter(buf, w, h)` | 呼び出し側が所有する RGBA の `[]byte`（BGRA サーフェス向けには `NewPixelPainterBGRA`） | ネイティブウィンドウ、ブラウザの canvas、Android のサーフェス、PNG または SVG |
| `NewCellPainter(w, h)` | セル（ルーン、前景色、背景色）の格子と、24 ビット ANSI のシリアライザ | [`tui`]({{< relref "/surfaces/terminal.md" >}}) を通したターミナル |

バッファは呼び出し側のものです。`PixelPainter` はプリミティブを書き込みに変換するだけなので、同じ `[]byte` を、コピーを挟まずにウィンドウのバックエンド、`<canvas>`、画像エンコーダに渡せます。

## 試してみる {#trying-it}

リポジトリ自身のデモは、同じ 3 つのウィジェットを両方のペインタで描画します。

```sh
go run ./cmd/wui-demo --out demo.png     # pixels, written as a PNG
go run ./cmd/tui-demo --theme dark       # cells, written as ANSI to stdout
```

また、[go-widgets.github.io/painter](https://go-widgets.github.io/painter/) では、ピクセルペインタをブラウザ上で確認できます。
