---
title: "Android"
linkTitle: "Android"
weight: 50
description: "go-widgets/android：インターフェース全体を CGO_ENABLED=0 の Go プロセスがレイアウトし描画する、インストール可能な APK。"
tags: [surfaces, android]
---

Android は、CGO を使わないプロセスがウィンドウを所有できない唯一のプラットフォームです。描画可能なサーフェスへの
経路はすべて JNI を通り、JNI には cgo が必要です。X11 や Wayland のように、代わりに話せるワイヤプロトコルも
ありません。

そのため [`go-widgets/android`](https://github.com/go-widgets/android) は 2 つのプロセスで構成されます。

| | |
|---|---|
| **Java ホスト**（`host/`） | `Activity`、`SurfaceView`、タッチ、キー、ライフサイクルを所有します。ピクセルを転送するだけで、ウィジェットについては何も知りません。 |
| **Go アプリケーション**（`cmd/gwapp`） | 通常の `CGO_ENABLED=0 GOOS=android` の実行ファイルです。レイアウト、ウィジェット、テーマ、ヒットテスト、フォーカスを所有し、他のすべてのバックエンドと変わりません。 |

Java ホストは、Linux で X サーバーがいる位置に立ちます。ピクセルは、アプリケーションが作成してソケット経由で
ホストに渡す **memfd** を通って届きます。Go 側は RGBA_8888 を書き込み、これは Android の ARGB_8888 と
バイト単位で一致するので、転送は変換なしのコピーで、ダメージを受けた矩形だけが対象です。入力、インセット、
IME のテキスト、アクセシビリティツリーは、抽象的な `LocalSocket` 上のフレーム化されたプロトコルで受け渡されます。

```go
c, err := android.Dial("my app", nil) // nil theme: toolkit.DefaultDark()
if errors.Is(err, android.ErrUnsupported) {
	return nil // not under a host
}
defer c.Close()
return c.Run(myWidgetTree())
```

`Client` は `window` の `Backend` を満たすので、アプリケーションは、ウィンドウより上の行を 1 行も変えずに、
Android とデスクトップのバックエンドの間を移れます。ホストのソケットがあれば、
[`window.Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}}) が自動的にそれを選びます。

## arm64 のみ {#arm64-only}

`android/arm64` は、Go が cgo なしでリンクできる唯一の Android ターゲットです。`arm`、`amd64`、`386` には
外部リンクが必要です。CI は両方の側面、つまりビルドできる 1 つとできない 3 つを検証するので、Go がこの制限を
解除した日には、ビルドがそれを知らせてくれます。

## アクセシビリティ {#accessibility}

スクリーンリーダーには、`SurfaceView` は 1 つの不透明な矩形にしか見えません。そこでホストは代わりに仮想ビュー
階層を提供します。アクセシブルなウィジェットごとに 1 つのノードがあり、Android が読み上げ内容を選ぶ基準となる
`android.widget.*` クラス、テキスト、境界、そしてアクティベーションアクションを持ちます。ツリーは、何かが読み取る
ときに**プル**され、プッシュされることはないので、アクセシビリティサービスが接続されていないアプリでは、
ツリーが構築されることはありません。アクティベーションは、要素の中心への通常のクリックとして、タッチと同じ
コードを通って戻ってきます。

## タッチとアニメーション {#touch-and-animation}

各タッチサンプルは、タッチイベントとしてツリーに届き、続いてマウスイベントとしても届きます。前者はジェスチャーに
対応するウィジェット向け、後者はクリックだけを待つ多くのウィジェット向けです。アニメーションするウィジェットは
フレームループで進みます。このループは何かがアニメーションを始めたときに開始し、何もアニメーションしなくなったときに
停止します。
