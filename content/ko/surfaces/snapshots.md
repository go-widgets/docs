---
title: "스냅숏: SVG 또는 PNG로 된 위젯"
linkTitle: "스냅숏"
weight: 60
description: "go-widgets/svg는 모든 RGBA 렌더링을 비트 단위로 정확한 독립형 SVG로 감싸거나 PNG로 씁니다."
tags: [surfaces, svg, png]
---

[`go-widgets/svg`](https://github.com/go-widgets/svg)는 RGBA 버퍼를 독립형 SVG로
바꿉니다. 렌더링의 픽셀 크기에 고정된 `viewBox`와, `image-rendering: pixelated`
아래의 base-64 PNG로 된 픽셀로 이루어집니다. 표시되는 모든 픽셀은 위젯이 그린 바로
그 픽셀이며, 다시 벡터화하거나 글꼴을 대체하지 않고, 어떤 배율로 확대해도 선명하게
유지됩니다.

```go
// any RGBA producer
svg.Snapshot(f, surface, w, h, "my image")

// a toolkit widget
btn := toolkit.NewButton("Click me", nil)
btn.SetBounds(toolkit.Rect{W: 200, H: 40})
widget.Snapshot(f, btn, 200, 40, toolkit.DefaultLight(), "widget: button")
widget.PNG(p, btn, 200, 40, toolkit.DefaultLight())
```

레이블은 이스케이프되어 SVG의 `<title>`과 `aria-label`이 됩니다. 루트 패키지는
아무것에도 의존하지 않으며, `widget` 하위 패키지는 별도로 분리되어 있어 자체 픽셀
생성기를 가진 사용자가 툴킷을 모듈 그래프에 끌어오지 않아도 됩니다.

`svg/cmd/gallery-render`는 위젯 카탈로그를 밝은 테마와 어두운 테마로 렌더링합니다:
[go-widgets.github.io/svg](https://go-widgets.github.io/svg/).

## 애니메이션 SVG

`svg.Animate`는 시각이 붙은 SVG 프레임 목록(예: painter의
[`svgrec`](../../rendering/painter/) 레코더 출력)을 하나의 애니메이션 SVG로
합칩니다. 프레임 사이에서 바뀌지 않는 하위 트리는 `<defs>`에 한 번만 저장되고
`<use>`로 재사용됩니다. 연속된 동일 프레임은 병합되며, 타이밍은 순수 CSS
(`@keyframes`, 스크립트 없음)이고 `prefers-reduced-motion`으로 멈춥니다.

```go
out, err := svg.Animate([]svg.Frame{
    {At: 0, SVG: first},
    {At: 100 * time.Millisecond, SVG: second},
}, svg.Options{End: time.Second, Title: "demo"})
```

1000×700 프레임 60개에서, 일부 영역만 움직이는 화면은 1.49 MB에서 65 KB로
(22.9배 작게), 정지 화면은 24 KB로(60배) 줄어듭니다. 입력은 허용 목록으로
파싱되고 한도(프레임 수, 바이트, 시간, 크기)가 있으며, 모든 거부는 타입이 있는
오류입니다. `Document.Video`는 두 개의 작은 인터페이스로 제공하는 래스터라이저와
비디오 인코더를 구동합니다. WebM 인코더는 포함되어 있지 않습니다.
