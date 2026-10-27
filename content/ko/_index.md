---
title: "go-widgets"
linkTitle: "홈"
type: docs
cascade:
  type: docs
description: "순수 Go 위젯 툴킷과 그것이 그려지는 모든 서피스: X11, Wayland, macOS, Windows의 네이티브 창, 브라우저 캔버스, 터미널, Android APK."
---

**하나의 위젯 트리를 Go가 그리고, Go가 닿는 모든 서피스(surface)에 표시합니다.**
X11, Wayland, macOS, Windows의 네이티브 창, 브라우저 탭의 `<canvas>`, 터미널,
Android APK가 그 대상입니다. C 툴체인도, 웹뷰도, 시스템 위젯 세트도 필요 없습니다.
모든 모듈은 `CGO_ENABLED=0`으로 빌드되며, 모든 픽셀은 툴킷이 직접 그립니다.

```go
w, err := window.Open(window.Config{Title: "Hello"})
if err != nil {
	return err
}
defer w.Close()

box := toolkit.NewVBox()
box.Append(toolkit.NewLabel("Hello from go-widgets"))
box.Append(toolkit.NewButton("Click me", func() { /* ... */ }))
return w.Run(box) // an X11 or Wayland window, an NSWindow, a Win32 window
```

위 코드는
[`cmd/windowdemo`](https://github.com/go-widgets/window/blob/main/cmd/windowdemo/main.go)를
줄인 것입니다. 같은 `box`를 브라우저, 터미널, PNG에 그리는 코드도 똑같고,
그 아래의 서피스만 다릅니다.

## 하나의 툴킷이 모든 것을 그리는 이유 {#why-one-toolkit-draws-everything}

플랫폼 고유의 버튼을 호출하는 위젯은 대상 플랫폼 중 가장 기능이 부족한 쪽만큼만
이식성이 있고, 플랫폼마다 모양도 달라집니다. go-widgets는 반대 방향을 택했습니다.
위젯의 `Draw`는 오직 [`painter.Painter`]({{< relref "/rendering/painter.md" >}})
하고만 통신하며, 이는 직사각형 채우기, 직사각형 테두리 그리기, 픽셀 찍기, 텍스트
그리기 같은 소수의 기본 동작(primitive)으로 이루어져 있습니다. 이 호출이 RGBA
픽셀이 될지 터미널 셀이 될지는 위젯이 아니라 백엔드가 결정할 일입니다.

그 대가는 숨기지 않고 말씀드립니다. 기본적으로 네이티브 외관이 없고, 접근성도
공짜로 주어지지 않습니다. 툴킷이 자체 [접근성 (accessibility) 트리]({{< relref "/rendering/toolkit.md#accessibility" >}})를
각 플랫폼에 직접 제공합니다. 그 대신 한 번 테스트한 하나의 위젯 세트가 어디서나 같은
동작을 하고, 어떤 머신에서든 이들 모든 대상으로 크로스 컴파일되는 바이너리를 얻습니다.

## 다음으로 읽을 문서 {#where-to-go-next}

| | |
|---|---|
| [첫 번째 애플리케이션]({{< relref "/getting-started.md" >}}) | `app-template`에서 시작하거나, 빈 창에서 시작 |
| [위젯이 화면에 도달하는 과정]({{< relref "/rendering/_index.md" >}}) | 페인터 경계, 위젯 세트, 스킨 |
| [페인터 경계]({{< relref "/rendering/painter.md" >}}) | 모든 위젯을 그릴 때 쓰는 기본 동작 |
| [툴킷]({{< relref "/rendering/toolkit.md" >}}) | 160가지 위젯 타입, 테마, 레이아웃, 텍스트, 접근성 |
| [스킨]({{< relref "/rendering/skin.md" >}}) | 위젯의 부품과 상태를 데이터 파일로 기술 |
| [상태는 뷰모델에 둡니다]({{< relref "/state/_index.md" >}}) | 위젯 필드를 직접 대입하지 않는 이유 |
| [MVVM]({{< relref "/state/mvvm.md" >}}) | 옵저버블, 커맨드, 바인더, 실행 취소 |
| [데이터 척추]({{< relref "/state/data.md" >}}) | 타입이 있는 레코드, 쿼리, 로컬 또는 원격 저장소 |
| [그려지는 곳]({{< relref "/surfaces/_index.md" >}}) | 모든 백엔드와 선택 방식 |
| [네이티브 창]({{< relref "/surfaces/native-window.md" >}}) | X11, Wayland, Cocoa, Win32, GTK4, Android, wasmbox, 브라우저 탭 |
| [애플리케이션과 트레이]({{< relref "/surfaces/application.md" >}}) | 수명 주기, 외관, 메뉴 막대 아이콘 |
| [브라우저 탭]({{< relref "/surfaces/browser.md" >}}) | 일반 `<canvas>` 또는 wasmdesk 창 |
| [터미널]({{< relref "/surfaces/terminal.md" >}}) | 셀 기반 네이티브 위젯과 대화형 러너 |
| [Android]({{< relref "/surfaces/android.md" >}}) | CGO 없는 프로세스가 그리는 실제 APK |
| [스냅숏]({{< relref "/surfaces/snapshots.md" >}}) | 위젯을 SVG 또는 PNG로 |
| [CI가 강제하는 규칙]({{< relref "/guard-rails/_index.md" >}}) | `mvvmlint`와 `bricolint` |
| [모듈]({{< relref "/modules.md" >}}) | 열아홉 개 전체, 소스와 참조 문서 포함 |
| [상태]({{< relref "/status.md" >}}) | 어디서 무엇이 검증되었고, 무엇이 아직 안 되었는지 |

## 라이선스 {#licence}

BSD-3-Clause.
