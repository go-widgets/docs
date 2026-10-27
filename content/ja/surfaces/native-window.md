---
title: "ネイティブウィンドウ"
linkTitle: "ネイティブウィンドウ"
weight: 10
description: "go-widgets/window：X11、Wayland、GTK4、Cocoa、Win32、Android、wasmbox、ブラウザのタブに対する 1 つの Open/Run API。すべて CGO_ENABLED=0 です。"
tags: [surfaces, window, x11, wayland, macos, windows]
---

[`go-widgets/window`](https://github.com/go-widgets/window) は実際のウィンドウを開き、その中でウィジェットツリーを
実行します。レイアウト、描画、表示、入力の `toolkit.Event` への変換を、ウィンドウが閉じられるまで繰り返します。

```go
w, err := window.Open(window.Config{Title: "Demo"})
if err != nil {
	return err
}
defer w.Close()
return w.Run(root) // root is any toolkit.Widget
```

`Config` では、**論理ポイント**での初期サイズ（0 なら読みやすいデフォルト値）、`WM_CLASS` のインスタンスと
クラス、`Theme`、`RenderScale` も指定できます。

## Open の選び方 {#how-open-chooses}

| プラットフォームと環境 | バックエンド | 画面への届き方 |
|---|---|---|
| Linux、`$GO_WIDGETS_GTK` が設定されている | GTK4 | GTK がウィンドウを所有し、フレームバッファは `GtkPicture` で、ネイティブコントロールはその上にある本物の GTK ウィジェットです。libgtk-4 ランタイムが必要です。 |
| Linux、`$WAYLAND_DISPLAY` が設定されている | Wayland | コンポジターの Unix ソケット越しの xdg-shell |
| Linux、それ以外で `$DISPLAY` がある | X11 | Unix ソケット越しの X11 コアプロトコル。MIT-SHM を使用 |
| macOS | Cocoa | [go-macos/objc](https://github.com/go-macos/objc)（purego）経由の `NSWindow` と `NSView` |
| Windows | Win32 | user32 と gdi32 のシステムコールによるトップレベルの `HWND`、`StretchDIBits` |
| Android、`$GW_ANDROID_SOCKET` が設定されている | Android ホスト | [Java ホスト]({{< relref "/surfaces/android.md" >}})へのフレーム化されたプロトコル。ピクセルは共有 memfd に置かれます |
| Android、それ以外 | Wayland または X11 | Termux の下のシェルには、接続できるディスプレイサーバーがあります |
| `js/wasm`、wasmdesk の内部 | wasmbox | [wasmdesk コンポジター]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}})のクライアント |
| `js/wasm`、通常のページ | ブラウザのタブ | `webcanvas` を通したページの `<canvas>`（`Config.Canvas`、デフォルトは `"screen"`）。コンポジター、`SharedArrayBuffer`、クロスオリジン分離は不要です |
| それ以外 | — | `window.ErrUnsupported`。クロスビルドもコンパイルできます |

すべてのバックエンドは `CGO_ENABLED=0` です。X11 のバックエンドは、ソケット越しにゼロから書かれたコアプロトコルで、
Xlib も XCB も使いません。両方のバイトオーダー、Xauthority クッキー、キーシムのマッピング、サーバーのリクエスト
サイズに合わせてタイル分割する `PutImage`、`SCM_RIGHTS` で共有セグメントを渡す MIT-SHM の高速経路に対応しています。
Windows は、`syscall.NewLazyDLL` と `syscall.NewCallback` のウィンドウプロシージャによって、プロセス自身の DLL を
通じて操作されます。macOS は purego による Objective-C ランタイムを通じて操作されます。

各バックエンドのプラットフォームに依存しない部分（イベントのマッピング、座標計算、ピクセルのパッキング、
ダメージ矩形）は、すべてのオペレーティングシステムで 100% カバーされたコーデックにあります。薄いプラットフォーム
固有のグルーコードは、実環境で動作を検証しています。[状況]({{< relref "/status.md" >}})を参照してください。

## 変更された部分だけ {#only-what-changed}

`DamageRenderer` を実装するルート（`toolkit/scene.HostRoot` など）は、再描画した矩形を報告し、GTK4 を除く
すべてのバックエンドは、その矩形だけを表示します。X11 は MIT-SHM の `ShmPutImage`、Wayland は `wl_shm` の
ダメージ、そして Cocoa、Win32、Android、wasmbox、ブラウザのタブが対象です。最初のフレーム、リサイズ、
X11 の `Expose` では、引き続きサーフェス全体が表示されます。

## HiDPI {#hidpi}

デフォルトでは、論理ポイント 1 つにつきフレームバッファのピクセルは 1 つです。UI は読みやすいサイズでレイアウトされ
描画され、コンポジターがそれをアップサンプリングします。Windows では、ウィンドウがモニターごとの DPI 認識を宣言し、
OS が論理フレームを物理的なクライアント領域に合わせて拡大縮小します。`RenderScale` は、パネルの実際の解像度での
フレームバッファを要求しますが、これが正しいのは、デバイスピクセルでレイアウトするルートの場合だけです。

## スクリーン、表示と非表示 {#screens-and-showing-and-hiding}

`window.Screens()` は、接続されているディスプレイを、プライマリを先頭に、デスクトップのパネルを除いた論理ポイントで
列挙し、`Open` の前でも呼び出せます。スクリーンの名前はパネル自身のもの（EDID から得られる `"DELL U2720Q"`）で、
何も公開していない場合はコネクタ名（`"HDMI-1"`）にフォールバックします。接続されている 2 つのパネルが同じ名前を
公開している場合もフォールバックします。2 つを区別できない名前は名前とは言えないからです。

`window.Show`、`Hide`、`Raise` は、開いているウィンドウを閉じずに画面から外し、また戻します。これは
[トレイアプリケーション]({{< relref "/surfaces/application.md#the-tray" >}})が必要とする操作です。

| バックエンド | Show | Hide | Raise |
|---|---|---|---|
| X11 | `MapWindow` | ICCCM の withdraw。タスクバーから消えます | マップ、前面へ、`_NET_ACTIVE_WINDOW` |
| Wayland | 再度コミット | null バッファとコミット | `ErrNotSupported`：xdg-shell にはそのようなリクエストがありません |
| macOS | `orderFront:` | `orderOut:` | アプリをアクティブにし、`makeKeyAndOrderFront:` |
| Windows | `SW_SHOWNA` | `SW_HIDE` | `SW_RESTORE` と `SetForegroundWindow` |
| GTK、Android、wasmbox、ブラウザのタブ | `ErrNotSupported` | `ErrNotSupported` | `ErrNotSupported` |

フォーカスを与えるかどうかはプラットフォームが決めます。フォーカス奪取の防止機能が働いている場合、`Raise` は
ウィンドウに注意を求めるマークを付けるだけになることがあり、その結果を知ることはできません。
