---
title: "最初のアプリケーション"
weight: 5
description: "MVVM に準拠済みで CI も通る app-template から始めるか、素のウィンドウを開いてウィジェットを追加します。"
tags: [getting-started, app-template, window]
---

## テンプレートから {#from-the-template}

[`go-widgets/app-template`](https://github.com/go-widgets/app-template) はテンプレートリポジトリで、検索フィールド、カテゴリフィルタ、一致した行の一覧、ステータス行を備えた、小さいながら実用的なブラウザアプリケーションです。より大きなアプリケーションに必要なものはすべて最初から組み込まれています。ウィジェットを含まないビューモデル、それにバインドされたビュー、wasm ホスト、100% のカバレッジゲート、そして
[MVVM ゲート]({{< relref "/guard-rails/mvvmlint.md" >}})
です。

```sh
git clone https://github.com/go-widgets/app-template my-app
cd my-app
go mod edit -module github.com/you/my-app
./build.sh          # dist/app.wasm, dist/wasm_exec.js, dist/index.html
```

`dist/` は HTTP で配信してください。`file://` から開いたページでは wasm をインスタンス化できません。

| ファイル | 内容 |
|---|---|
| `viewmodel.go` | すべての状態を [`mvvm`]({{< relref "/state/mvvm.md" >}}) のオブザーバブルとコマンドとして保持します。ウィジェットは含みません。canvas なしでテストできます。 |
| `scene.go` | ウィジェット群で、それぞれが `mvvmtk` を通じてビューモデルにバインドされています。ウィジェットの状態フィールドを自分で代入することはありません。 |
| `main.go` | ビルドタグ（`js && wasm`）を持つ唯一のファイルです。入力をツールキットに転送し、フレームを `<canvas>` に転送します。 |

編集するのは最初の 2 つのファイルです。状態は常にバインダ経由で流してください。CI の `mvvm` ジョブは、ウィジェットのフィールドを直接代入するプルリクエストを失敗させます。

## 素のウィンドウから {#from-a-bare-window}

```go
package main

import (
	"github.com/go-widgets/toolkit"
	"github.com/go-widgets/window"
)

func main() {
	w, err := window.Open(window.Config{Title: "Demo"})
	if err == window.ErrUnsupported {
		return // no native back-end on this platform
	}
	if err != nil {
		panic(err)
	}
	defer w.Close()

	box := toolkit.NewVBox()
	box.Append(toolkit.NewLabel("Hello"))
	box.Append(toolkit.NewButton("Click me", func() {}))
	w.Run(box) // layout, draw, present, dispatch input, until closed
}
```

幅と高さを 0 のままにすると、バックエンドが読みやすい既定値を選びます。macOS ではメイン画面の何分の一かの大きさになります。`CGO_ENABLED=0 go build` を実行すると、どのマシンからでも各プラットフォーム向けのバイナリが生成されます。

## 目的別のモジュール {#which-module-for-what}

| 目的 | 最初に使うもの |
|---|---|
| デスクトップのウィンドウ | [`window`]({{< relref "/surfaces/native-window.md" >}}) |
| トレイアイコンとシステムの外観を備えたデスクトップアプリケーション | [`application`]({{< relref "/surfaces/application.md" >}}) |
| ブラウザ上のページ | [`webcanvas`]({{< relref "/surfaces/browser.md" >}})、または上記のテンプレート |
| ターミナルプログラム | [`tui`]({{< relref "/surfaces/terminal.md" >}}) |
| Android アプリ | [`android`]({{< relref "/surfaces/android.md" >}}) |
| README 用のウィジェットの画像 | [`svg`]({{< relref "/surfaces/snapshots.md" >}}) |

何かを書く前にウィジェットセットを見たい場合は、
[ギャラリー](https://go-widgets.github.io/gallery/)
を開いてください。すべてのウィジェットファミリーを、ブラウザの canvas 上でライブで確認できます。
