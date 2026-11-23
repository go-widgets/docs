---
title: "ブラウザのタブ"
linkTitle: "ブラウザ"
weight: 30
description: "go-widgets/webcanvas を使って素の <canvas> でウィジェットツリーを実行する、または wasmdesk コンポジターのウィンドウとして実行する。"
tags: [surfaces, wasm, browser, webcanvas, wasmdesk]
---

ツールキットは DOM に依存しない純粋な Go なので、`GOOS=js GOARCH=wasm` で他のターゲットと同じようにビルドできます。
その結果を人の目に届ける方法は 2 つあります。

## 素のキャンバス {#a-plain-canvas}

[`go-widgets/webcanvas`](https://github.com/go-widgets/webcanvas) は、RGBA のフレームバッファを `<canvas>` に
転送し、DOM のポインタイベントとキーボードイベントをシーンに送り返します。コンポジターも `SharedArrayBuffer` も
クロスオリジン分離ヘッダーも不要で、どんな静的ホスティングでも配信できます。

```go
//go:build js && wasm

package main

import "github.com/go-widgets/webcanvas"

func main() { webcanvas.Run("screen", myScene()) }
```

シーンは `App` を実装します。`Size`、`Draw(buf []byte)`、そしてイベントの種類ごとに 1 つのメソッドを持ち、
各メソッドは何かが変わったかどうかを報告するので、変化のないフレームは再描画されません。`Ticker`、`Animator`、
`Resizer`、`Scroller` も実装できます。

[ギャラリー](https://go-widgets.github.io/gallery/)と
[`app-template`]({{< relref "/getting-started.md" >}}) はこの方法で動いています。

あるいは `window` に選ばせることもできます。window v0.87.0 以降、`js/wasm` 向けにビルドされた
[`window.Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}}) は、通常のページ上では、
`webcanvas` を通してページの `<canvas>`（`Config.Canvas`、デフォルトは `"screen"`）に描画するバックエンドを
返します。そのため、`window` に対して書かれたアプリケーションは、そのままネイティブウィンドウとしても、
ブラウザのタブでも、wasmdesk でも動作します。

## 変更された部分だけを再描画する {#repainting-only-what-changed}

何もしなければ、毎フレームがシーン全体の `Draw`、フレームバッファ全体のブラウザへのコピー、キャンバス全体の
`putImageData` になります。カーソルが点滅しただけ、あるいは `Layer` 内のゲストの画像が新しいだけでも同じです。
何が変わったかを把握しているシーンは `DrawDamaged(buf []byte) []Rect`（`webcanvas.DamageDrawer`、
webcanvas v0.15.0）を実装し、ホストはその矩形だけをコピーして描き込みます。

- **契約**は [`window`]({{< relref "/surfaces/native-window.md" >}}) の `DamageRenderer` がデスクトップホストに
  与えるものを、ブラウザ向けに言い換えたものです。シーンは変更部分を、フレーム間で保持されるフレームバッファに
  再描画し、再描画した矩形を返します（`webcanvas.Rect` は `toolkit.Rect` です）。結果が**空**なら何も変わって
  おらず、何も描き込まれません。最初のフレームとリサイズ後には、引き続き `Draw` が実行されます。
- **レイヤー**は毎フレーム描き直されますが、シーンの再描画は伴いません。移動、縮小、消滅したレイヤーの古い矩形は
  シーンのバッファから復元されます。`Alpha < 1` のレイヤー、または `Opaque` でないレイヤーの下では、毎フレーム
  シーンが復元されます。`Opaque: true` はすべてのピクセルが不透明であることを約束し、静止したゲストの画像が
  何も描き込まないようにします。
- **使わなくても**何も変わりません。`DrawDamaged` のないシーンは従来どおり全体が描画され、描き込まれます。
  `webcanvas/gpu` プレゼンターには独自の版があります(後述)。
- **計測**：node 上の `GOOS=js`、1280×800、偽の 2D コンテキストで、120×40 のウィジェットを 1 つ変えるフレームは
  2.89 ms ではなく 0.031 ms、変更がなく不透明なレイヤーが 1 つあるフレームは 2.90 ms ではなく 0.021 ms です。

## GPU での描画 {#on-the-gpu}

`webcanvas.RunWith(id, app, presenter)` は `Presenter` を受け取り、パッケージ
`webcanvas/gpu` が WebGPU でフレームを描く Presenter を提供します。
- **仕組み:** `Paint(painter.Painter)` も実装している App は、
  [gpupainter](https://github.com/go-widgets/gpupainter) を通して
  [go-gfx/gpu2d](https://github.com/go-gfx/gpu2d) のリストに記録されます。
  WebGPU レンダラーがフレームを描き、GPU 上でキャンバスのテクスチャにコピーされます。
- **Presenter の理由:** キャンバスは一生に一種類のコンテキストしか持てないため、
  Presenter が最初のフレームの前に一度だけ決めます。
- **正確さ:** ピクセルは `Draw` の CPU フレームとバイト単位で同一です(テストで保証)。
- **ダメージ:** `PaintDamaged(painter.Painter) []Rect` も実装している App
  (`gpu.DamagePainted`、webcanvas v0.16.0 と v0.17.0)は、toolkit の `RenderDamaged` と同じく、
  変更された部分だけを、返した矩形の範囲に限って painter に描き込みます。結果が**空**なら、
  そのフレームは記録も表示もされません。静止したフレームは GPU にコマンドバッファを送りません
  (以前は 2、今は 0。Chrome で計測、アロケーション 0)、その記録にかかる時間は中央値で 122.9 µs ではなく 11.4 ns です
  (Apple M4 Max、1 コア、負荷 3 未満で 5 ラウンド、シーン全体の記録との比較)。ダメージのあるフレームは
  `PaintDamaged` が描いたとおりに記録されるため、シーンはダメージの下の地を描き直す必要があります。
  1,200 セルのうち 1 セルがダメージを受けた場合、レンダラーに渡るリストは 2,401 インスタンスから 2 になり、その記録にかかる時間は中央値で 133.2 µs ではなく 129.9 ns です
  (同じマシン、同じラウンド)。
  最初のフレーム、リサイズ、`PaintDamaged` のない App は全体が記録されます。
- **キャンバスのテクスチャは残りません:** 表示のたびに、WebGPU の仕様は `getCurrentTexture()` に
  透明な黒のテクスチャを与えます(Chrome も同じ)。そのためレンダラー自身のターゲットがフレームの残りを保持し、
  表示される各フレームは今も全体を GPU 上でキャンバスにコピーします。
- **フォールバック:** WebGPU のないブラウザや `Paint` のない App は 2D キャンバスのままです。
  どちらもエラーではありません。

[Web ギャラリー](https://go-widgets.github.io/gallery/)はこの方法で描画しています。

## wasmdesk デスクトップ上のウィンドウ {#a-window-on-the-wasmdesk-desktop}

wasmdesk のワーカー内では、[`window.Open`]({{< relref "/surfaces/native-window.md" >}}) は
[wasmdesk/wasmbox](https://github.com/wasmdesk/wasmbox) ブラウザコンポジターのクライアントを返します。
このクライアントは `SharedArrayBuffer` にサーフェスを確保し、`MessagePort` 越しに `hello` を送り、`welcome` を
待ち、共有バッファに描画して `commit` を送ります。`commit` はサーフェス全体、またはダメージを受けた矩形だけです。
コンポジターの `input` メッセージは、X11 の場合と同様に `toolkit.Event` になります。

したがって、`window` に対して書かれたアプリケーションは、ネイティブウィンドウとしても、ブラウザのタブの中の
デスクトップのウィンドウとしても、そのまま動作します。
[デスクトップシェルのデモ](https://github.com/go-widgets/desktop)は、まさにそれを 1 つのソースから行います。

プロトコルのコーデックは、すべてのプラットフォームで 100% カバーされています。実際の動作の検証では、ヘッドレスの
Chromium を本物の wasmdesk コンポジターに対して実行します。クライアントを起動し、コンポジターが合成した
ピクセルを読み取り、クリックして、そのクリックがウィジェットに届いたことを確認します。
