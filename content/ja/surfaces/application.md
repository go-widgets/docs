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
