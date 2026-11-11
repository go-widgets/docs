---
title: "モジュール"
weight: 50
description: "go-widgets の 24 個のモジュールについて、それぞれの用途と、ソースおよび API リファレンスの場所を示します。"
tags: [modules]
---

API リファレンスは [pkg.go.dev](https://pkg.go.dev/) がソースから生成しており、リリースされたタグに追従します。これらのページは解説を目的としており、リファレンスをコピーするものではありません。タグは執筆時点で最新のものです。正式な情報源はリポジトリ自身のタグ一覧です。

| モジュール | タグ | 概要 | リファレンス |
|---|---|---|---|
| [`painter`](https://github.com/go-widgets/painter) | v0.17.0 | [描画の継ぎ目]({{< relref "/rendering/painter.md" >}}): ピクセルペインタとセルペインタ | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/painter) |
| [`toolkit`](https://github.com/go-widgets/toolkit) | v0.364.0 | [ウィジェットセット]({{< relref "/rendering/toolkit.md" >}})、テーマ、レイアウト、テキスト | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/toolkit) |
| [`skin`](https://github.com/go-widgets/skin) | v0.3.0 | [データとして記述されたウィジェット]({{< relref "/rendering/skin.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/skin) |
| [`gpupainter`](https://github.com/go-widgets/gpupainter) | v0.4.0 | GPU 向けにフレームを記録する `painter.Painter`（go-gfx/gpu2d）。[Metal または WebGPU]({{< relref "/surfaces/browser.md" >}}) で描画し、CPU ペインタとバイト単位で同一 | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/gpupainter) |
| [`shellterm`](https://github.com/go-widgets/shellterm) | v0.1.0 | 行指向のターミナル。ツールキットの `Terminal` と行エディタを、利用者が用意するシェルの前に置きます | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/shellterm) |
| [`isoicons`](https://github.com/go-widgets/isoicons) | v0.4.0 | ツールキットのアイソメトリック図のための、アイソメトリックアイコンパック（クラウドネイティブ、AWS） | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/isoicons) |
| [`mvvm`](https://github.com/go-widgets/mvvm) | v0.15.0 | [オブザーバブル、コマンド、バインダ、undo]({{< relref "/state/mvvm.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvm) |
| [`mvvmtk`](https://github.com/go-widgets/mvvmtk) | v0.15.0 | [ツールキットのウィジェット向けの 1 呼び出しバインダ]({{< relref "/state/mvvm.md#mvvmtk-one-call-per-widget" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmtk) |
| [`lineedit`](https://github.com/go-widgets/lineedit) | v0.1.1 | 純粋なモデルとしての行エディタ。履歴、逆方向検索、Tab 補完。標準ライブラリのみ | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/lineedit) |
| [`vt`](https://github.com/go-widgets/vt) | v0.1.0 | 純粋なモデルとしての端末エミュレータ。ECMA-48 と xterm のサブセット、バイト列を入力しセルを出力。何も応答しない。標準ライブラリのみ | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/vt) |
| [`vcs`](https://github.com/go-widgets/vcs) | v0.1.0 | 作業ツリーの各ファイルがどの状態にあるかと、ドライバのレジストリ。標準ライブラリのみ。git ドライバはネストされたモジュール `vcs/git`（`git/v0.1.0`） | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/vcs) |
| [`data`](https://github.com/go-widgets/data) | v0.4.0 | [型付きレコード、クエリ、ローカルまたはリモートのストア]({{< relref "/state/data.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/data) |
| [`window`](https://github.com/go-widgets/window) | v0.91.0 | 8 つのバックエンドに対応した[ネイティブウィンドウ]({{< relref "/surfaces/native-window.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/window) |
| [`application`](https://github.com/go-widgets/application) | v0.8.1 | ウィンドウの上に載る[アプリケーションのライフサイクル]({{< relref "/surfaces/application.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/application) |
| [`tray`](https://github.com/go-widgets/tray) | v0.15.0 | [トレイアイコンとそのメニュー]({{< relref "/surfaces/application.md#the-tray" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tray) |
| [`webcanvas`](https://github.com/go-widgets/webcanvas) | v0.9.0 | [ブラウザの `<canvas>` 内のシーン]({{< relref "/surfaces/browser.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/webcanvas) |
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
- amd64、arm64、riscv64、loong64、ppc64le、s390x にクロスコンパイルします。最後のものはビッグエンディアンで、あらゆるワイヤエンコーディングの正しさを保つ助けになります。該当する場合は `js/wasm` にも対応します。22 個のライブラリとツールすべてが、追加された日か 2026-10-10 のいずれか遅い方以降これを満たしています。残りの 2 つはブラウザアプリケーションです（[状況]({{< relref "/status.md#what-ci-does-not-cover-yet" >}})）。
- CI でのステートメントカバレッジのゲート: 24 個すべてで 100% です。テストで実行できないファイル（ブラウザのループ、ネイティブの実行ループ）がある場合、モジュール側でどのファイルを除外しているか、およびその理由を明記しています。例外については [状況]({{< relref "/status.md" >}}) を参照してください。
- BSD-3-Clause。
