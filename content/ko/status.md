---
title: "상태"
weight: 60
description: "실제 디스플레이, 실제 기기, 실제 터미널에서 검증된 것, 컴파일만 된 것, 그리고 CI가 아직 다루지 않는 것."
tags: [status]
---

이 페이지들은 [모듈]({{< relref "/modules.md" >}})에 나열된 태그의 모듈과 대조해
확인했습니다. 모듈의 README와 코드가 서로 다를 때는 코드를 따릅니다.

## 실제 환경에서 검증됨 {#proven-live}

프로세스 내부에서 테스트한 코덱은 코덱만 증명합니다. 다음은 플랫폼 자체에 직접
물어본 항목입니다.

| 항목 | 검증한 환경 |
|---|---|
| X11 창 | Xvfb 아래의 CI: 알려진 패턴을 표시, 캡처, 샘플링하고, `xdotool`로 클릭과 키 입력을 합성해 전달된 `toolkit.Event`를 검증하며, `Show`/`Hide`/`Raise`를 서버에서 다시 읽어 확인 |
| Cocoa 창 | macOS CI 레인: 창을 열어 렌더링하고, 픽셀을 샘플링하며, 클릭과 키 입력을 합성하고, 다른 고루틴에서 실제 `NSWindow`에 `Show`/`Hide`/`Raise`를 요청한 뒤 AppKit에서 다시 읽어 확인 |
| Win32 창 | Windows 11 arm64 VM: `VBox`, `Label`, `Button`을 렌더링하는 실제 창에서, 주입한 세 번의 클릭이 윈도우 프로시저를 거쳐 카운터를 0에서 3까지 올림 |
| Wayland 창 | xdg-shell의 map 및 unmap 규칙을 강제하는 가짜 컴포지터를 상대로 검증 |
| wasmbox 창 | 실제 wasmdesk 컴포지터를 구동하는 헤드리스 Chromium: 합성된 픽셀을 다시 읽고, 실제 클릭이 위젯으로 라우팅됨 |
| Android | Android 15 arm64 기기: 변경 영역만 전송하는 blit을 측정하고, memfd 더티 메모리를 측정하고, 접근성 트리를 기기에서 벤치마크 |
| macOS 트레이 | 실제 macOS 세션: 프로그램이 멈추면 아이콘이 메뉴 막대에서 사라지고, 시작하면 돌아오며, 메뉴가 열림 |
| Linux 트레이 | 실제 세션 버스를 쓰는 CI: 이름을 획득하고, 메뉴가 `GetLayout`에 응답하며, `Quit`가 이름을 반납함 |
| Windows 트레이 | Windows 11 VM의 아이콘과 메뉴 |
| 터미널 데모 | CI의 실제 pty: 키 바이트를 보내고 렌더링된 프레임을 검증 |

## 컴파일만 됨, 실행은 안 됨 {#compiled-not-run}

- Win32 `Show`/`Hide`/`Raise`: 결정 표는 단위 테스트를 거쳤지만, 창의 스레드에서의
  호출은 컴파일만 되었습니다.
- Windows 트레이의 `Attach`.
- `webcanvas`의 DOM 루프(`run_js.go`): `js && wasm` 빌드 태그 뒤에 있어 측정되는
  커버리지 밖에 있으며, 이를 사용하는 애플리케이션이 실행해 봅니다.
- `application`의 네이티브 `Run`: 실제 창을 여는 부분은 커버리지 게이트에서
  제외되며, 계약, 이벤트 변환, 준비 카운터는 커버됩니다.

## 아직 안 된 것 {#not-done}

- Windows와 Linux 트레이는 `MenuItem.Icon`을 무시합니다. 아이콘이 있는 행도
  아이콘 없이 그려집니다.
- `window`의 Wayland 백엔드는 `Raise`를 할 수 없습니다. xdg-shell에 그런 요청이
  없고, xdg-activation에는 사용자 자신의 입력에서 얻은 토큰이 필요합니다.
- Android: `android/arm64`만 지원합니다. cgo 없이 Go가 링크하는 유일한 Android
  대상이기 때문입니다.
