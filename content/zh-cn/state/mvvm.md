---
title: "MVVM：可观察对象、命令、绑定器"
linkTitle: "MVVM"
weight: 10
description: "go-widgets/mvvm、它的工具包和终端绑定器、mvvmtk 的一次调用辅助函数，以及撤销栈。"
tags: [state, mvvm, mvvmtk, undo]
---

[`go-widgets/mvvm`](https://github.com/go-widgets/mvvm) 只用泛型，不用反射。
它的核心包完全不导入任何控件包，因此同一个视图模型既能驱动像素
[工具包]({{< relref "/rendering/toolkit.md" >}})，也能驱动
[终端控件]({{< relref "/surfaces/terminal.md" >}})。

| 原语 | 作用 |
|---|---|
| `Observable[T]` | 一个属性：`Get`、`Set`、`Subscribe`。设置相等的值不会有任何效果。 |
| `Command` | 一个操作，带有 `CanExecute` 和 `RaiseCanExecuteChanged` |
| `ObservableList[T]` | 一个集合，会报告每一次插入、删除、替换、移动和重置 |

## 一个表单 {#a-form}

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

## 绑定 {#binding}

通用绑定器通过**指向控件值字段的指针和指向其回调槽的指针**来访问控件：
`BindField`（双向）、`OneWay`、`BindCommand`、`BindList`。控件在事件处理函数中触发回调，
而直接写字段则不会触发任何通知；`Observable.Set` 又会忽略相等的值——因此双向绑定可以回响而不会递归。

工具包正在改为让控件把状态**作为**可观察对象公开（`Label.Text()` 返回一个
`*mvvm.Observable[string]`）。这样就没有需要初始化的字段，也没有需要组合的回调，
只剩两个必须保持一致的属性：`BindTwoWay(src, dst, invalidate)`。

回调带有多个参数的控件，会在各个后端专属的包中得到一个具名的小型适配器，
只有这些包才会导入后端：

| 包 | 适配器 |
|---|---|
| `mvvm/tkbind` | `BindRange`（双手柄的 `RangeSlider`）、`BindContainer` 和 `BindCardActive`（由列表或索引驱动容器的子项和卡片布局）、`BindStore` 和 `BindTable`（让 `Table` 建立在[数据存储]({{< relref "/state/data.md" >}})之上：排序、分组和经过校验的行内编辑） |
| `mvvm/tuibind` | `BindDropdown`、`BindTableSelection` |

## mvvmtk：每个控件一次调用 {#mvvmtk-one-call-per-widget}

[`go-widgets/mvvmtk`](https://github.com/go-widgets/mvvmtk) 填入每个工具包控件真实的字段名和回调名，
因此视图从不需要写出它们：

```go
unbind := mvvmtk.BindText(search, vm.Query, win.Invalidate) // search.Text ⇄ vm.Query
defer unbind()
```

| 辅助函数 | 方向 |
|---|---|
| `BindText`（`SearchEntry`）、`BindEntryText`、`BindChecked`、`BindSelectedIndex`、`BindSpin`、`BindListSelection`、`BindViewSwitcher` | 双向 |
| `BindLabel`、`BindProgress` | 视图模型 → 控件 |
| `BindListItems`、`BindDropDownOptions`、`BindViews`、`BindTree` | 列表 → 控件 |
| `BindCommand` | 按钮的点击、其 `Disabled` 状态及其变灰都跟随该命令 |

每个辅助函数都会返回一个 `unbind`，用来解除绑定并恢复此前存在的回调。绑定到无法运行的命令上的按钮，
不接受点击，不接受 Enter 或空格，也不能获得键盘焦点。

## 撤销与重做 {#undo-and-redo}

`mvvm/undo` 是仅基于核心原语构建的命令栈，因此在任何视图下都能工作：

```go
s := undo.New() // unlimited, coalescing on; undo.WithLimit(n), undo.WithCoalescing(false)
s.Push(undo.NewCommand("Write hello", do, undoIt))
s.Undo()
s.Redo()

mvvm.BindCommand(s.UndoCommand(), &undoBtn.OnClick, setEnabled)
mvvm.OneWay(s.UndoTextBinding(), &undoBtn.Text, repaint) // "Undo Write hello"
```

连续的按键会合并为一步，在撤销之后再推入新命令会丢弃重做尾部。
