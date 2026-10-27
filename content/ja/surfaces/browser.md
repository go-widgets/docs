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
