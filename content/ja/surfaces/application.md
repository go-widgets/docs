---
title: "アプリケーションとそのトレイ"
linkTitle: "アプリケーションとトレイ"
weight: 20
description: "go-widgets/application はウィンドウの上位のライフサイクルを担い、go-widgets/tray はメニューバーまたは通知領域にアイコンとメニューを置きます。"
tags: [surfaces, application, tray]
---

## アプリケーション {#the-application}

[`go-widgets/application`](https://github.com/go-widgets/application) は
[`window`]({{< relref "/surfaces/native-window.md" >}}) の上位のレイヤーです。ウィンドウを開き、そのループを回し、
最初のフレームが表示されたらトレイアイコンを取り付け、システムの現在の外観（ダークかライトか、アクセントカラー、
システムフォント）をアプリケーションに渡し、アクセシビリティツリーを公開し、最初のフレームが画面に表示されたら
`onReady` を呼び出します。

```go
spec := application.Spec{
	Name:       "News Reader",
	Identifier: "com.example.reader",
	Version:    "1.0.0",
	Tray: func() *tray.Menu {
		return tray.NewMenu().Add(
			tray.Item("Refresh", refresh),
			tray.Item("Quit", quit),
		)
	},
}
cfg := application.Config{Title: "News Reader", Width: 1200, Height: 800}
err := application.Run(spec, cfg, handler, func() { log.Println("on screen") })
```

`handler` は `Handler`（`Frame`、`Resize`、マウス、スクロール、キーのイベント）を実装し、さらにオプションの
機能（外観、ショートカット、修飾キー付きのクリックとキー、副クリック、コンテキストメニュー、クリップボード、
アクセシビリティ、ネイティブコントロール）のどれでも実装できます。ループはハンドラをラップしません。
各機能を型アサーションで確認し、見つかったものに対応します。テストがパッケージ内のそれらのアサーションを
洗い出し、パッケージのドキュメントがそのすべてに言及していなければ失敗します。

取り付けられないトレイは、沈黙ではなく `Run` のエラーになります。`errors.Is(err, tray.ErrNoBackend)` で理由が
分かります。トレイがどうなってもウィンドウは必要というアプリケーションは、`Spec.Tray` を nil のままにして、
自前でトレイを実行します。

`Run` はループが戻るときにウィンドウを閉じるので、トレイアプリケーションの「開く」から `Run` をもう一度呼び出して
新しいウィンドウを開けます。ウィンドウではないホスト（デスクトップシェルやブラウザのタブ）向けには、`Bind` と
`BindScaled` が、同じハンドラに接続された `*toolkit.Surface` を返します。

## メニューバー {#menu-bar}

`application.Config.Menu` はトレイアイコンと同じ `tray.Menu` を受け取り、
アプリケーションのメニューバーとして設定します。`application.SetMenu` は
任意の goroutine から後で変更でき、メニューが変わったときだけ再設定します。
`tray.EditMenu()` は標準の編集メニュー (取り消す、カット、コピー、ペースト、
すべてを選択) です。

- macOS: 画面上部のメニュー。アプリケーションメニューが先頭に来ます。
- Windows: ウィンドウのメニューバー。キーの割り当てはテキストで表示されます。
- X11、Wayland、GTK、wasm、Android: `window.ErrNotSupported` を返し、`Run` は無視します。
  GTK は未対応です。GTK バインディングにはまだメニューモデルがありません。

## 描画されたウィジェットのアクセシビリティ {#painted-accessibility}

独自の[ツールキット]({{< relref "/rendering/toolkit.md#accessibility" >}})ウィジェットを描画するハンドラーは、`Accessible` を実装しない限り、プラットフォームのアクセシビリティ層からは不透明な長方形 1 枚にしか見えません。`application.A11yElementsOf(root)` は、ウィジェットツリーからその記述（各ウィジェットのロール、ラベル、値、境界。最大 `MaxA11yElements` = 1000）を作ります。

```go
func (v *View) A11yElements() []application.A11yElement {
	return application.A11yElementsOf(v.root)
}
```

macOS では、描画されたボタンが AX ツリーに `AXButton` としてラベル付きで現れ、それに対する `AXPress` はクリックになります。ハンドラーが埋め込むネイティブコントロールも、その隣のツリーに残ります。Windows（UI Automation）と Linux（AT-SPI）のバックエンドも同じ記述を使います。要素の無効状態はまだ公開されず、Windows のブリッジは要素の矩形を拒否します。

## トレイ {#the-tray}

トレイアイコンは OS との統合であり、描画されるウィジェットではありません。そのため
[`go-widgets/tray`](https://github.com/go-widgets/tray) は各プラットフォーム固有の API を、やはり
`CGO_ENABLED=0` のまま操作します。

| プラットフォーム | API | 経由するもの |
|---|---|---|
| macOS | `NSStatusItem`, `NSMenu` | purego と Objective-C ランタイム |
| Windows | `Shell_NotifyIcon`, `TrackPopupMenu` | `golang.org/x/sys/windows` |
| Linux | `StatusNotifierItem`, `com.canonical.dbusmenu` | DBus（Go で実装） |

```go
menu := tray.NewMenu().Add(
	tray.Item("Open", open),
	tray.Checkbox("Notifications", true, setNotify),
	tray.SubMenu("Recent", tray.NewMenu().Add(tray.Item("file.txt", nil))),
	tray.Separator(),
	tray.Item("Quit", func() { t.Quit() }),
)
t := tray.New(iconPNG).SetTooltip("My App").SetMenu(menu)
t.Run() // blocks on the platform's loop until Quit
```

ウィンドウのループをすでに実行しているプログラムは、`Run` にメインスレッドを渡せません。`Attach` は同じアイコンを
表示して戻り、`Quit` がそれを取り除きます。

ネイティブのバックエンドはデフォルトで有効で、覚えておくべきビルドタグはありません。バックエンドがない
プラットフォームでは、`Run` は `ErrNoBackend` を返します。これは「ここにはトレイがない」ことと、何もせずに
黙っているトレイとを区別するためです。テストやヘッドレスのサービスは `WithBackend(tray.NewHeadless())` を
渡します。

### 通知

`(*Tray).Notify(tray.Notification{Title, Body, OnClick})` はネイティブのシステム通知を出します。macOS では UNUserNotificationCenter、Linux では `org.freedesktop.Notifications`、Windows では Shell_NotifyIcon のバルーンです。タイトルは最大 63 ルーン、本文は最大 255 ルーンです（`MaxNotificationTitle`、`MaxNotificationBody`）。タイトルが空だと `ErrEmptyTitle` になります。エラーは原因を示します。`ErrNoNotifier`（バックエンドなし）、`ErrNotRunning`、`ErrNotBundled`（macOS はアプリバンドルにだけ配信します）、`ErrNotAuthorized`（ユーザーが拒否）。`tray.NewHeadless()` は送られた内容をテスト用に `Headless.Notified` に記録します。

## リンクを開く

`application.OpenURL(rawURL)` は `http` または `https` の URL をユーザーのブラウザで開きます。macOS では NSWorkspace、Windows では ShellExecuteW、Linux では OpenURI ポータルに続いて `xdg-open` を使います。URL にはホストが必要で、ポートがある場合は 1〜65535、全体は 8 KiB 以下でなければなりません。それ以外は `ErrURLScheme` または `ErrURLInvalid` で拒否され、ランチャーのないプラットフォームでは `ErrOpenURLUnsupported` が返ります。
