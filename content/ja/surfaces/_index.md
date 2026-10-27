---
title: "描画先"
linkTitle: "サーフェス"
weight: 30
description: "go-widgets のツリーを表示できるすべてのサーフェスと、それらの間の選択方法。"
tags: [surfaces, back-ends]
---

| サーフェス | モジュール | ペインタ |
|---|---|---|
| X11、Wayland、macOS、Windows の[ネイティブウィンドウ]({{< relref "/surfaces/native-window.md" >}}) | `window` | ピクセル |
| トレイアイコンとシステムの外観を備えた[アプリケーション]({{< relref "/surfaces/application.md" >}}) | `application`, `tray` | ピクセル |
| [ブラウザのタブ]({{< relref "/surfaces/browser.md" >}})：素の `<canvas>` | `webcanvas`、または `window` | ピクセル |
| [ブラウザ上のデスクトップ]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}})：wasmdesk コンポジターのウィンドウ | `window`（wasmbox） | ピクセル |
| [ターミナル]({{< relref "/surfaces/terminal.md" >}}) | `tui` | セル |
| [Android の APK]({{< relref "/surfaces/android.md" >}}) | `android`, `window` | ピクセル |
| [画像]({{< relref "/surfaces/snapshots.md" >}})：SVG または PNG | `svg` | ピクセル |

ウィジェットツリーは、どのサーフェスでも変わりません。変わるのは、バッファを所有するのが誰か、入力を届けるのが
誰か（ディスプレイサーバー、ブラウザ、ターミナル、Java ホスト）という点で、各バックエンドは、その入力を同じ
`toolkit.Event` に変換します。

## 存在するものに応じて選ばれる {#chosen-by-what-is-there}

`window.Open` はバックエンドの引数を取りません。環境、つまりビルド対象のプラットフォーム、`$WAYLAND_DISPLAY`、
`$DISPLAY`、Android ホストがエクスポートするソケットを調べ、動作できるものを選びます。Linux 向けにビルドした
バイナリは、Wayland セッションでは Wayland ウィンドウを、X では X11 ウィンドウを開きます。同じ Android の
バイナリが、APK の中ではホストに接続し、Termux の下では通常の Linux ウィンドウを開きます。
[`Open` の選び方]({{< relref "/surfaces/native-window.md#how-open-chooses" >}})を参照してください。
