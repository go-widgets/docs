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

## SVG への記録 {#recording-to-svg}

`svgrec.Recorder`（パッケージ `github.com/go-widgets/painter/svgrec`）は 3 つ目のペインタです。同じ呼び出しを**ネイティブな SVG ドキュメント**に変換します。出力は `<rect>`、`<path>`、`<clipPath>` の各要素で、`<image>` はラスタが実際に描かれる箇所にだけ使われます。結果は劣化がなく拡大縮小でき、しかも決定的です。同じ呼び出しからは同じバイト列が得られます。

```go
r, _ := svgrec.New(1000, 700)
widget.Draw(r, painter.LightTheme())
doc, err := r.SVG() // err is the first call that was refused, if any
```

`Clipper`、`Translator`、`PathPainter`、`ImagePainter`、`MaskPainter` を実装しています。SVG に忠実な対応物がない場合、対応表は黙って近似するのではなく、その旨を明記します。

| 呼び出し | SVG |
|---|---|
| `FillRect`、`PutPixel` | `<rect>` |
| `FillRoundRect` | `<rect rx>`。角のアンチエイリアスはピクセルペインタとは異なる |
| `StrokeRect`、`StrokeRoundRect` | `evenodd` の枠。リングはピクセルペインタと同様に内側へ描かれる |
| `Text` | グリフのアウトライン（後述） |
| `PushClip` / `PushTranslate` | `<clipPath>` と入れ子の `<g>`。平行移動は座標に適用される |
| `FillPath`、`StrokePath` | `<path>`。線幅はユーザー単位なので viewBox に合わせて拡大縮小される。長さ 0 のサブパスは、ピクセルペインタでは何も描かれないが、SVG では点が描かれる |
| `DrawImage`、`DrawMask` | PNG の data URI を持つ `<image>` |

**テキストは `<text>` ではなくアウトラインになります。** 組み込みの 5x7 フォントはピクセルの格子なので、各文字列は連結されたランからなる 1 つの `<path>` になり、支援技術向けにそのテキストを `<title>` に持つ `<g>` で包まれます。サブセットフォントを埋め込むには、フォントライタと、それを尊重するビューアが必要です。尊重しないビューアは別の書体にフォールバックし、レイアウトがずれてしまいます。代償として、テキストは選択できません。TrueType の書体で組まれたテキストはまだアウトラインとしては記録されません。`FacePainter` は提供されないため、ラスタの `DrawMask` として届きます。

ペインタのメソッドはエラーを返さないため、拒否された呼び出し（不正な UTF-8、XML が表せない文字、1<<24 を超える座標、`Limits` を超える呼び出しやドキュメント）は破棄され、最初のエラーが `Err()` と `SVG()` で報告されます。ファズターゲットは出力をすべて `encoding/xml` で解析し直し、ライタ自身が生成しない要素や属性があれば失敗します。

ピクセルペインタと比べると（純 Go の `go-gfx/gfx/svg` でラスタライズした場合）、サンプルウィジェット、矩形、ビットマップテキスト、クリップ、平行移動はピクセル単位で一致します。パスの差はチャンネルあたり最大 1、アンチエイリアスされた角丸の差は最悪のシーンで、ピクセルの 7.5 % においてチャンネルあたり最大 61 です。

## 試してみる {#trying-it}

リポジトリ自身のデモは、同じ 3 つのウィジェットを両方のペインタで描画します。

```sh
go run ./cmd/wui-demo --out demo.png     # pixels, written as a PNG
go run ./cmd/tui-demo --theme dark       # cells, written as ANSI to stdout
```

また、[go-widgets.github.io/painter](https://go-widgets.github.io/painter/) では、ピクセルペインタをブラウザ上で確認できます。
