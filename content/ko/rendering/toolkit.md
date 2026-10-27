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
| Composite | `FileChooser`, `ColorChooser`, `ColorPicker`, `FontChooser`, `Calendar`, `DatePicker`, `DateRangePicker`, `TimePicker`, `MarkdownView`, `MarkdownEditor`, `TerminalView` |
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
