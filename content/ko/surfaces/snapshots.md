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
