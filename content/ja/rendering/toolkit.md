---
title: "ツールキット"
linkTitle: "ツールキット"
weight: 20
description: "ペインタ（painter）の継ぎ目を通して描画される 160 種類のウィジェット：テーマ、レイアウト、テキスト、入力、アクセシビリティ。"
tags: [rendering, toolkit, widgets, theme, accessibility]
---

[`go-widgets/toolkit`](https://github.com/go-widgets/toolkit) はウィジェットの
セットです。すべてのウィジェットが同じインターフェース（`Bounds`、`SetBounds`、
`Draw(p painter.Painter, theme *Theme)`、`HitTest`、`OnEvent`）を実装しているため、
ホストはボタンもテーブルもイベントカレンダーも同じように扱えます。標準ライブラリのみを使用し、
`CGO_ENABLED=0` で、ステートメントカバレッジは 100% です。

v0.329.0 では、公開されているウィジェット型が **160 種類**、ウィジェットのコードが約 **69,000 行**あります。
この 2 つの数値はそのタグで測定したもので、以前の値を引き継いだものではありません。ツールキットの README に
測定用のコマンドが載っているので、数値を書き写すのではなく、コマンドを再実行してください。

## ファミリー {#the-families}

| ファミリー | ウィジェット |
|---|---|
| アクション | `Button`, `ToggleButton`, `CheckButton`, `RadioButton` + `RadioGroup`, `Switch`, `SplitButton`, `IconButton`, `CycleButton`, `Chip`, `SegmentedBar` |
| 入力 | `Entry`, `TextView`（選択、IME プレビュー、シンタックススパン、行番号）, `SpinButton`, `Scale`, `RangeSlider`, `SearchEntry`, `TagField`, `ComboBox`, `FormField` と `Validate`/`Rule` |
| 選択 | `ListBox`, `TreeView`, `DropDown` |
| コンテナ | 差し替え可能な `Layout` を持つ `Container`（`FitLayout`, `BoxLayout`, `BorderLayout`, `CardLayout`, `FlowLayout`）、`HBox`, `VBox`, `Grid`, `Frame`, `Dock`, `Border`, `Stack`, `Overlay`, `Paned`, `Splitter`, `Expander`, `Accordion`、宣言的な `Node` ビルダー |
| ナビゲーション | `Menu`, `MenuBar`（`Alt`+文字キー対応）, `ContextMenu`, `Popover`, `CommandPalette`, `Dialog`, `MessageDialog`, `Wizard`, `Notebook`, `ViewSwitcher`, `Carousel` |
| フィードバック | `ProgressBar`, `ProgressCircle`, `LevelBar`, `Spinner`, `Tooltip`, `Notification`, `Toast`, `Banner`, `Alert`, `Badge`, `Skeleton` |
| データ | `Table`（セル編集、列の固定、グループ行、集計、行エキスパンダー）, `TreeTable`, `PropertyGrid`, `Kanban`, `Gantt`, `Agenda`（週、月、四半期、年。複数のカレンダー） |
| チャート | `LineChart`, `BarChart`, `PieChart`, `AreaChart`, `ScatterChart`, `RadarChart`, `Gauge`, `Sparkline` |
| 複合 | `FileChooser`, `ColorChooser`, `ColorPicker`, `FontChooser`, `Calendar`, `DatePicker`, `DateRangePicker`, `TimePicker`, `MarkdownView`, `MarkdownEditor`, `TerminalView` |
| シェル | クライアントサイドの装飾を持つ `Window`, `HeaderBar`, `Toolbar`, `Ribbon`, `Statusbar`, `StatusIcon`, `StatusArea`, `Wallpaper`, `Thumbnail` |

そのすべてを実際に動かして見られます：[ギャラリー](https://go-widgets.github.io/gallery/)。

## リボン {#ribbon}

`Ribbon` は Office 風のコマンドリボンです。ラベル付きのグループをタブにまとめ、
大きなボタン（アイコンがテキストの上）、小さなボタン（アイコンがテキストの横）、トグル、
スプリットボタン、ドロップダウン、小さなギャラリーを置けます。Windows のリボンのガイドラインに従っており、
ウィンドウが狭くなると各グループが優先度順に、フル表示、コンパクト、アイコンのみ、1 つのポップアップボタン、
と段階的に縮小するため、リボンは収まる中で最大の組み合わせを表示します。タブだけに折りたたむこともでき
（Ctrl+F1、またはシェブロン）、そのときはタブを選ぶと本体がのぞき表示されます。Alt を押すとキーチップが表示され、
ツールチップにはショートカットと説明が載ります。タブとツールバーの操作は WAI-ARIA のパターンに従い、
境界線とフォーカスリングはライトテーマでもダークテーマでも 3:1 のコントラストを保ちます。

どのコントロールも `Action` のビューです。`Action` はメニュー、パレット、リボンが共有する 1 つのコマンドで、
`NewCommandAction` はビューモデルの `mvvm.Command` から `Action` を作るため、その `CanExecute` がコントロールを
グレーアウトします。アクティブなタブ、折りたたみ状態、トグル、ドロップダウンの選択は `mvvm.Observable` です。
リボンを `PopoverHost` の下に置けば、そのメニューはホスト側のコードなしでページの上に描画されます。

`RibbonGrid` はコントロールの一種で、そのポップアップは `GridCols` × `GridRows` のセルのグリッドです
（0 のとき 8 × 6、最大 16 × 16）。ワープロの「表」ボタンと同じです。ポインタまたは矢印キーで、角から
その下のセルまでが点灯し、キャプションに「3 × 2」と表示されます。クリックまたは Enter で選択して
`OnGrid(cols, rows)` を呼び出し、Home と End でジャンプし、Escape で閉じます。ライトとダークの両方で描画され、
支援技術にはグリッドが開いている間は展開状態のボタンとして伝えられます。

```go
&toolkit.RibbonControl{Kind: toolkit.RibbonGrid, Label: "Table",
	GridCols: 8, GridRows: 8,
	OnGrid: insertTable}
```

## スプリッター {#splitter}

`Splitter` は N 個のペインを横に（`vertical` が true なら縦に）並べ、隣り合う各ペインの間に、
ドラッグでき、フォーカスも受け取れる分割バーを置きます。WAI-ARIA の window splitter パターンに従っています。
ペインは **固定**（その `Size` は論理ピクセル単位の `mvvm.Observable[int]` で、分割バーがそれに書き込みます）か、
**可変**（固定ペインが残した領域を、同じく `Observable` である `Weight` に応じて分け合います）のどちらかです。
各ペインには `Min` と `Max` があります。`Collapsible` な固定ペインは、分割バーを `Min` の半分より
小さくなるまで引くか、ダブルクリックまたは Enter で閉じ（サイズ 0）、同じ操作でまた開きます。ペインの
`Visible` オブザーバブルで、そのペインと分割バーを非表示にできます。サイズがオブザーバブルに保持されているため、
ビューモデルがバインド、永続化、復元できます。`OnResized` は 1 回のジェスチャーが終わるたびに、ユーザーが変更した
ペインについて呼び出されます。ウィンドウのリサイズにレイアウトが追従しただけの場合は呼び出されません。

```go
files := mvvm.NewObservable(240) // fixed pane, logical px
w := mvvm.NewObservable(1)       // flexible pane, weight

s := toolkit.NewSplitter(false,
	&toolkit.SplitPane{Widget: tree, Name: "Files", Size: files,
		Min: 120, Max: 480, Collapsible: true},
	&toolkit.SplitPane{Widget: editor, Name: "Editor", Weight: w, Min: 200},
)
s.OnResized = func(changed []int) { save(files.Get(), w.Get()) }
```

分割バーはフォーカスを受け取ります。矢印キー（縦の場合は上下）で `Step` 論理ピクセルずつ動かし、Shift を
押すとその 4 倍動きます。Home と End はその前にあるペインを最小または最大にし、Enter は制御対象の固定ペインを
折りたたむか元に戻し、Tab は分割バー間を移動します。各分割バーは `RoleSeparator` で、`aria-valuemin`、
`aria-valuemax`、`aria-valuenow` を持ち、名前は「Resize」とペインの `Name` です。`Cursor(x, y)` は分割バーの
上で `"col-resize"` または `"row-resize"` を返し、`SplitterColors(theme)` は分割線とアクセントの色を返します。
これらは、フォーカスリングと同様、2 つのデフォルトテーマのどちらでも背景とサーフェス（surface）に対して
3:1 のコントラストを保ちます。

## テーマ {#themes}

1 つの `Theme` の値がツリー全体に伝わります。ルートで色を 1 つ変えれば、すべてのウィジェットがその色で再描画されます。
`DefaultLight()` と `DefaultDark()` は組み込みで、`LoadGTKTheme(css)` は GTK 3 や libadwaita の任意のテーマ
（Adwaita、WhiteSur、Solarized など）の `@define-color` ブロックを読み込んで `Theme` にします。

## テキスト {#text}

コンパイル時に組み込まれるデフォルトは 5×7 のビットマップフォントなので、ピクセルテストのジオメトリは変わりません。
1 回の呼び出しで、UI 全体をアンチエイリアス付きのシェーピング済みテキストに切り替えられます。

```go
toolkit.UseOpenTypeText() // Atkinson Hyperlegible, bundled; js/wasm-safe
```

書体は [go-opentype/fonts](https://github.com/go-opentype/fonts) から提供され、
[go-opentype](https://github.com/go-opentype/opentype) が Go でラスタライズします。C のフォントライブラリも
システムのフォント検索も使いません。`NewTrueTypeFont(ttf, px)` は別の書体を読み込み、`NewFallbackFont` は
複数の書体を連結します（たとえば CJK やアラビア文字の書体を追加する場合）。ウィジェット自身の `Font` フィールドは、
そのウィジェットに限ってグローバルな設定を上書きします。

## 入力 {#input}

マウス、キーボード、IME の変換入力、タッチ（`EventTouchStart`/`Move`/`End`。タップ、長押し、スワイプには
`GestureRecognizer`）、ドラッグアンドドロップ（`DragSource`、`DropTarget`）に対応しています。`ListBox`、`Table`、
`TreeView` での複数行の選択は、マウスでもキーボードでも行え、Ctrl に加えて ⌘ にも対応しています。macOS では
Ctrl クリックが副クリックだからです。

## バイナリサイズ：プログラムは使うものだけをリンクする {#binary-size}

ツールキットは 1 つの大きなパッケージですが、リンカにとっては問題ありません。Go のデッドコード除去は、
`main.main` **と、インポートされたすべてのパッケージの init タスク**から到達できるものだけを残します。
プログラムが一度も使わないウィジェットは削除されます。削除されないのは、パッケージレベルの初期化子や、
インポートされたパッケージの `init` が到達するものです。そのため v0.345.0 以降、ツールキットは自身の初期化子から
処理を取り除き、`init` がコードを引き込むパッケージのインポートを避けています。

- `Material` と `Thumbnail` は、`github.com/go-images/images` ではなく、コーデックを含まない
  [`github.com/go-images/images/ops`](https://github.com/go-images/images) を使います。
  `github.com/go-images/images` のデコーダは `init` で自身を登録します（WebAssembly コードで約 1.7 MB）。
- `CollabText`、`NewCollabText`、`MetaName`、`MetaColor` は
  **`github.com/go-widgets/toolkit/collabui`** に移りました。これらは稼働中の `go-crdt/collab` セッションを
  必要とし、そのトランスポートが `net/http` と `crypto/tls` を引き込むためです。`CollabSheet` と
  `IsoCRDTDocument` は `go-crdt/crdt` だけで済むので、`toolkit` に残っています。
- メール用 `Rule` の正規表現、組み込みの iso アイコンレジストリ、`RichEditorToolbar` のボタンテーブルは、
  初期化子ではなく、初めて使われたときに構築されます。

⛔ **この移動は破壊的変更であり、非推奨の転送用定義はありません。** 上記 4 つの名前のインポートを
`collabui` に変更してください。それ以外は何も変わりません。

`GOOS=js GOARCH=wasm` で `-ldflags="-s -w"` を付けたときのサイズです（生のサイズと gzip -9）。

| プログラム | 変更前（生 / gzip） | 変更後（生 / gzip） |
|---|---|---|
| `Label` 1 つ（v0.345.0） | 10.49 / 2.88 MB | 7.48 / 2.13 MB |
| `Label` 1 つ（v0.345.1） | 10.49 / 2.88 MB | 5.66 / 1.61 MB |
| go-wasmcontainers/dashboard | 20.42 / 5.17 MB | 17.72 / 4.58 MB |
| リレーのフローマップ | 22.62 / 5.44 MB | 19.95 / 4.85 MB |

残っているのは主にランタイムと、テキストレンダラーが必要とする OpenType スタックです。自分のプログラムで
シンボルが残っている理由を調べるには、`-ldflags=-dumpdep` でビルドし、そのシンボルから `main.main` または
`go:main.inittasks` まで、最初の親をたどる連鎖を追ってください。

v0.345.0 では `ColorHex` と `ParseColorHex` も追加され、色を 16 進数で書き出し、読み戻せるようになりました。

## アクセシビリティ {#accessibility}

自前でピクセルを描画するツールキットは、描いたものが何かを伝えない限り、スクリーンリーダーに読み上げるものを
与えません。ウィジェットは `Accessible`（ロール、名前、値）を実装し、`CollectA11y` がそれらを集めてホストが
公開できるようにします。Linux では [`window`]({{< relref "/surfaces/native-window.md" >}}) を通じて AT-SPI で、
[Android]({{< relref "/surfaces/android.md#accessibility" >}}) では仮想ビュー階層として公開されます。
