---
title: "터미널"
linkTitle: "터미널"
weight: 40
description: "go-widgets/tui는 셀 페인터를 통해 위젯을 렌더링합니다: 스냅숏 렌더러, 대화형 러너, 문자 격자에 맞춘 위젯."
tags: [surfaces, tui, terminal]
---

[`go-widgets/tui`](https://github.com/go-widgets/tui)는
[셀 페인터]({{< relref "/rendering/painter.md" >}})를 통해 위젯 트리를 터미널
프레임으로 바꿉니다. 표준 라이브러리만 사용하며 `x/term`도, cgo도 없습니다.

## 한 프레임 {#one-frame}

```go
_ = tui.RenderOnce(os.Stdout, widgets, nil) // nil theme: light
```

`RenderOnceSized(w, cols, rows, widgets, theme)`는 크기를 명시적으로 받고,
`RenderOnce`는 `COLUMNS`와 `LINES`를 읽으며 없으면 80×24로 대체합니다. `TIOCGWINSZ`를
호출하지 않고 환경을 읽기 때문에 패키지에 플랫폼별 코드가 들어가지 않습니다.

## 대화형 프로그램 {#an-interactive-program}

```go
app := tui.NewApp()
app.Root = buildScene()
app.Keys["q"] = func(a *tui.App) { a.Quit() }
app.Keys["Ctrl+C"] = func(a *tui.App) { a.Quit() }
os.Exit(app.Run())
```

`Run`은 대체 화면(alternate screen)과 raw 모드로 들어가고, 입력을 `Root`에
전달하며(전역 키 핸들러가 먼저), `SIGWINCH`를 따르고, 패닉이 났을 때도 항상 터미널을
복원합니다. 키 핸들러는 `Consume`을 호출하여 이벤트가 `Root`에 가지 않게 합니다.
`InputTarget`은 소비되지 않은 모든 이벤트를 하나의 위젯으로 보내며, 명령 팔레트가
열려 있는 동안 타이핑을 가로채는 방식이 이것입니다. `NewFocusRing`은 폼에 Tab과
Shift+Tab 순회를 제공합니다.

## 셀에 맞춘 위젯 {#widgets-sized-for-cells}

대부분의 툴킷 위젯은 픽셀 단위로 설계되어, 패딩 상수가 셀 페인터 아래에서는 셀을
세어 부풀어 오릅니다. 그래서 `tui`에는 자체 **셀 네이티브**(cell-native) 위젯
모음이 있습니다. `TextEditor`, `Entry`, `Button`, `CheckButton`, `RadioButton`,
`Scale`, `ProgressBar`, `Sparkline`, `ListBox`, `TreeView`, `Table`, `MenuBar`,
`Notebook`, `HSplit`, `VSplit`, `Dialog`, `Dropdown`, `Spinner` 등이며, 글리프 하나가
셀 하나입니다. 각 위젯은 여전히 `toolkit.Widget`이므로 같은 인스턴스가 픽셀로도
그려집니다.

`TextEditor`는 읽고 쓸 수 있는 코드 편집기입니다. 거터, 실행 취소와 다시 실행, 찾기와
바꾸기, 선택, 줄 이동, 그리고 Go(`go/scanner` 사용), JavaScript와 TypeScript,
Python, Ruby, 셸, C와 C++, Rust, JSON, YAML, HCL, TOML, LaTeX, Markdown의 구문 강조를
외부 의존성 없이 제공합니다.

```sh
go run ./cmd/tui-widgets | less -R     # every cell-native widget, one per slot
go run ./cmd/tui-widget-explorer       # the same, interactive
go run ./cmd/tui-explorer              # a file browser
go run ./cmd/tui-editor --file=x.go    # a modal editor
```

두 데모 프로그램은 실제 pty에서 처음부터 끝까지 테스트됩니다. 실제 키 바이트를
입력하고, 렌더링된 프레임을 검증합니다.
