---
title: "첫 번째 애플리케이션"
weight: 5
description: "이미 MVVM 규칙을 따르고 CI가 통과하는 app-template에서 시작하거나, 빈 창을 열어 위젯을 추가합니다."
tags: [getting-started, app-template, window]
---

## 템플릿에서 시작하기 {#from-the-template}

[`go-widgets/app-template`](https://github.com/go-widgets/app-template)은
템플릿 저장소입니다. 검색 필드, 카테고리 필터, 조건에 맞는 행의 목록, 상태 줄을
갖춘 작지만 실제로 동작하는 브라우저 애플리케이션입니다. 더 큰 애플리케이션에
필요한 것이 이미 모두 연결되어 있습니다. 위젯이 전혀 없는 뷰모델, 거기에 바인딩된
뷰, wasm 호스트, 100% 커버리지 게이트, 그리고
[MVVM 게이트]({{< relref "/guard-rails/mvvmlint.md" >}})입니다.

```sh
git clone https://github.com/go-widgets/app-template my-app
cd my-app
go mod edit -module github.com/you/my-app
./build.sh          # dist/app.wasm, dist/wasm_exec.js, dist/index.html
```

`dist/`는 HTTP로 서비스하십시오. `file://`로 연 페이지는 wasm을 인스턴스화할 수 없습니다.

| 파일 | 담고 있는 내용 |
|---|---|
| `viewmodel.go` | 모든 상태를 [`mvvm`]({{< relref "/state/mvvm.md" >}}) 옵저버블과 커맨드로 담습니다. 위젯은 없습니다. 캔버스 없이 테스트할 수 있습니다. |
| `scene.go` | 위젯들이며, 각각 `mvvmtk`를 통해 뷰모델에 바인딩됩니다. 위젯의 상태 필드를 직접 대입하는 일은 없습니다. |
| `main.go` | 빌드 태그(`js && wasm`)가 붙은 유일한 파일입니다. 입력을 툴킷에 전달하고 프레임을 `<canvas>`로 전송합니다. |

앞의 두 파일을 수정하되, 상태는 계속 바인더를 통해 흐르게 하십시오. 위젯의 필드를
직접 대입하는 풀 리퀘스트는 CI의 `mvvm` 작업이 실패 처리합니다.

## 빈 창에서 시작하기 {#from-a-bare-window}

```go
package main

import (
	"github.com/go-widgets/toolkit"
	"github.com/go-widgets/window"
)

func main() {
	w, err := window.Open(window.Config{Title: "Demo"})
	if err == window.ErrUnsupported {
		return // no native back-end on this platform
	}
	if err != nil {
		panic(err)
	}
	defer w.Close()

	box := toolkit.NewVBox()
	box.Append(toolkit.NewLabel("Hello"))
	box.Append(toolkit.NewButton("Click me", func() {}))
	w.Run(box) // layout, draw, present, dispatch input, until closed
}
```

너비와 높이를 0으로 두면 백엔드에 읽기 좋은 기본값을 요청합니다. macOS에서는 주 화면의
일부 크기가 됩니다. `CGO_ENABLED=0 go build`는 어떤 머신에서든 각 플랫폼용 바이너리를
만들어 냅니다.

## 목적별 모듈 {#which-module-for-what}

| 원하는 것 | 시작할 모듈 |
|---|---|
| 데스크톱 창 | [`window`]({{< relref "/surfaces/native-window.md" >}}) |
| 트레이 아이콘과 시스템 외관을 갖춘 데스크톱 애플리케이션 | [`application`]({{< relref "/surfaces/application.md" >}}) |
| 브라우저의 페이지 | [`webcanvas`]({{< relref "/surfaces/browser.md" >}}) 또는 위의 템플릿 |
| 터미널 프로그램 | [`tui`]({{< relref "/surfaces/terminal.md" >}}) |
| Android 앱 | [`android`]({{< relref "/surfaces/android.md" >}}) |
| README에 넣을 위젯 그림 | [`svg`]({{< relref "/surfaces/snapshots.md" >}}) |

코드를 작성하기 전에 위젯 세트를 먼저 보려면
[갤러리](https://go-widgets.github.io/gallery/)를 여십시오. 모든 위젯 계열이
브라우저 캔버스에서 실시간으로 동작합니다.
