---
title: "Android"
linkTitle: "Android"
weight: 50
description: "go-widgets/android: 인터페이스 전체를 CGO_ENABLED=0 Go 프로세스가 레이아웃하고 그리는 설치 가능한 APK."
tags: [surfaces, android]
---

Android는 CGO를 쓰지 않는 프로세스가 창을 소유할 수 없는 유일한 플랫폼입니다. 그릴 수
있는 서피스에 이르는 모든 경로가 JNI를 거치고, JNI에는 cgo가 필요하기 때문입니다.
X11과 Wayland처럼 대신 사용할 수 있는 와이어 프로토콜도 없습니다.

그래서 [`go-widgets/android`](https://github.com/go-widgets/android)는 두 개의
프로세스로 이루어집니다.

| | |
|---|---|
| **Java 호스트** (`host/`) | `Activity`, `SurfaceView`, 터치, 키, 수명 주기를 소유합니다. 픽셀을 전송(blit)할 뿐 위젯에 대해서는 아무것도 모릅니다. |
| **Go 애플리케이션** (`cmd/gwapp`) | 평범한 `CGO_ENABLED=0 GOOS=android` 실행 파일입니다. 레이아웃, 위젯, 테마, 히트 테스트, 포커스를 소유하며, 다른 모든 백엔드와 다르지 않습니다. |

Java 호스트는 Linux에서 X 서버가 있는 자리에 서 있습니다. 픽셀은 애플리케이션이
만들어 소켓으로 호스트에 넘기는 **memfd**를 통해 전달됩니다. Go 쪽은 RGBA_8888을
쓰는데 이것은 Android의 ARGB_8888과 바이트 단위로 같으므로, 전송은 변환 없이 손상된
(damaged) 사각형만 복사하는 것입니다. 입력, 인셋, IME 텍스트, 접근성 트리는 추상
`LocalSocket` 위의 프레임 프로토콜로 오갑니다.

```go
c, err := android.Dial("my app", nil) // nil theme: toolkit.DefaultDark()
if errors.Is(err, android.ErrUnsupported) {
	return nil // not under a host
}
defer c.Close()
return c.Run(myWidgetTree())
```

`Client`는 `window`의 `Backend`를 만족하므로, 애플리케이션은 창 위쪽 코드를 한 줄도
바꾸지 않고 Android와 데스크톱 백엔드 사이를 오갈 수 있습니다. 호스트의 소켓이
있으면 [`window.Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}})이
스스로 이것을 선택합니다.

## arm64 전용 {#arm64-only}

`android/arm64`는 Go가 cgo 없이 링크하는 유일한 Android 타깃이며, `arm`, `amd64`,
`386`은 외부 링킹이 필요합니다. CI는 빌드되는 하나와 빌드되지 않는 셋, 두 쪽을 모두
검증하므로, Go가 이 제약을 풀면 빌드가 그것을 알려 줍니다.

## 접근성 {#accessibility}

스크린 리더는 `SurfaceView`를 하나의 불투명한 사각형으로 봅니다. 그래서 호스트는
대신 가상 뷰 계층을 제공합니다. 접근 가능한 위젯마다 노드가 하나씩 있고, Android가
안내 문구를 고르는 기준인 `android.widget.*` 클래스, 텍스트, 경계, 활성화 동작을
갖습니다. 트리는 누군가 읽을 때 **끌어오며(pull)** 밀어 넣지 않으므로, 접근성
서비스가 연결되지 않은 앱은 트리를 만들지 않습니다. 활성화는 요소 중앙에서의 평범한
클릭으로 돌아오며, 터치와 같은 코드를 거칩니다.

## 터치와 애니메이션 {#touch-and-animation}

각 터치 샘플은 터치 이벤트로, 그다음 마우스 이벤트로 트리에 도달합니다. 앞의 것은
제스처를 인식하는 위젯을 위한 것이고, 뒤의 것은 클릭만 기다리는 많은 위젯을 위한
것입니다. 애니메이션 위젯은 무언가 애니메이션을 시작할 때 시작하고 아무것도 하지
않을 때 멈추는 프레임 루프로 진행됩니다.
