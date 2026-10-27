---
title: "ターミナル"
linkTitle: "ターミナル"
weight: 40
description: "go-widgets/tui は、セルペインタを通してウィジェットを描画します。スナップショットレンダラー、対話型ランナー、文字グリッド向けに作られたウィジェットを備えています。"
tags: [surfaces, tui, terminal]
---

[`go-widgets/tui`](https://github.com/go-widgets/tui) は、
[セルペインタ]({{< relref "/rendering/painter.md" >}})を通して、ウィジェットツリーをターミナルのフレームに
変換します。標準ライブラリのみで、`x/term` も cgo も使いません。

## 1 フレーム {#one-frame}

```go
_ = tui.RenderOnce(os.Stdout, widgets, nil) // nil theme: light
```

`RenderOnceSized(w, cols, rows, widgets, theme)` はサイズを明示的に受け取ります。`RenderOnce` は `COLUMNS` と
`LINES` を読み取り、なければ 80×24 にフォールバックします。`TIOCGWINSZ` を呼ばず、環境変数を読むことで、
パッケージにプラットフォームごとのコードが入らずに済みます。

## 対話型プログラム {#an-interactive-program}

```go
app := tui.NewApp()
app.Root = buildScene()
app.Keys["q"] = func(a *tui.App) { a.Quit() }
app.Keys["Ctrl+C"] = func(a *tui.App) { a.Quit() }
os.Exit(app.Run())
```

`Run` は代替スクリーンと raw モードに入り、入力を `Root` に振り分け（グローバルなキーハンドラが先）、`SIGWINCH` に
追従し、パニックのときも含め、必ずターミナルを元の状態に戻します。キーハンドラは `Consume` を呼ぶと、そのイベントを
`Root` に渡さずに済ませられます。`InputTarget` は、消費されなかったすべてのイベントを 1 つのウィジェットに送ります。
コマンドパレットが、開いている間の入力を受け止めるのはこの仕組みです。`NewFocusRing` は、フォームに Tab と
Shift+Tab での移動を与えます。

## セル向けにサイズを決めたウィジェット {#widgets-sized-for-cells}

ツールキットのほとんどのウィジェットはピクセル単位で設計されています。そのパディングの定数は、セルペインタの下では
セル数として数えられ、膨らんでしまいます。そこで `tui` には独自の**セルネイティブ**なセットがあります。
`TextEditor`、`Entry`、`Button`、`CheckButton`、`RadioButton`、`Scale`、`ProgressBar`、`Sparkline`、`ListBox`、
`TreeView`、`Table`、`MenuBar`、`Notebook`、`HSplit`、`VSplit`、`Dialog`、`Dropdown`、`Spinner` などで、
1 つのグリフが 1 つのセルになります。それぞれが依然として `toolkit.Widget` なので、同じインスタンスをピクセルにも
描画できます。

`TextEditor` は読み書き可能なコードエディタです。ガター、元に戻す・やり直し、検索と置換、選択、行の移動、
そして Go（`go/scanner` を使用）、JavaScript と TypeScript、Python、Ruby、シェル、C と C++、Rust、JSON、YAML、
HCL、TOML、LaTeX、Markdown のシンタックスハイライトを備え、外部依存はありません。

```sh
go run ./cmd/tui-widgets | less -R     # every cell-native widget, one per slot
go run ./cmd/tui-widget-explorer       # the same, interactive
go run ./cmd/tui-explorer              # a file browser
go run ./cmd/tui-editor --file=x.go    # a modal editor
```

2 つのデモプログラムは、実際の pty でエンド・ツー・エンドにテストされています。本物のキーのバイト列を入力し、
描画されたフレームを検証します。
