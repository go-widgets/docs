---
title: "go-widgets"
linkTitle: "ホーム"
type: docs
cascade:
  type: docs
description: "純粋な Go で書かれたウィジェットツールキットと、それが描画するすべてのサーフェス: X11、Wayland、macOS、Windows 上のネイティブウィンドウ、ブラウザの canvas、ターミナル、Android APK。"
---

**1 つのウィジェットツリーを Go が描画し、Go が届くあらゆるサーフェスに表示します。** X11、Wayland、macOS、Windows のネイティブウィンドウ、ブラウザタブ内の `<canvas>`、ターミナル、Android APK に対応しています。C ツールチェーンも、webview も、システムのウィジェットセットも不要です。すべてのモジュールが `CGO_ENABLED=0` でビルドでき、すべてのピクセルをツールキット自身が描画します。

```go
w, err := window.Open(window.Config{Title: "Hello"})
if err != nil {
	return err
}
defer w.Close()

box := toolkit.NewVBox()
box.Append(toolkit.NewLabel("Hello from go-widgets"))
box.Append(toolkit.NewButton("Click me", func() { /* ... */ }))
return w.Run(box) // an X11 or Wayland window, an NSWindow, a Win32 window
```

これは
[`cmd/windowdemo`](https://github.com/go-widgets/window/blob/main/cmd/windowdemo/main.go)
を短くしたものです。同じ `box` をブラウザ、ターミナル、PNG に描画する場合も、コードは同じで、下にあるサーフェスが違うだけです。

## なぜ 1 つのツールキットですべてを描画するのか {#why-one-toolkit-draws-everything}

プラットフォーム固有のボタンを呼び出すウィジェットは、対象とするプラットフォームのうち最も機能の乏しいものと同じ程度にしか移植できず、見た目もプラットフォームごとに異なります。go-widgets はその逆を行きます。ウィジェットの `Draw` が話す相手は
[`painter.Painter`]({{< relref "/rendering/painter.md" >}})
だけで、これは矩形の塗りつぶし、矩形の輪郭描画、ピクセルの設定、テキストの描画といった少数のプリミティブから成ります。これらの呼び出しが RGBA ピクセルになるのか、ターミナルのセルになるのかは、バックエンドの責務であり、ウィジェットの関知するところではありません。

その代償は率直に述べておきます。既定ではネイティブな外観にならず、アクセシビリティも無料では得られません。ツールキットが独自の
[アクセシビリティツリー]({{< relref "/rendering/toolkit.md#accessibility" >}})
を各プラットフォームに公開します。得られるのは、1 度だけテストされ、どこでも同じ動作をする 1 つのウィジェットセットと、どのマシンからでもそれらすべてへクロスコンパイルできるバイナリです。

## 次に読むページ {#where-to-go-next}

| | |
|---|---|
| [最初のアプリケーション]({{< relref "/getting-started.md" >}}) | `app-template` から、または素のウィンドウから |
| [ウィジェットが画面に届くまで]({{< relref "/rendering/_index.md" >}}) | ペインタ（painter）の継ぎ目、ウィジェットセット、スキン |
| [ペインタの継ぎ目]({{< relref "/rendering/painter.md" >}}) | すべてのウィジェットが描画に使うプリミティブ |
| [ツールキット]({{< relref "/rendering/toolkit.md" >}}) | 160 種類のウィジェット、テーマ、レイアウト、テキスト、アクセシビリティ |
| [スキン]({{< relref "/rendering/skin.md" >}}) | ウィジェットのパーツと状態をデータファイルで記述 |
| [状態はビューモデルに置く]({{< relref "/state/_index.md" >}}) | ウィジェットのフィールドを手で代入しない理由 |
| [MVVM]({{< relref "/state/mvvm.md" >}}) | オブザーバブル、コマンド、バインダ、undo |
| [データの背骨]({{< relref "/state/data.md" >}}) | 型付きレコード、クエリ、ローカルまたはリモートのストア |
| [描画先]({{< relref "/surfaces/_index.md" >}}) | すべてのバックエンドと、その選ばれ方 |
| [ネイティブウィンドウ]({{< relref "/surfaces/native-window.md" >}}) | X11、Wayland、Cocoa、Win32、GTK4、Android、wasmbox、ブラウザタブ |
| [アプリケーションとそのトレイ]({{< relref "/surfaces/application.md" >}}) | ライフサイクル、外観、メニューバーのアイコン |
| [ブラウザタブ]({{< relref "/surfaces/browser.md" >}}) | 素の `<canvas>`、または wasmdesk のウィンドウ |
| [ターミナル]({{< relref "/surfaces/terminal.md" >}}) | セルネイティブなウィジェットと対話型ランナー |
| [Android]({{< relref "/surfaces/android.md" >}}) | CGO を使わないプロセスが描画する本物の APK |
| [スナップショット]({{< relref "/surfaces/snapshots.md" >}}) | ウィジェットを SVG または PNG として出力 |
| [CI が強制するルール]({{< relref "/guard-rails/_index.md" >}}) | `mvvmlint` と `bricolint` |
| [モジュール]({{< relref "/modules.md" >}}) | 19 個すべてについて、ソースとリファレンス |
| [状況]({{< relref "/status.md" >}}) | どこで何が検証済みで、何が未完了か |

## ライセンス {#licence}

BSD-3-Clause。
