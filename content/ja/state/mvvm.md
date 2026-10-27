---
title: "MVVM：オブザーバブル、コマンド、バインダー"
linkTitle: "MVVM"
weight: 10
description: "go-widgets/mvvm、そのツールキット用とターミナル用のバインダー、mvvmtk のワンコール・ヘルパー、そして元に戻すスタック。"
tags: [state, mvvm, mvvmtk, undo]
---

[`go-widgets/mvvm`](https://github.com/go-widgets/mvvm) はジェネリクスのみで、リフレクションを使いません。
コアパッケージはウィジェットのパッケージを一切インポートしないため、1 つのビューモデルでピクセル描画の
[ツールキット]({{< relref "/rendering/toolkit.md" >}})も
[ターミナルウィジェット]({{< relref "/surfaces/terminal.md" >}})も同じように動かせます。

| プリミティブ | 役割 |
|---|---|
| `Observable[T]` | プロパティ：`Get`、`Set`、`Subscribe`。等しい値を設定しても何も起こりません。 |
| `Command` | アクション。`CanExecute` と `RaiseCanExecuteChanged` を持ちます |
| `ObservableList[T]` | 挿入、削除、置換、移動、リセットをそれぞれ通知するコレクション |

## フォーム {#a-form}

```go
type FormVM struct {
	Name  *mvvm.Observable[string]
	Names *mvvm.ObservableList[string]
	Save  *mvvm.Command
}

vm := &FormVM{Name: mvvm.NewObservable(""), Names: mvvm.NewObservableList[string]()}
vm.Save = mvvm.NewCommand(
	func() { vm.Names.Append(vm.Name.Get()); vm.Name.Set("") },
	func() bool { return vm.Name.Get() != "" }, // CanExecute
)
mvvm.BindCanExecute(vm.Save, vm.Name) // Save greys out while the name is empty
```

## バインディング {#binding}

汎用のバインダーは、**値フィールドへのポインタとコールバックスロットへのポインタ**を通してウィジェットに
到達します：`BindField`（双方向）、`OneWay`、`BindCommand`、`BindList`。ウィジェットはイベントハンドラから
コールバックを呼び出し、フィールドへの直接の書き込みでは何も通知しません。また `Observable.Set` は等しい値を
無視するので、双方向バインディングは再帰せずにエコーできます。

ツールキットは、ウィジェットが自分の状態を**オブザーバブルとして**公開する方向へ移行しつつあります
（`Label.Text()` は `*mvvm.Observable[string]` を返します）。そうなると、初期値を設定するフィールドも、
組み合わせるコールバックもなく、一致させるべき 2 つのプロパティだけが残ります：
`BindTwoWay(src, dst, invalidate)`。

コールバックが複数の引数を取るウィジェットには、バックエンドごとのパッケージに小さな名前付きアダプターが
用意されています。バックエンドをインポートするのは、これらのパッケージだけです。

| パッケージ | アダプター |
|---|---|
| `mvvm/tkbind` | `BindRange`（2 つのハンドルを持つ `RangeSlider`）、`BindContainer` と `BindCardActive`（リストまたはインデックスからコンテナの子要素とカードレイアウトをバインド）、`BindStore` と `BindTable`（[データストア]({{< relref "/state/data.md" >}})の上の `Table`：ソート、グループ化、検証付きのインライン編集） |
| `mvvm/tuibind` | `BindDropdown`、`BindTableSelection` |

## mvvmtk：ウィジェットごとに 1 回の呼び出し {#mvvmtk-one-call-per-widget}

[`go-widgets/mvvmtk`](https://github.com/go-widgets/mvvmtk) は、ツールキットの各ウィジェットの実際のフィールド名と
コールバック名を補うので、ビューがそれらの名前を書く必要はありません。

```go
unbind := mvvmtk.BindText(search, vm.Query, win.Invalidate) // search.Text ⇄ vm.Query
defer unbind()
```

| ヘルパー | 方向 |
|---|---|
| `BindText`（`SearchEntry`）、`BindEntryText`、`BindChecked`、`BindSelectedIndex`、`BindSpin`、`BindListSelection`、`BindViewSwitcher` | 双方向 |
| `BindLabel`、`BindProgress` | ビューモデル → ウィジェット |
| `BindListItems`、`BindDropDownOptions`、`BindViews`、`BindTree` | リスト → ウィジェット |
| `BindCommand` | ボタンのクリック、その `Disabled` 状態、グレーアウトがコマンドに従います |

どのヘルパーも、バインディングを解除し、以前あったコールバックを復元する `unbind` を返します。実行できない
コマンドにバインドされたボタンは、クリックも、Enter や Space も、キーボードフォーカスも受け付けません。

## 元に戻す・やり直し {#undo-and-redo}

`mvvm/undo` は、コアのプリミティブだけで作られたコマンドスタックなので、どのビューの下でも動作します。

```go
s := undo.New() // unlimited, coalescing on; undo.WithLimit(n), undo.WithCoalescing(false)
s.Push(undo.NewCommand("Write hello", do, undoIt))
s.Undo()
s.Redo()

mvvm.BindCommand(s.UndoCommand(), &undoBtn.OnClick, setEnabled)
mvvm.OneWay(s.UndoTextBinding(), &undoBtn.Text, repaint) // "Undo Write hello"
```

連続したキー入力は 1 つのステップにまとめられ、元に戻した後にプッシュすると、やり直しの履歴は破棄されます。
