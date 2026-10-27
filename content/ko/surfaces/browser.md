---
title: "브라우저 탭"
linkTitle: "브라우저"
weight: 30
description: "go-widgets/webcanvas로 일반 <canvas>에서, 또는 wasmdesk 컴포지터의 창으로 위젯 트리를 실행합니다."
tags: [surfaces, wasm, browser, webcanvas, wasmdesk]
---

툴킷은 DOM에 의존하지 않는 순수 Go이므로 `GOOS=js GOARCH=wasm`이 다른 타깃과
똑같이 빌드합니다. 결과물을 사용자 앞에 보여 주는 방법은 두 가지입니다.

## 일반 캔버스 {#a-plain-canvas}

[`go-widgets/webcanvas`](https://github.com/go-widgets/webcanvas)는 RGBA
프레임버퍼를 `<canvas>`에 전송하고 DOM의 포인터 및 키보드 이벤트를 장면으로 다시
전달합니다. 컴포지터도, `SharedArrayBuffer`도, 교차 출처 격리 헤더도 필요 없으며,
어떤 정적 호스트로도 제공할 수 있습니다.

```go
//go:build js && wasm

package main

import "github.com/go-widgets/webcanvas"

func main() { webcanvas.Run("screen", myScene()) }
```

장면은 `App`을 구현합니다. `Size`, `Draw(buf []byte)`, 그리고 이벤트 종류마다 하나의
메서드이며, 각 메서드는 무언가 바뀌었는지를 보고하므로 바뀌지 않은 프레임은 다시
그려지지 않습니다. `Ticker`, `Animator`, `Resizer`, `Scroller`도 구현할 수 있습니다.

[갤러리](https://go-widgets.github.io/gallery/)와
[`app-template`]({{< relref "/getting-started.md" >}})은 이 방식으로 실행됩니다.

아니면 `window`가 선택하게 할 수도 있습니다. window v0.87.0부터 일반 페이지에서
`js/wasm`용으로 빌드된
[`window.Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}})은
`webcanvas`를 통해 페이지의 `<canvas>`(`Config.Canvas`, 기본값 `"screen"`)에 그리는
백엔드를 반환합니다. 그러면 `window`에 맞춰 작성된 애플리케이션이 네이티브 창으로도,
브라우저 탭에서도, wasmdesk에서도 그대로 실행됩니다.

## wasmdesk 데스크톱의 창 {#a-window-on-the-wasmdesk-desktop}

wasmdesk 워커 안에서 [`window.Open`]({{< relref "/surfaces/native-window.md" >}})은
[wasmdesk/wasmbox](https://github.com/wasmdesk/wasmbox) 브라우저 컴포지터의
클라이언트를 반환합니다. 클라이언트는 `SharedArrayBuffer`에 서피스를 할당하고,
`MessagePort`로 `hello`를 보내고, `welcome`을 기다리고, 공유 버퍼에 그린 뒤
`commit`을 게시합니다. 서피스 전체이거나 손상된 사각형만입니다. 컴포지터의 `input`
메시지는 X11에서처럼 `toolkit.Event`가 됩니다.

그래서 `window`에 맞춰 작성된 애플리케이션은 네이티브 창으로도, 브라우저 탭 안의
데스크톱 창으로도 그대로 실행됩니다.
[데스크톱 셸 데모](https://github.com/go-widgets/desktop)가 바로 그렇게 하며, 소스는
하나입니다.

프로토콜 코덱은 모든 플랫폼에서 100% 커버됩니다. 실제 검증은 헤드리스 Chromium을
실제 wasmdesk 컴포지터에 대해 실행합니다. 클라이언트를 띄우고, 컴포지터가 직접
합성한 픽셀을 읽고, 클릭하고, 그 클릭이 위젯에 도달했는지 확인합니다.
