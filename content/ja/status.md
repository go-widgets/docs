---
title: "状況"
weight: 60
description: "実際のディスプレイ、実機、実際のターミナルで検証済みのもの、コンパイルのみのもの、CI がまだ対象にしていないもの。"
tags: [status]
---

これらのページは、[モジュール]({{< relref "/modules.md" >}}) に記載したタグ時点のモジュールと照合して確認しています。モジュールの README とコードが食い違う場合は、コードに従っています。

## 実環境での検証済み {#proven-live}

プロセス内でテストしたコーデックが証明するのは、そのコーデックだけです。以下は、プラットフォーム自体に問い合わせた箇所です。

| 対象 | 検証した環境 |
|---|---|
| X11 ウィンドウ | CI（Xvfb 上）: 既知のパターンを表示してキャプチャしサンプリング、`xdotool` でクリックとキーを合成して、ディスパッチされた `toolkit.Event` を検証、`Show`/`Hide`/`Raise` をサーバから読み戻し |
| Cocoa ウィンドウ | macOS の CI レーン: ウィンドウを開いて描画し、ピクセルをサンプリング、クリックとキーを合成、別の goroutine から実際の `NSWindow` に `Show`/`Hide`/`Raise` を要求して AppKit から読み戻し |
| Win32 ウィンドウ | Windows 11 arm64 の VM: `VBox`、`Label`、`Button` を描画する実際のウィンドウで、3 回注入したクリックがウィンドウプロシージャ経由でカウンタを 0 から 3 に進めることを確認 |
| Wayland ウィンドウ | xdg-shell の map と unmap の規則を強制する偽のコンポジタに対して検証 |
| wasmbox ウィンドウ | ヘッドレス Chromium で実際の wasmdesk コンポジタを駆動: 合成されたピクセルを読み戻し、実際のクリックをウィジェットにルーティング |
| Android | Android 15 arm64 の実機: 差分のみの blit を計測、memfd のダーティメモリを計測、アクセシビリティツリーを実機でベンチマーク |
| macOS トレイ | 実際の macOS セッション: プログラムの停止でアイコンがメニューバーから消え、起動で戻り、メニューが開く |
| Linux トレイ | 実際のセッションバスを使う CI: 名前が確保され、メニューが `GetLayout` に応答し、`Quit` で取り下げられる |
| Windows トレイ | Windows 11 VM 上のアイコンとそのメニュー |
| ターミナルデモ | CI 内の実際の pty: キーのバイト列を送り、描画されたフレームを検証 |

## コンパイルのみで未実行 {#compiled-not-run}

- Win32 の `Show`/`Hide`/`Raise`: 判定表はユニットテスト済みですが、ウィンドウのスレッド上での呼び出しはコンパイルされているだけです。
- Windows トレイの `Attach`。
- `webcanvas` の DOM ループ（`run_js.go`）: `js && wasm` ビルドタグの内側にあるため、計測対象のカバレッジの外にあります。これを使うアプリケーションが動作を検証しています。
- `application` のネイティブな `Run`: 実際のウィンドウを開く処理はカバレッジゲートから除外されています。契約、イベントの変換、ready カウンタはカバーされています。

## 未完了 {#not-done}

- Windows と Linux のトレイは `MenuItem.Icon` を無視します。アイコン付きの行は、アイコンなしで描画されます。
- `window` の Wayland バックエンドは `Raise` できません。xdg-shell にそのようなリクエストはなく、xdg-activation にはユーザー自身の入力から得たトークンが必要です。
- Android: `android/arm64` のみです。cgo なしで Go がリンクできる Android ターゲットはこれだけだからです。
- `window.Screens` は `js/wasm` では `ErrScreensUnsupported` を返します。

## CI が対象とする範囲 {#what-ci-does-not-cover-yet}

2026-10-10 に、このページに列挙していた 5 つの欠落が解消されました。

| モジュール | 以前 | 現在 |
|---|---|---|
| `mvvmtk` | カバレッジゲートなし。amd64 と arm64 のみ | ツールキットと同じ厳密な 100% のゲート（すでに 100.0% でした）。Linux は 6 種すべて |
| `application` | Linux、macOS、Windows とも amd64 のみ | Linux は 6 種すべて。macOS と Windows は amd64 と arm64 |
| `tray` | OS とアーキテクチャの組み合わせ 5 つ | Linux は 6 種すべて、macOS は 2 種、Windows は 1 種 |
| `mvvmlint` | クロスビルドなし | Linux は 6 種すべて、macOS と Windows |
| `bricolint` | 「6 arches」のステップが実際には 5 つをビルド | loong64 を追加 |

これで、17 個のライブラリとツールすべてが 64 ビット Linux の 6 つのアーキテクチャにクロスビルドされ、19 個すべてが 100% のステートメントカバレッジでゲートされています。`app-template` と `gallery` はブラウザアプリケーションで、唯一のターゲットである `js/wasm` 向けにビルドされます。

## セキュリティ {#security}

2026-10-10 に `govulncheck`（シンボルレベル: モジュールから到達可能なコードのみ）で計測しました。

- 19 個のモジュールのうち 15 個が脆弱なコードに到達していました。go1.27.1 の標準ライブラリ（`html/template`、`net/http`、`crypto/tls`、`mime/multipart`。go1.27.2 で修正）と、`golang.org/x/net` v0.58.0（`data` 内の gRPC 経由の `http2`。v0.60.0 で修正）です。
- 同じ日の夜に修正しました。すべてのモジュールの CI は go1.27.2 でビルドし、すべてのモジュールが `x/net` v0.60.0 を要求しています。多くのモジュールでは間接的な依存で、Renovate が更新しないため、手作業で引き上げました。19 個すべてのモジュールに対する 2 回目の `govulncheck` では、到達可能な脆弱性は見つかっていません。
- リリース: android v0.16.0、app-template v0.5.0、application v0.8.0、bricolint v0.4.1、data v0.4.0、mvvm v0.15.0、mvvmlint v0.4.1、mvvmtk v0.15.0、painter v0.16.0、skin v0.3.0、svg v0.7.0、tray v0.15.0、webcanvas v0.4.1、window v0.89.0。各バージョンは `gorelease` によって API から導出しました。2026-10-11 には toolkit v0.329.0、tui v0.62.0、desktop v0.20.0、gallery v0.7.0、isoicons v0.4.0 をリリースしました。この 4 つについては、モジュールグラフが公開されたことのないバージョンである `github.com/ajroetker/go-highway/hwy/goat` v0.0.0 に到達するため、`gorelease` がモジュールグラフを読み込めません
  （[ajroetker/go-highway#73](https://github.com/ajroetker/go-highway/issues/73)）。そこで、同じルールのもと、linux、darwin、windows 上の `apidiff` でバージョンを導出しました。
- 23 のリポジトリすべてのワークフロー（31 ファイル）を `actionlint` と `wfaudit` で監査しました。特権トリガー、ワークフロー単位の書き込み権限、アップロードされたチェックアウトに残るトークン、シェルステップ内の信頼できない式は、いずれもありません。すべてのワークフローが権限を宣言しているため、リポジトリの既定トークンに依存するものはありません。
