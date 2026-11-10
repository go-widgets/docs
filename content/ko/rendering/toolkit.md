---
title: "툴킷"
linkTitle: "툴킷"
weight: 20
description: "페인터 경계를 통해 그려지는 160개의 위젯 타입: 테마, 레이아웃, 텍스트, 입력, 접근성."
tags: [rendering, toolkit, widgets, theme, accessibility]
---

[`go-widgets/toolkit`](https://github.com/go-widgets/toolkit)은 위젯 모음
(widget set)입니다. 모든 위젯은 같은 인터페이스인 `Bounds`, `SetBounds`,
`Draw(p painter.Painter, theme *Theme)`, `HitTest`, `OnEvent`를 구현하므로, 호스트는
버튼, 표, 이벤트 캘린더를 똑같이 다룹니다. 표준 라이브러리만 사용하며,
`CGO_ENABLED=0`이고, 구문 커버리지는 100%입니다.

v0.329.0 기준으로 위젯 코드 약 **69,000줄**에 **공개된 위젯 타입이 160개**
있습니다. 이 두 숫자는 해당 태그에서 측정한 값이며 이전 값을 그대로 가져온 것이
아닙니다. 툴킷의 README에 측정 명령이 있으니, 수치를 되풀이하지 말고 명령을 다시
실행해야 합니다.

## 계열 (families) {#the-families}

| 계열 | 위젯 |
|---|---|
| Action | `Button`, `ToggleButton`, `CheckButton`, `RadioButton` + `RadioGroup`, `Switch`, `SplitButton`, `IconButton`, `CycleButton`, `Chip`, `SegmentedBar` |
| Input | `Entry`, `TextView` (selection, IME preview, syntax spans, line numbers), `SpinButton`, `Scale`, `RangeSlider`, `SearchEntry`, `TagField`, `ComboBox`, `FormField` with `Validate`/`Rule` |
| Selection | `ListBox`, `TreeView`, `DropDown` |
| Containers | `Container` with a swappable `Layout` (`FitLayout`, `BoxLayout`, `BorderLayout`, `CardLayout`, `FlowLayout`); `HBox`, `VBox`, `Grid`, `Frame`, `Dock`, `Border`, `Stack`, `Overlay`, `Paned`, `Splitter`, `Expander`, `Accordion`; a declarative `Node` builder |
| Navigation | `Menu`, `MenuBar` (with `Alt`+letter), `ContextMenu`, `Popover`, `CommandPalette`, `Dialog`, `MessageDialog`, `Wizard`, `Notebook`, `ViewSwitcher`, `Carousel` |
| Feedback | `ProgressBar`, `ProgressCircle`, `LevelBar`, `Spinner`, `Tooltip`, `Notification`, `Toast`, `Banner`, `Alert`, `Badge`, `Skeleton` |
| Data | `Table` (cell editing, frozen columns, group rows, aggregates, row expanders), `TreeTable`, `PropertyGrid`, `Kanban`, `Gantt`, `Agenda` (week, month, quarter, year; several calendars) |
| Charts | `LineChart`, `BarChart`, `PieChart`, `AreaChart`, `ScatterChart`, `RadarChart`, `Gauge`, `Sparkline` |
| Composite | `FileChooser`, `FileTree`, `ColorChooser`, `ColorPicker`, `FontChooser`, `Calendar`, `DatePicker`, `DateRangePicker`, `TimePicker`, `MarkdownView`, `MarkdownEditor`, `TerminalView` |
| Shell | `Window` with client-side decorations, `HeaderBar`, `Toolbar`, `Ribbon`, `Statusbar`, `StatusIcon`, `StatusArea`, `Wallpaper`, `Thumbnail` |

모든 위젯을 실시간으로 볼 수 있습니다: [갤러리](https://go-widgets.github.io/gallery/).

## 리본 (Ribbon) {#ribbon}

`Ribbon`은 Office 스타일의 명령 리본입니다. 이름이 붙은 그룹들이 탭으로 나뉘어 있고,
그 안에 큰 버튼(텍스트 위에 아이콘), 작은 버튼(텍스트 옆에 아이콘), 토글, 분할 버튼,
드롭다운, 작은 갤러리가 들어갑니다. Windows 리본 지침을 따릅니다. 창이 좁아지면
각 그룹이 우선순위에 따라 단계적으로 줄어들어(전체, 압축, 아이콘만, 그리고 하나의
팝업 버튼) 리본은 들어갈 수 있는 가장 큰 조합을 보여 줍니다. 리본은 탭만 남기도록
접을 수 있고(Ctrl+F1 또는 갈매기표), 그러면 탭이 본문을 잠깐 엿보여 줍니다. Alt를
누르면 키 팁이 표시되고, 툴팁에는 단축키와 설명이 담깁니다. 탭과 툴바의 탐색은
WAI-ARIA 패턴을 따르며, 경계선과 포커스 링은 밝은 테마와 어두운 테마 모두에서 3:1
대비를 유지합니다.

모든 컨트롤은 `Action`에 대한 뷰입니다. `Action`은 메뉴, 팔레트, 리본이 함께 쓰는
하나의 명령이며, `NewCommandAction`은 뷰 모델(view-model)의 `mvvm.Command`로부터
하나를 만들기 때문에 `CanExecute`가 컨트롤을 비활성(회색) 상태로 만듭니다. 활성
탭, 접힌 상태, 토글, 드롭다운의 선택은 `mvvm.Observable`입니다. 리본을
`PopoverHost` 아래에 두면 호스트 코드 없이도 리본의 메뉴가 페이지 위에 그려집니다.

`RibbonGrid`는 팝업이 `GridCols` × `GridRows` 셀 격자인 컨트롤 종류로(0이면 8 × 6,
최대 16 × 16), 워드 프로세서의 표 버튼과 같습니다. 포인터나 화살표 키를 움직이면
모서리에서 그 아래의 셀까지 셀이 밝아지고, 캡션에 "3 × 2"가 표시됩니다. 클릭하거나
Enter를 누르면 선택되어 `OnGrid(cols, rows)`가 호출되고, Home과 End는 건너뛰며,
Escape는 닫습니다. 밝은 테마와 어두운 테마 모두에서 그려지며, 격자가 열려 있는
동안에는 확장된 버튼으로 보조 기술에 보고됩니다.

```go
&toolkit.RibbonControl{Kind: toolkit.RibbonGrid, Label: "Table",
	GridCols: 8, GridRows: 8,
	OnGrid: insertTable}
```

## 스플리터 (Splitter) {#splitter}

`Splitter`는 N개의 패널을 나란히(또는 `vertical`이 true이면 위아래로) 배치하고,
이웃한 각 쌍 사이에 드래그할 수 있고 포커스를 받을 수 있는 구분선을 둡니다.
WAI-ARIA 창 스플리터(window-splitter) 패턴을 따릅니다. 패널은 **고정**(fixed)이거나
**유연**(flexible)합니다. 고정 패널의 `Size`는 논리 픽셀 단위의
`mvvm.Observable[int]`이며 구분선이 값을 씁니다. 유연 패널은 고정 패널이 남긴
공간을 역시 `Observable`인 `Weight`에 따라 나눠 가집니다. 모든 패널에는 `Min`과
`Max`가 있습니다. `Collapsible` 고정 패널은 구분선을 `Min`의 절반 아래로 끌거나
더블클릭 또는 Enter를 누르면 닫히고(크기 0), 같은 방법으로 다시 열립니다. 패널의
`Visible` observable은 패널과 그 구분선을 켜고 끕니다. 크기가 observable에 있으므로
뷰 모델이 이를 바인딩하고, 저장하고, 복원할 수 있습니다. `OnResized`는 사용자가
끝낸 제스처마다, 사용자가 바꾼 패널들을 인자로 한 번 호출되며, 창 크기 조정을 따라
레이아웃이 바뀐 경우에는 호출되지 않습니다.

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

구분선은 포커스를 받습니다. 화살표 키(세로일 때는 위와 아래)는 구분선을 `Step`
논리 픽셀씩 움직이고, Shift를 누르면 그 네 배만큼 움직입니다. Home과 End는 앞쪽
패널을 최소 크기와 최대 크기로 만들고, Enter는 구분선이 제어하는 고정 패널을 접거나
복원하며, Tab은 구분선 사이를 이동합니다. 각 구분선은 `aria-valuemin`,
`aria-valuemax`, `aria-valuenow`를 가진 `RoleSeparator`이며, 이름은 "Resize"와
패널의 `Name`입니다. `Cursor(x, y)`는 구분선 위에서 `"col-resize"` 또는
`"row-resize"`를 반환하고, `SplitterColors(theme)`는 구분선 선과 강조색을 주는데,
이 색은 포커스 링과 마찬가지로 두 기본 테마 모두에서 배경 및 표면과 3:1 대비를
유지합니다.

## 테마 (Themes) {#themes}

하나의 `Theme` 값이 트리 전체로 전파됩니다. 루트에서 색을 바꾸면 모든 위젯이 그
색으로 다시 그려집니다. `DefaultLight()`와 `DefaultDark()`가 내장되어 있고,
`LoadGTKTheme(css)`는 Adwaita, WhiteSur, Solarized 같은 GTK 3 또는 libadwaita
테마의 `@define-color` 블록을 읽어 `Theme`으로 만듭니다.

## 텍스트 (Text) {#text}

컴파일되어 들어 있는 기본 글꼴은 5×7 비트맵 글꼴이므로 픽셀 테스트가 기하 구조를
그대로 유지합니다. 한 번의 호출로 UI 전체를 안티앨리어싱되고 셰이핑(shaping)된
텍스트로 바꿀 수 있습니다.

```go
toolkit.UseOpenTypeText() // Atkinson Hyperlegible, bundled; js/wasm-safe
```

글꼴은 [go-opentype/fonts](https://github.com/go-opentype/fonts)에서 가져오며
[go-opentype](https://github.com/go-opentype/opentype)가 Go로 래스터화합니다. C
글꼴 라이브러리도, 시스템 글꼴 조회도 사용하지 않습니다.
`NewTrueTypeFont(ttf, px)`는 다른 글꼴을 불러오고, `NewFallbackFont`는 여러 글꼴을
연결하여 예를 들어 CJK나 아랍어 글꼴을 추가할 수 있으며, 위젯 자체의 `Font` 필드는
그 위젯에 한해서 전역 글꼴을 덮어씁니다.

## 입력 (Input) {#input}

마우스, 키보드, IME 조합, 터치(`EventTouchStart`/`Move`/`End`, 탭, 길게 누르기,
스와이프를 위한 `GestureRecognizer`), 드래그 앤 드롭(`DragSource`, `DropTarget`)을
지원합니다. `ListBox`, `Table`, `TreeView`의 다중 행 선택은 마우스와 키보드 모두로
동작하며, macOS에서는 Ctrl-클릭이 보조 클릭이므로 Ctrl뿐 아니라 ⌘도 인식합니다.

## 한 버퍼를 여러 사람이 (Collaboration) {#collaboration}

`collabui.CollabText`는 `CodeEditor`를 [`go-crdt/collab`](https://github.com/go-crdt/collab)
세션의 텍스트 파트 하나에 연결합니다. 병합은 CRDT의 몫이며 이 바인딩은 거기에
아무것도 더하지 않습니다. 편집이 **어디에** 적용되는지는 바인딩의 몫이고, 그것은
다른 질문입니다.

피어의 편집은 회선에서 도착하는 즉시 레플리카에 병합됩니다. 에디터는 그 뒤에,
호스트가 바인딩이 건넨 클로저를 실행할 때 다시 그려집니다. 그 사이에 화면은 한
텍스트를 보여주고 레플리카는 다른 텍스트를 들고 있으며, 그 창 안에서의 타건은
화면을 겨냥한 것이었습니다. 원래 오프셋 그대로 적용하면 다른 곳에 떨어지고, 게다가
모든 레플리카가 그 결과에 동의합니다. 그래서 결함처럼 보이지 않았습니다. 결함이었습니다.

| 사람이 한 일 | 이전 | 지금 |
|---|---|---|
| 화면의 `abc` 뒤에 `Y`를 입력, `Xabc`가 이미 병합됨 | `XabYc` | `XabcY` |
| 화면의 `bc`를 삭제, 그 사이에 피어의 `X` | `acd` — 피어의 `X`가 파괴됨 | `aXd` |

이제 로컬 편집은 적용되기 전에 그 간극을 건너 옮겨집니다. Ellis & Gibbs(SIGMOD '89)의
포함 변환이며, Jupiter(Nichols 외, UIST '95)가 변환하는 쪽과 같은 쪽입니다 — 이미
병합된 것에 대해 자신의 편집을 변환합니다. 오프셋은 자기보다 먼저 떨어진 것과 함께
움직이고, 그것을 대체한 것 안으로 같은 거리만큼 들어갑니다. 삭제는 그 사람이 볼 수
있었던 문자를 가져가고 볼 수 없었던 문자는 가져가지 않습니다. 삭제 범위 안에 있는
피어의 삽입은 그 *주위*가 삭제되어 살아남습니다. 위치는 추측이지만, 조금 전까지
누군가의 화면에 있던 문자는 추측이 아닙니다.

### 버전 관리되는 작업 트리 위의 FileTree {#filetree-vcs}

v0.363.0부터 `FileTree.SetRepository(vcs.Repository)`가 [`vcs`](https://github.com/go-widgets/vcs) 저장소에서 `M`, `A`, `D`, `?`, `!` 표시를 채웁니다. 호스트가 각 `Status`를 직접 설정할 필요가 없습니다. 툴킷은 `vcs`(표준 라이브러리만 사용)만 import하고 드라이버는 절대 import하지 않으며, 드라이버는 호스트가 등록합니다. git 드라이버는 별도 모듈인 `vcs/git`(go-git 사용, `git` 바이너리는 실행하지 않음)이고, `vcs.Detect(dir)`는 가장 가까운 저장소까지 상위 디렉터리를 거슬러 올라갑니다.

```go
import (
    "context"

    "github.com/go-widgets/toolkit"
    "github.com/go-widgets/vcs"
    _ "github.com/go-widgets/vcs/git" // registers the "git" driver
)

// newTree shows dir's files with their git state. post hands a closure to the
// host's UI goroutine.
func newTree(dir string, entries []toolkit.FileEntry, post func(func())) (*toolkit.FileTree, error) {
    repo, err := vcs.Detect(dir) // walks up to the nearest repository
    if err != nil {
        return nil, err // vcs.ErrNoRepository when there is none
    }
    tree := toolkit.NewFileTree(entries)
    tree.PostUI = post // Status runs off the UI goroutine
    tree.SetRepository(repo)
    tree.StatusError().Subscribe(func(err error) { /* show it */ })
    return tree, nil
}

// After a change the tree was not told of, say a save:
func saved(tree *toolkit.FileTree) { tree.RefreshStatusAsync() }

// A refresh you can wait for, on a worker or in a test:
func wait(tree *toolkit.FileTree) error { return tree.RefreshStatus(context.Background()) }
```

저장소 상태가 표시가 되는 방식:

- `Modified`, `Added`, `Deleted`, `Untracked`, `Conflicted`는 이름이 같은 `FileStatus`에 대응합니다. `Renamed`는 새 경로의 `FileAdded`가 됩니다. `Ignored`는 표시가 없습니다(`FileIgnored`는 아무것도 그리지 않기 때문입니다).
- **폴더는 그 아래 무언가에 표시가 있으면 `FileModified`, 그 아래 무언가가 충돌 상태이면 `FileConflicted`입니다**(충돌이 우선). 앞 문단이 추측하기를 거부한 바로 그 정책이며, 저장소를 연결하는 것이 그 정책을 받아들이는 것입니다. 저장소가 직접 보고하는 폴더(서브모듈)는 그 아래에 충돌이 없는 한 자기 상태를 유지합니다. 호스트가 항목에 설정한 `Status`는 저장소의 것보다 우선합니다.
- 삭제된 파일은 호스트가 여전히 목록에 넣을 때만 표시됩니다. 트리는 행을 만들어 내지 않습니다.
- `Marks()`는 폴더를 포함한 모든 표시의 `Observable`로, 범례나 "변경 3개" 배지에 쓸 수 있습니다. `StatusError()`는 마지막 갱신의 오류이며, 이때는 이전 표시가 유지됩니다.

트리는 생성, 이름 변경, 삭제, 이동, 가져오기를 스스로 적용하지 않습니다. 호스트가 적용하고 `Entries`를 설정하면 그 `Set`이 표시를 갱신합니다. `RefreshStatus(ctx)`는 지금 묻고, 표시가 적용되면 돌아옵니다.

**`PostUI`가 필요한 이유**(v0.364.0). `Repository.Status`는 큰 저장소에서 몇 초가 걸릴 수 있고(go-git은 상태 전체를 메모리에 만들며 컨텍스트를 받지 않습니다), UI goroutine에서 실행하면 안 됩니다. 브라우저에서는 페이지의 이벤트 루프를 막게 됩니다. `PostUI`를 설정하면 트리가 시작하는 모든 갱신(`SetRepository`, `Entries` 변경, `RefreshStatusAsync`)이 별도 goroutine에서 `Status`를 실행하고, 결과를 **호스트가 UI goroutine에서 실행하는** 클로저로 돌려줍니다. `collabui.CollabText`와 같은 약속입니다. 더 새로운 갱신은 진행 중인 것을 취소하고 늦게 도착한 응답은 아무 효과가 없습니다. `PostUI`가 없으면 모두 동기이며, 테스트나 작은 트리에는 알맞습니다.

## 바이너리 크기: 프로그램은 사용하는 것만 링크합니다 {#binary-size}

툴킷은 하나의 큰 패키지이지만 링커에게는 문제가 되지 않습니다. Go의 데드 코드 제거는
`main.main`에서, **그리고 가져온 모든 패키지의 init 작업에서** 도달할 수 있는 것만
남깁니다. 프로그램이 건드리지 않는 위젯은 제거됩니다. 제거되지 않는 것은 패키지 수준
초기화 식이나 가져온 패키지의 `init`이 도달하는 모든 것이므로, v0.345.0부터 툴킷은
자체 초기화 식에서 작업을 하지 않도록 비워 두고, `init`이 코드를 끌고 들어오는
패키지를 가져오지 않습니다.

- `Material`과 `Thumbnail`은 `init`에서 디코더가 스스로를 등록하는
  `github.com/go-images/images`(WebAssembly 코드 약 1.7 MB) 대신 코덱이 없는
  [`github.com/go-images/images/ops`](https://github.com/go-images/images)를
  사용합니다.
- `CollabText`, `NewCollabText`, `MetaName`, `MetaColor`는
  **`github.com/go-widgets/toolkit/collabui`**로 옮겨졌습니다. 이들은 실시간
  `go-crdt/collab` 세션이 필요하고, 그 전송 계층이 `net/http`와 `crypto/tls`를
  끌어오기 때문입니다. `CollabSheet`와 `IsoCRDTDocument`는 `go-crdt/crdt`만
  필요하므로 `toolkit`에 남아 있습니다.
- 이메일 `Rule`의 정규식, 내장 iso 아이콘 레지스트리, `RichEditorToolbar` 버튼 표는
  초기화 식이 아니라 처음 사용될 때 만들어집니다.

⛔ **이 이동은 호환성을 깨는 변경이며, 폐기 예정(deprecated) 전달 함수는 없습니다.**
위의 네 이름의 import를 `collabui`로 바꾸십시오. 그 외에는 바뀌는 것이 없습니다.

`GOOS=js GOARCH=wasm`과 `-ldflags="-s -w"`로 빌드한 크기이며, 원본 크기와 gzip -9
크기입니다.

| 프로그램 | 이전 (raw / gzip) | 이후 (raw / gzip) |
|---|---|---|
| one `Label` (v0.345.0) | 10.49 / 2.88 MB | 7.48 / 2.13 MB |
| one `Label` (v0.345.1) | 10.49 / 2.88 MB | 5.66 / 1.61 MB |
| go-wasmcontainers/dashboard | 20.42 / 5.17 MB | 17.72 / 4.58 MB |
| the relay flow map | 22.62 / 5.44 MB | 19.95 / 4.85 MB |

남은 크기의 대부분은 런타임과 텍스트 렌더러에 필요한 OpenType 스택입니다. 자신의
프로그램에서 어떤 심볼을 남겨 두는 것이 무엇인지 알아보려면 `-ldflags=-dumpdep`로
빌드하고, 그 심볼에서 `main.main` 또는 `go:main.inittasks`까지 첫 번째 부모
체인을 따라가십시오.

v0.345.0은 색을 16진수로 쓰고 다시 읽기 위한 `ColorHex`와 `ParseColorHex`도
추가합니다.

## 접근성 (Accessibility) {#accessibility}

자신의 픽셀을 직접 그리는 툴킷은 무엇을 그렸는지 말해 주지 않으면 스크린 리더에게
읽을 거리를 주지 못합니다. 위젯은 역할, 이름, 값을 가진 `Accessible`을 구현하고,
`CollectA11y`가 이를 모아 호스트가 게시하도록 합니다. Linux에서는
[`window`]({{< relref "/surfaces/native-window.md" >}})를 통해 AT-SPI로, 그리고
[Android]({{< relref "/surfaces/android.md#accessibility" >}})에서는 가상 뷰 계층으로
게시됩니다.
