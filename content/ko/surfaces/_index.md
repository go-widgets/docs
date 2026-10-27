---
title: "그려지는 곳"
linkTitle: "서피스"
weight: 30
description: "go-widgets 트리를 표시할 수 있는 모든 서피스와, 그중에서 선택하는 방법."
tags: [surfaces, back-ends]
---

| 서피스 | 모듈 | 페인터 |
|---|---|---|
| X11, Wayland, macOS, Windows의 [네이티브 창]({{< relref "/surfaces/native-window.md" >}}) | `window` | 픽셀 |
| 트레이 아이콘과 시스템 모양을 갖춘 [애플리케이션]({{< relref "/surfaces/application.md" >}}) | `application`, `tray` | 픽셀 |
| [브라우저 탭]({{< relref "/surfaces/browser.md" >}}): 일반 `<canvas>` | `webcanvas` 또는 `window` | 픽셀 |
| [브라우저 데스크톱]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}}): wasmdesk 컴포지터의 창 | `window` (wasmbox) | 픽셀 |
| [터미널]({{< relref "/surfaces/terminal.md" >}}) | `tui` | 셀 |
| [Android APK]({{< relref "/surfaces/android.md" >}}) | `android`, `window` | 픽셀 |
| [그림]({{< relref "/surfaces/snapshots.md" >}}): SVG 또는 PNG | `svg` | 픽셀 |

위젯 트리는 이들 사이에서 바뀌지 않습니다. 바뀌는 것은 버퍼를 누가 소유하고 입력을
누가 전달하는가(디스플레이 서버, 브라우저, 터미널, Java 호스트)이며, 각 백엔드는
그 입력을 같은 `toolkit.Event`로 변환합니다.

## 있는 환경에 따라 선택됩니다 {#chosen-by-what-is-there}

`window.Open`은 백엔드 인자를 받지 않습니다. 환경, 즉 빌드된 플랫폼,
`$WAYLAND_DISPLAY`, `$DISPLAY`, Android 호스트가 내보내는 소켓을 살펴보고 동작할 수
있는 것을 고릅니다. Linux용으로 빌드한 바이너리는 Wayland 세션에서는 Wayland 창을,
X에서는 X11 창을 엽니다. 같은 Android 바이너리는 APK 안에서는 호스트에 연결하고,
Termux에서는 일반 Linux 창을 엽니다.
[`Open`이 선택하는 방법]({{< relref "/surfaces/native-window.md#how-open-chooses" >}})을
참고하십시오.
