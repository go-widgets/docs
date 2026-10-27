---
title: "MVVM: observable, 명령, 바인더"
linkTitle: "MVVM"
weight: 10
description: "go-widgets/mvvm, 그 툴킷 및 터미널 바인더, mvvmtk의 한 번 호출 헬퍼, 그리고 실행 취소 스택."
tags: [state, mvvm, mvvmtk, undo]
---

[`go-widgets/mvvm`](https://github.com/go-widgets/mvvm)은 제네릭만 사용하며
리플렉션을 쓰지 않습니다. 핵심 패키지는 위젯 패키지를 전혀 가져오지 않으므로, 하나의
뷰 모델이 픽셀 [툴킷]({{< relref "/rendering/toolkit.md" >}})과
[터미널 위젯]({{< relref "/surfaces/terminal.md" >}})을 똑같이 구동합니다.

| 기본 요소 | 역할 |
|---|---|
| `Observable[T]` | 속성: `Get`, `Set`, `Subscribe`. 같은 값을 설정하면 아무 일도 일어나지 않습니다. |
| `Command` | `CanExecute`와 `RaiseCanExecuteChanged`가 있는 동작 |
| `ObservableList[T]` | 삽입, 제거, 교체, 이동, 초기화를 각각 알리는 컬렉션 |

## 폼 {#a-form}

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

## 바인딩 {#binding}

제네릭 바인더는 **값 필드에 대한 포인터와 콜백 슬롯에 대한 포인터**를 통해 위젯에
접근합니다. `BindField`(양방향), `OneWay`, `BindCommand`, `BindList`가 있습니다.
위젯은 이벤트 핸들러에서 콜백을 호출하고 필드를 직접 쓰는 것에는 반응하지 않으며,
`Observable.Set`은 같은 값을 무시하므로 양방향 바인딩은 재귀 없이 되울릴 수
있습니다.

툴킷은 위젯이 자신의 상태를 observable **로** 노출하도록 옮겨 가는 중입니다
(`Label.Text()`는 `*mvvm.Observable[string]`을 반환합니다). 그러면 초기화할 필드도
조합할 콜백도 없고, 서로 일치해야 하는 두 속성만 남습니다: `BindTwoWay(src, dst,
invalidate)`.

콜백이 여러 인자를 받는 위젯에는 백엔드별 패키지에 작은 이름 붙은 어댑터가
제공됩니다. 백엔드를 가져오는 패키지는 이들뿐입니다.

| 패키지 | 어댑터 |
|---|---|
| `mvvm/tkbind` | `BindRange` (the two-handle `RangeSlider`), `BindContainer` and `BindCardActive` (a container's children and a card layout from a list or an index), `BindStore` and `BindTable` (a `Table` over a [data store]({{< relref "/state/data.md" >}}): sorting, grouping and validated inline edits) |
| `mvvm/tuibind` | `BindDropdown`, `BindTableSelection` |

## mvvmtk: 위젯당 한 번의 호출 {#mvvmtk-one-call-per-widget}

[`go-widgets/mvvmtk`](https://github.com/go-widgets/mvvmtk)는 각 툴킷 위젯의 실제
필드 이름과 콜백 이름을 채워 주므로, 뷰는 그 이름을 직접 쓰지 않습니다.

```go
unbind := mvvmtk.BindText(search, vm.Query, win.Invalidate) // search.Text ⇄ vm.Query
defer unbind()
```

| 헬퍼 | 방향 |
|---|---|
| `BindText` (`SearchEntry`), `BindEntryText`, `BindChecked`, `BindSelectedIndex`, `BindSpin`, `BindListSelection`, `BindViewSwitcher` | 양방향 |
| `BindLabel`, `BindProgress` | 뷰 모델 → 위젯 |
| `BindListItems`, `BindDropDownOptions`, `BindViews`, `BindTree` | 리스트 → 위젯 |
| `BindCommand` | 버튼의 클릭, `Disabled` 상태, 비활성 표시가 명령을 따릅니다 |

모든 헬퍼는 바인딩을 해제하고 이전에 있던 콜백을 복원하는 `unbind`를 반환합니다.
실행할 수 없는 명령에 바인딩된 버튼은 클릭도, Enter나 Space도, 키보드 포커스도
받지 않습니다.

## 실행 취소와 다시 실행 {#undo-and-redo}

`mvvm/undo`는 핵심 기본 요소만으로 만든 명령 스택이므로 어떤 뷰 아래에서도
동작합니다.

```go
s := undo.New() // unlimited, coalescing on; undo.WithLimit(n), undo.WithCoalescing(false)
s.Push(undo.NewCommand("Write hello", do, undoIt))
s.Undo()
s.Redo()

mvvm.BindCommand(s.UndoCommand(), &undoBtn.OnClick, setEnabled)
mvvm.OneWay(s.UndoTextBinding(), &undoBtn.Text, repaint) // "Undo Write hello"
```

연속된 키 입력은 하나의 단계로 합쳐지고, 실행 취소 뒤에 푸시하면 다시 실행 꼬리가
버려집니다.
