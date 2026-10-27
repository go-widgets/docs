---
title: "네이티브 창"
linkTitle: "네이티브 창"
weight: 10
description: "go-widgets/window: X11, Wayland, GTK4, Cocoa, Win32, Android, wasmbox, 브라우저 탭 위의 하나의 Open/Run API, 모두 CGO_ENABLED=0."
tags: [surfaces, window, x11, wayland, macos, windows]
---

[`go-widgets/window`](https://github.com/go-widgets/window)는 실제 창을 열고 그 안에서
위젯 트리를 실행합니다. 레이아웃, 그리기, 표시(present), 입력을 `toolkit.Event`로
변환하는 과정을 창이 닫힐 때까지 반복합니다.

```go
w, err := window.Open(window.Config{Title: "Demo"})
if err != nil {
	return err
}
defer w.Close()
return w.Run(root) // root is any toolkit.Widget
```

`Config`는 **논리 포인트**(logical point) 단위의 초기 크기(0이면 읽기 편한 기본값을
요청), `WM_CLASS` 인스턴스와 클래스, `Theme`, `RenderScale`도 받습니다.

## Open이 선택하는 방법 {#how-open-chooses}

| 플랫폼과 환경 | 백엔드 | 화면에 도달하는 방법 |
|---|---|---|
| Linux, `$GO_WIDGETS_GTK` 설정됨 | GTK4 | GTK가 창을 소유합니다. 프레임버퍼는 `GtkPicture`이고 네이티브 컨트롤은 그 위의 실제 GTK 위젯입니다. libgtk-4 런타임이 필요합니다. |
| Linux, `$WAYLAND_DISPLAY` 설정됨 | Wayland | 컴포지터의 유닉스 소켓 위의 xdg-shell |
| Linux, 그 외에는 `$DISPLAY` | X11 | 유닉스 소켓 위의 X11 코어 프로토콜, MIT-SHM 사용 |
| macOS | Cocoa | [go-macos/objc](https://github.com/go-macos/objc)(purego)를 통한 `NSWindow`와 `NSView` |
| Windows | Win32 | user32와 gdi32 시스템 호출을 통한 최상위 `HWND`, `StretchDIBits` |
| Android, `$GW_ANDROID_SOCKET` 설정됨 | Android 호스트 | [Java 호스트]({{< relref "/surfaces/android.md" >}})로의 프레임 프로토콜, 픽셀은 공유 memfd에 |
| Android, 그 외 | Wayland 또는 X11 | Termux의 셸에도 연결할 디스플레이 서버가 있습니다 |
| `js/wasm`, wasmdesk 안 | wasmbox | [wasmdesk 컴포지터]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}})의 클라이언트 |
| `js/wasm`, 일반 페이지 | 브라우저 탭 | `webcanvas`를 통한 페이지의 `<canvas>`(`Config.Canvas`, 기본값 `"screen"`): 컴포지터도, `SharedArrayBuffer`도, 교차 출처 격리도 없음 |
| 그 외 모두 | — | `window.ErrUnsupported`, 그래서 크로스 빌드도 컴파일됩니다 |

모든 백엔드는 `CGO_ENABLED=0`입니다. X11 백엔드는 소켓 위에 처음부터 작성한 코어
프로토콜로, Xlib도 XCB도 쓰지 않으며, 양쪽 바이트 순서, Xauthority 쿠키, 키심(keysym)
매핑, 서버의 요청 크기 아래로 타일링한 `PutImage`, 그리고 `SCM_RIGHTS`로 공유
세그먼트를 전달하는 MIT-SHM 고속 경로를 갖추고 있습니다. Windows는 프로세스 자체의
DLL을 `syscall.NewLazyDLL`과 `syscall.NewCallback` 창 프로시저로 통해 접근하고,
macOS는 purego로 Objective-C 런타임을 통해 접근합니다.

각 백엔드에서 플랫폼과 무관한 부분(이벤트 매핑, 좌표 계산, 픽셀 패킹, 손상 사각형)은
모든 운영 체제에서 100% 커버되는 코덱에 있습니다. 얇은 플랫폼 접착 코드는 실제로
검증됩니다. [상태]({{< relref "/status.md" >}})를 참고하십시오.

## 바뀐 부분만 {#only-what-changed}

(`toolkit/scene.HostRoot`처럼) `DamageRenderer`를 구현하는 루트는 다시 그린
사각형을 보고하며, GTK4를 제외한 모든 백엔드는 그 사각형만 표시합니다. X11은 MIT-SHM
`ShmPutImage`로, Wayland는 `wl_shm` 손상 영역으로, 그리고 Cocoa, Win32, Android,
wasmbox, 브라우저 탭이 그렇게 합니다. 첫 프레임, 크기 조정, X11 `Expose`는 여전히
서피스 전체를 표시합니다.

## HiDPI {#hidpi}

기본값은 논리 포인트당 프레임버퍼 픽셀 하나입니다. UI는 읽기 편한 크기로 레이아웃되고
그려지며 컴포지터가 이를 확대합니다. Windows에서는 창이 모니터별 DPI 인식을
선언하고, OS가 논리 프레임을 물리 클라이언트 영역으로 확대합니다. `RenderScale`은
패널의 실제 해상도로 프레임버퍼를 요청하는데, 이것은 장치 픽셀 단위로 레이아웃하는
루트에서만 올바릅니다.

## 화면, 그리고 표시와 숨김 {#screens-and-showing-and-hiding}

`window.Screens()`는 연결된 디스플레이를 기본 화면부터 나열하며, 논리 포인트
단위이고 데스크톱의 패널은 제외되며, `Open` 전에 호출할 수 있습니다. 화면의 이름은
(EDID에서 가져온 `"DELL U2720Q"`처럼) 패널 자체의 이름이고, 패널이 이름을 게시하지
않을 때는 커넥터 이름(`"HDMI-1"`)으로 대체됩니다. 연결된 두 패널이 같은 이름을
게시하는 경우에도 마찬가지인데, 둘을 구별하지 못하는 이름은 이름이 아니기
때문입니다.

`window.Show`, `Hide`, `Raise`는 열린 창을 닫지 않고 화면에서 내렸다가 다시
가져옵니다. [트레이 애플리케이션]({{< relref "/surfaces/application.md#the-tray" >}})에
필요한 기능입니다.

| 백엔드 | Show | Hide | Raise |
|---|---|---|---|
| X11 | `MapWindow` | ICCCM withdraw, so it leaves the taskbar | map, raise, `_NET_ACTIVE_WINDOW` |
| Wayland | commit again | null buffer and commit | `ErrNotSupported`: xdg-shell has no such request |
| macOS | `orderFront:` | `orderOut:` | activate the app, `makeKeyAndOrderFront:` |
| Windows | `SW_SHOWNA` | `SW_HIDE` | `SW_RESTORE` and `SetForegroundWindow` |
| GTK, Android, wasmbox, browser tab | `ErrNotSupported` | `ErrNotSupported` | `ErrNotSupported` |

포커스는 플랫폼이 허용하는 것입니다. 포커스 탈취 방지 아래에서 `Raise`는 창이 주의를
원한다는 표시만 할 수 있으며, 그 여부를 알 수 없습니다.