- `window.Screens`는 `js/wasm`에서 `ErrScreensUnsupported`를 반환합니다.

## CI가 다루는 범위 {#what-ci-does-not-cover-yet}

2026-10-10에 이 페이지에 나열되어 있던 다섯 가지 공백이 해소되었습니다.

| 모듈 | 이전 | 현재 |
|---|---|---|
| `mvvmtk` | 커버리지 게이트 없음, amd64와 arm64만 | 툴킷이 쓰는 것과 똑같은 100% 게이트(이미 100.0%였음), Linux는 여섯 가지 전부 |
| `application` | Linux, macOS, Windows가 amd64만 | Linux는 여섯 가지 전부, macOS와 Windows는 amd64와 arm64 |
| `tray` | OS/아키텍처 조합 다섯 개 | Linux는 여섯 가지 전부, macOS는 둘, Windows는 하나 |
| `mvvmlint` | 크로스 빌드 없음 | Linux는 여섯 가지 전부, macOS와 Windows |
| `bricolint` | 다섯 개를 빌드하는 "6 arches" 단계 | loong64 추가 |

그 결과 열일곱 개의 라이브러리와 도구가 이제 모두 여섯 가지 64비트 Linux
아키텍처용으로 크로스 빌드되고, 열아홉 개 전부가 100% 구문 커버리지 게이트를
통과합니다. `app-template`과 `gallery`는 브라우저 애플리케이션이며 유일한 대상인
`js/wasm`용으로 빌드됩니다.

## 보안 {#security}

2026-10-10에 `govulncheck`(심볼 수준: 모듈이 도달할 수 있는 코드만 대상)로
측정했습니다.

- 19개 모듈 중 15개가 취약한 코드에 도달했습니다. go1.27.1의 표준 라이브러리
  (`html/template`, `net/http`, `crypto/tls`, `mime/multipart`; go1.27.2에서
  수정)와 `golang.org/x/net` v0.58.0(`data`의 gRPC를 통한 `http2`; v0.60.0에서
  수정)입니다.
- 같은 날 저녁에 수정했습니다. 모든 모듈의 CI가 go1.27.2로 빌드하고, 모든 모듈이
  `x/net` v0.60.0을 요구합니다. 대부분의 모듈에서 이는 간접 요구 사항이라 Renovate가
  갱신하지 않으므로 수동으로 올렸습니다. 19개 모듈 전체에 대한 두 번째 `govulncheck`
  검사에서는 도달 가능한 취약점이 나오지 않습니다.
- 릴리스(release): android v0.16.0, app-template v0.5.0, application v0.8.0,
  bricolint v0.4.1, data v0.4.0, mvvm v0.15.0, mvvmlint v0.4.1, mvvmtk
  v0.15.0, painter v0.16.0, skin v0.3.0, svg v0.7.0, tray v0.15.0, webcanvas
  v0.4.1, window v0.89.0이며, 각 버전은 `gorelease`가 API로부터 도출했습니다.
  2026-10-11에는 toolkit v0.329.0, 그리고 tui v0.62.0, desktop v0.20.0, gallery
  v0.7.0, isoicons v0.4.0이 릴리스되었습니다. 이 네 모듈은 모듈 그래프가
  `github.com/ajroetker/go-highway/hwy/goat` v0.0.0에 닿는데 이 버전은 게시된 적이
  없어([ajroetker/go-highway#73](https://github.com/ajroetker/go-highway/issues/73))
  `gorelease`가 모듈 그래프를 로드하지 못하므로, 같은 규칙에 따라 linux, darwin,
  windows에서 `apidiff`로 버전을 도출했습니다.
- 23개 저장소 전체의 워크플로(파일 31개)를 `actionlint`와 `wfaudit`로
  감사했습니다. 권한이 상승되는 트리거, 워크플로 수준의 쓰기 권한 부여, 업로드된
  체크아웃에 남은 토큰, 셸 단계의 신뢰할 수 없는 표현식은 없습니다. 모든 워크플로가
  권한을 명시적으로 선언하므로, 저장소의 기본 토큰에 의존하는 것은 없습니다.
