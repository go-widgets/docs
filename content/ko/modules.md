---
title: "모듈"
weight: 50
description: "go-widgets의 열아홉 개 모듈, 각 모듈의 용도, 그리고 소스와 API 참조 문서의 위치."
tags: [modules]
---

API 참조 문서는 릴리스된 태그를 따라가는 [pkg.go.dev](https://pkg.go.dev/)가
소스에서 생성합니다. 이 페이지들은 설명을 제공할 뿐 그 내용을 복사하지 않습니다.
태그는 작성 시점의 최신 태그이며, 기준은 각 저장소 자체의 태그 목록입니다.

| 모듈 | 태그 | 설명 | 참조 문서 |
|---|---|---|---|
| [`painter`](https://github.com/go-widgets/painter) | v0.16.0 | [그리기 경계]({{< relref "/rendering/painter.md" >}}): 픽셀 및 셀 페인터 | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/painter) |
| [`toolkit`](https://github.com/go-widgets/toolkit) | v0.345.1 | [위젯 세트]({{< relref "/rendering/toolkit.md" >}}), 테마, 레이아웃, 텍스트 | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/toolkit) |
| [`skin`](https://github.com/go-widgets/skin) | v0.3.0 | [데이터로 기술한 위젯]({{< relref "/rendering/skin.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/skin) |
| [`isoicons`](https://github.com/go-widgets/isoicons) | v0.4.0 | 툴킷의 아이소메트릭 다이어그램용 아이소메트릭 아이콘 팩(클라우드 네이티브, AWS) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/isoicons) |
| [`mvvm`](https://github.com/go-widgets/mvvm) | v0.15.0 | [옵저버블, 커맨드, 바인더, 실행 취소]({{< relref "/state/mvvm.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvm) |
| [`mvvmtk`](https://github.com/go-widgets/mvvmtk) | v0.15.0 | [툴킷 위젯을 위한 단일 호출 바인더]({{< relref "/state/mvvm.md#mvvmtk-one-call-per-widget" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmtk) |
| [`data`](https://github.com/go-widgets/data) | v0.4.0 | [타입이 있는 레코드, 쿼리, 로컬 또는 원격 저장소]({{< relref "/state/data.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/data) |
| [`window`](https://github.com/go-widgets/window) | v0.89.1 | 여덟 가지 백엔드를 지원하는 [네이티브 창]({{< relref "/surfaces/native-window.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/window) |
| [`application`](https://github.com/go-widgets/application) | v0.8.1 | 창 위에 놓이는 [애플리케이션 수명 주기]({{< relref "/surfaces/application.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/application) |
| [`tray`](https://github.com/go-widgets/tray) | v0.15.0 | [트레이 아이콘과 그 메뉴]({{< relref "/surfaces/application.md#the-tray" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tray) |
| [`webcanvas`](https://github.com/go-widgets/webcanvas) | v0.4.1 | [브라우저 `<canvas>`에 그리는 장면]({{< relref "/surfaces/browser.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/webcanvas) |
| [`tui`](https://github.com/go-widgets/tui) | v0.62.0 | [터미널 렌더링과 셀 기반 네이티브 위젯]({{< relref "/surfaces/terminal.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/tui) |
| [`android`](https://github.com/go-widgets/android) | v0.16.0 | [CGO 없는 프로세스가 그리는 APK]({{< relref "/surfaces/android.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/android) |
| [`svg`](https://github.com/go-widgets/svg) | v0.7.0 | [SVG 또는 PNG로 렌더링]({{< relref "/surfaces/snapshots.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/svg) |
| [`mvvmlint`](https://github.com/go-widgets/mvvmlint) | v0.4.2 | [MVVM 검사]({{< relref "/guard-rails/mvvmlint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/mvvmlint) |
| [`bricolint`](https://github.com/go-widgets/bricolint) | v0.4.1 | [직접 그린 UI 금지 검사]({{< relref "/guard-rails/bricolint.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/bricolint) |
| [`app-template`](https://github.com/go-widgets/app-template) | v0.5.0 | [애플리케이션의 출발점]({{< relref "/getting-started.md" >}}) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/app-template) |
| [`gallery`](https://github.com/go-widgets/gallery) | v0.7.0 | 모든 위젯 계열의 [라이브 데모](https://go-widgets.github.io/gallery/) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/gallery) |
| [`desktop`](https://github.com/go-widgets/desktop) | v0.20.0 | go-freedesktop과 go-widgets를 조합한 데스크톱 셸(네이티브 및 wasmdesk) | [pkg.go.dev](https://pkg.go.dev/github.com/go-widgets/desktop) |

## 모든 모듈이 지키는 기준 {#what-every-module-is-held-to}

- `CGO_ENABLED=0`: cgo를 쓰지 않으며, 라이브러리 대신 명령줄 도구를 호출하지도
  않습니다.
- amd64, arm64, riscv64, loong64, ppc64le, s390x용으로 크로스 컴파일합니다.
  s390x는 마지막 빅엔디언 대상으로, 모든 와이어 인코딩을 엄밀하게 유지해 줍니다.
  해당되는 경우 `js/wasm`용으로도 컴파일합니다. 열일곱 개의 라이브러리와 도구는
  2026-10-10부터 모두 그렇게 하고 있으며, 나머지 둘은 브라우저
  애플리케이션입니다([상태]({{< relref "/status.md#what-ci-does-not-cover-yet" >}})).
- CI의 구문 커버리지 게이트: 열아홉 개 모두 100%입니다. 테스트에서 실행할 수 없는
  파일(브라우저 루프, 네이티브 실행 루프)이 있으면 모듈이 어느 파일이 제외되었는지와
  그 이유를 밝힙니다. 예외는 [상태]({{< relref "/status.md" >}})를 참조하십시오.
- BSD-3-Clause.
