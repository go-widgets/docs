---
title: "애플리케이션과 트레이"
linkTitle: "애플리케이션과 트레이"
weight: 20
description: "go-widgets/application은 창 위의 수명 주기를 소유하고, go-widgets/tray는 메뉴 막대나 알림 영역에 아이콘과 메뉴를 둡니다."
tags: [surfaces, application, tray]
---

## 애플리케이션 {#the-application}

[`go-widgets/application`](https://github.com/go-widgets/application)은
[`window`]({{< relref "/surfaces/native-window.md" >}}) 위의 계층입니다. 창을 열고,
루프를 구동하고, 첫 프레임이 나타난 뒤 트레이 아이콘을 붙이고, 실시간 시스템
모양(어두운 모드 또는 밝은 모드, 강조색, 시스템 글꼴)을 애플리케이션에 전달하고,
접근성 트리를 게시하며, 첫 프레임이 화면에 나타나면 `onReady`를 호출합니다.

```go
spec := application.Spec{
	Name:       "News Reader",
	Identifier: "com.example.reader",
	Version:    "1.0.0",
	Tray: func() *tray.Menu {
		return tray.NewMenu().Add(
			tray.Item("Refresh", refresh),
			tray.Item("Quit", quit),
		)
	},
}
cfg := application.Config{Title: "News Reader", Width: 1200, Height: 800}
err := application.Run(spec, cfg, handler, func() { log.Println("on screen") })
```

`handler`는 `Frame`, `Resize`, 마우스, 스크롤, 키 이벤트로 이루어진 `Handler`를
구현하며, 선택적 기능(capability)을 얼마든지 구현할 수 있습니다. 모양, 단축키,
수정자가 있는 클릭과 키, 보조 클릭, 컨텍스트 메뉴, 클립보드, 접근성, 네이티브
컨트롤입니다. 루프는 핸들러를 감싸지 않고, 각 기능을 타입 단언으로 확인하여 찾은
것을 지원합니다. 테스트가 패키지에서 이 단언을 찾아내며, 패키지 문서가 그 모두를
명시하지 않으면 실패합니다.

붙일 수 없는 트레이는 침묵이 아니라 `Run`의 오류입니다. `errors.Is(err,
tray.ErrNoBackend)`로 이유를 알 수 있습니다. 트레이에 무슨 일이 생기든 창이 필요한
애플리케이션은 `Spec.Tray`를 nil로 두고 트레이를 따로 실행합니다.

`Run`은 루프가 반환될 때 창을 닫으므로, 트레이 애플리케이션의 "Open"은 `Run`을 다시
호출하여 새 창을 얻을 수 있습니다. 창이 아닌 호스트(데스크톱 셸, 브라우저 탭)를
위해서는 `Bind`와 `BindScaled`가 같은 핸들러에 연결된 `*toolkit.Surface`를
반환합니다.

## 그려진 위젯의 접근성 {#painted-accessibility}

자체 [툴킷]({{< relref "/rendering/toolkit.md#accessibility" >}}) 위젯을 그리는 핸들러는 `Accessible`을 구현하지 않으면 플랫폼의 접근성 계층에는 불투명한 사각형 하나로만 보입니다. `application.A11yElementsOf(root)`는 위젯 트리에서 그 설명(각 위젯의 역할, 레이블, 값, 경계. 최대 `MaxA11yElements` = 1000)을 만듭니다.

```go
func (v *View) A11yElements() []application.A11yElement {
	return application.A11yElementsOf(v.root)
}
```

macOS에서는 그려진 버튼이 AX 트리에 레이블과 함께 `AXButton`으로 나타나고, 그 버튼에 대한 `AXPress`는 클릭이 됩니다. 핸들러가 내장하는 네이티브 컨트롤도 그 옆에 트리에 남습니다. Windows(UI Automation)와 Linux(AT-SPI) 백엔드도 같은 설명을 사용합니다. 요소의 비활성 상태는 아직 공개되지 않으며, Windows 브리지는 요소 사각형을 거부합니다.

## 트레이 {#the-tray}

트레이 아이콘은 그려지는 위젯이 아니라 OS 통합이므로,
[`go-widgets/tray`](https://github.com/go-widgets/tray)는 각 플랫폼 고유의 API를
구동하며, 여기서도 `CGO_ENABLED=0`입니다.

| 플랫폼 | API | 경유 |
|---|---|---|
| macOS | `NSStatusItem`, `NSMenu` | purego와 Objective-C 런타임 |
| Windows | `Shell_NotifyIcon`, `TrackPopupMenu` | `golang.org/x/sys/windows` |
| Linux | `StatusNotifierItem`, `com.canonical.dbusmenu` | Go로 구현한 DBus |

```go
menu := tray.NewMenu().Add(
	tray.Item("Open", open),
	tray.Checkbox("Notifications", true, setNotify),
	tray.SubMenu("Recent", tray.NewMenu().Add(tray.Item("file.txt", nil))),
	tray.Separator(),
	tray.Item("Quit", func() { t.Quit() }),
)
t := tray.New(iconPNG).SetTooltip("My App").SetMenu(menu)
t.Run() // blocks on the platform's loop until Quit
```

이미 창의 루프를 실행 중인 프로그램은 `Run`에 메인 스레드를 줄 수 없습니다.
`Attach`는 같은 아이콘을 띄우고 바로 반환하며, `Quit`이 이를 내립니다.

네이티브 백엔드는 기본으로 켜져 있으며 기억해야 할 빌드 태그는 없습니다. 네이티브
백엔드가 없는 플랫폼에서 `Run`은 `ErrNoBackend`를 반환합니다. "여기에는 트레이가
없다"와 조용히 아무것도 하지 않는 트레이를 구분해 주는 것입니다. 테스트나 헤드리스
서비스는 `WithBackend(tray.NewHeadless())`를 전달합니다.

### 알림

`(*Tray).Notify(tray.Notification{Title, Body, OnClick})`는 네이티브 시스템 알림을 띄웁니다. macOS에서는 UNUserNotificationCenter, Linux에서는 `org.freedesktop.Notifications`, Windows에서는 Shell_NotifyIcon 풍선입니다. 제목은 최대 63룬, 본문은 최대 255룬입니다(`MaxNotificationTitle`, `MaxNotificationBody`). 제목이 비어 있으면 `ErrEmptyTitle`입니다. 오류는 원인을 알려 줍니다. `ErrNoNotifier`(백엔드 없음), `ErrNotRunning`, `ErrNotBundled`(macOS는 앱 번들에만 전달함), `ErrNotAuthorized`(사용자가 거부함). `tray.NewHeadless()`는 보낸 내용을 테스트용으로 `Headless.Notified`에 기록합니다.

## 링크 열기

`application.OpenURL(rawURL)`은 `http` 또는 `https` URL을 사용자의 브라우저에서 엽니다. macOS에서는 NSWorkspace, Windows에서는 ShellExecuteW, Linux에서는 OpenURI 포털 다음 `xdg-open`을 사용합니다. URL에는 호스트가 있어야 하고, 포트가 있다면 1~65535, 전체는 8 KiB 이하여야 합니다. 그 외에는 `ErrURLScheme` 또는 `ErrURLInvalid`로 거부되며, 런처가 없는 플랫폼은 `ErrOpenURLUnsupported`를 반환합니다.
