---
title: "モジュール"
weight: 50
description: "go-widgets の 19 個のモジュールについて、それぞれの用途と、ソースおよび API リファレンスの場所を示します。"
tags: [modules]
---

API リファレンスは [pkg.go.dev](https://pkg.go.dev/) がソースから生成しており、リリースされたタグに追従します。これらのページは解説を目的としており、リファレンスをコピーするものではありません。タグは執筆時点で最新のものです。正式な情報源はリポジトリ自身のタグ一覧です。

| モジュール | タグ | 概要 | リファレンス |
|---|---|---|---|
| [`painter`](https://github.com/go-widgets/painter) | v0.16.0 | [描画の継ぎ目]({{< relref "/rendering/painter.md" >}}): ピクセルペインタとセルペインタ | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/painter) |
| [`toolkit`](https://github.com/go-widgets/toolkit) | v0.345.1 | [ウィジェットセット]({{< relref "/rendering/toolkit.md" >}})、テーマ、レイアウト、テキスト | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/toolkit) |
| [`skin`](https://github.com/go-widgets/skin) | v0.3.0 | [データとして記述されたウィジェット]({{< relref "/rendering/skin.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/skin) |
| [`isoicons`](https://github.com/go-widgets/isoicons) | v0.4.0 | ツールキットのアイソメトリック図のための、アイソメトリックアイコンパック（クラウドネイティブ、AWS） | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/isoicons) |
| [`mvvm`](https://github.com/go-widgets/mvvm) | v0.15.0 | [オブザーバブル、コマンド、バインダ、undo]({{< relref "/state/mvvm.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvm) |
| [`mvvmtk`](https://github.com/go-widgets/mvvmtk) | v0.15.0 | [ツールキットのウィジェット向けの 1 呼び出しバインダ]({{< relref "/state/mvvm.md#mvvmtk-one-call-per-widget" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmtk) |
| [`data`](https://github.com/go-widgets/data) | v0.4.0 | [型付きレコード、クエリ、ローカルまたはリモートのストア]({{< relref "/state/data.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/data) |
| [`window`](https://github.com/go-widgets/window) | v0.89.1 | 8 つのバックエンドに対応した[ネイティブウィンドウ]({{< relref "/surfaces/native-window.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/window) |
| [`application`](https://github.com/go-widgets/application) | v0.8.1 | ウィンドウの上に載る[アプリケーションのライフサイクル]({{< relref "/surfaces/application.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/application) |
| [`tray`](https://github.com/go-widgets/tray) | v0.15.0 | [トレイアイコンとそのメニュー]({{< relref "/surfaces/application.md#the-tray" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tray) |
| [`webcanvas`](https://github.com/go-widgets/webcanvas) | v0.4.1 | [ブラウザの `<canvas>` 内のシーン]({{< relref "/surfaces/browser.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/webcanvas) |
| [`tui`](https://github.com/go-widgets/tui) | v0.62.0 | [ターミナルへの描画とセルネイティブなウィジェット]({{< relref "/surfaces/terminal.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tui) |
| [`android`](https://github.com/go-widgets/android) | v0.16.0 | [CGO を使わないプロセスが描画する APK]({{< relref "/surfaces/android.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/android) |
| [`svg`](https://github.com/go-widgets/svg) | v0.7.0 | [SVG または PNG としての描画結果]({{< relref "/surfaces/snapshots.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/svg) |
| [`mvvmlint`](https://github.com/go-widgets/mvvmlint) | v0.4.2 | [MVVM のチェック]({{< relref "/guard-rails/mvvmlint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmlint) |
| [`bricolint`](https://github.com/go-widgets/bricolint) | v0.4.1 | [手描き UI 禁止のチェック]({{< relref "/guard-rails/bricolint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/bricolint) |
| [`app-template`](https://github.com/go-widgets/app-template) | v0.5.0 | [アプリケーションの出発点]({{< relref "/getting-started.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/app-template) |
| [`gallery`](https://github.com/go-widgets/gallery) | v0.7.0 | すべてのウィジェットファミリーの[ライブデモ](https://go-widgets.github.io/gallery/) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/gallery) |
| [`desktop`](https://github.com/go-widgets/desktop) | v0.20.0 | go-freedesktop と go-widgets を組み合わせたデスクトップシェル。ネイティブでも wasmdesk 上でも動作 | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/desktop) |

## すべてのモジュールが守る条件 {#what-every-module-is-held-to}

- `CGO_ENABLED=0`: cgo は使わず、ライブラリの代わりにコマンドラインツールを呼び出すこともしません。
- amd64、arm64、riscv64、loong64、ppc64le、s390x にクロスコンパイルします。最後のものはビッグエンディアンで、あらゆるワイヤエンコーディングの正しさを保つ助けになります。該当する場合は `js/wasm` にも対応します。17 個のライブラリとツールすべてが 2026-10-10 以降これを満たしています。残りの 2 つはブラウザアプリケーションです（[状況]({{< relref "/status.md#what-ci-does-not-cover-yet" >}})）。
- CI でのステートメントカバレッジのゲート: 19 個すべてで 100% です。テストで実行できないファイル（ブラウザのループ、ネイティブの実行ループ）がある場合、モジュール側でどのファイルを除外しているか、およびその理由を明記しています。例外については [状況]({{< relref "/status.md" >}}) を参照してください。
- BSD-3-Clause。
