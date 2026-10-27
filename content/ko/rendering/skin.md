---
title: "스킨: 위젯의 부품과 상태를 데이터로"
linkTitle: "스킨"
weight: 30
description: "go-widgets/skin은 위젯의 부품, 상태별 모양, 신호로 구동되는 전환을 .skin.json 파일에 기술하고, 하나의 엔진이 이를 렌더링합니다."
tags: [rendering, skin, theme]
---

툴킷의 위젯은 Go로 작성되어 있습니다. `Button`의 부품, 호버, 눌림, 비활성 모양,
클릭 피드백이 모두 `button.go`에 들어 있습니다. 빠르고 정확하지만, 모양을 바꿀
때마다 코드를 바꿔야 합니다.

[`go-widgets/skin`](https://github.com/go-widgets/skin)은 그 기술을 데이터로
옮깁니다. Enlightenment의 Edje에서 가져온 발상입니다. `.skin.json` 문서는 위젯의
**부품(part)**, 각 **상태(state)**에서의 모양, 신호가 도착했을 때 부품을 상태
사이에서 움직이는 **프로그램(program)**을 정의합니다. 그러면 스킨을 바꾸거나 새
컨트롤을 만들 때 다시 컴파일할 필요가 없습니다. 프로그램이 다른 바이트를 읽어 들일
뿐입니다.

```go
theme, err := skin.Load(jsonBytes) // parse and validate every collection
if err != nil {
	return err
}
obj, _ := theme.New("button")
obj.Bind(skin.NewMVVMContext().Set("$.label", labelObs))
obj.SetBounds(toolkit.Rect{W: 96, H: 28})

// each frame
obj.OnEvent(ev)               // pointer events become signals, signals states
busy := obj.Tick(time.Now())  // advance the transitions
obj.Draw(p, theme.Palette())
```

`skin.Object`는 `toolkit.Widget`이므로 직접 작성한 위젯 옆에서 `VBox`에 들어갈 수
있습니다. 레이아웃은 위젯을 배치하고, 스킨은 위젯 하나의 내부를 구성합니다.

## 형식 {#the-format}

```jsonc
{
  "collections": {
    "button": {
      "min": { "w": 96, "h": 28 },
      "parts": [
        { "name": "bg", "type": "rect",
          "states": {
            "default": { "color": "@surface",     "border": "@border", "radius": 6 },
            "hover":   { "color": "@surface_alt", "border": "@border", "radius": 6 },
            "pressed": { "color": "@accent",      "border": "@border", "radius": 6 } } },
        { "name": "label", "type": "text", "text_from": "$.label", "align": [0.5, 0.5],
          "states": { "default": { "ink": "@on_surface" } } }
      ],
      "programs": [
        { "on": "mouse,in",    "target": ["bg", "label"], "to": "hover", "in": 0.12, "ease": "ease_out_cubic" },
        { "on": "mouse,click", "emit": "clicked" }
      ]
    }
  }
}
```

- **부품**은 `rect`, `text`, `image` 중 하나이며, 객체 또는 앞선 부품을 기준으로 한
  두 개의 상대 모서리(`rel1`, `rel2`)와 픽셀 및 글꼴 상대(`offset_em`) 오프셋으로
  배치됩니다. 그래서 텍스트 부품은 글꼴을 따라갑니다.
- **상태**는 색상, 테두리, 반지름, 표시 여부를 설정하며, 부품을 이동하거나 크기를
  바꿀 수도 있습니다.
- **프로그램**은 신호에 반응해 지정한 지속 시간 동안 이름 있는 이징으로 부품을 한
  상태로 옮기고, 신호를 바깥으로 `emit`할 수도 있습니다. 예를 들어
  [`mvvm.Command`]({{< relref "/state/mvvm.md" >}})로 보낼 수 있습니다.
- 색상은 툴킷 `Theme`에 대해 해석되는 `@tokens`(모든 GTK `@define-color` 항목을
  `@extra:<name>`으로 포함), 16진수 값, 또는 RGBA 배열입니다.

JSON은 표준 라이브러리가 디코딩하므로 서드파티 의존성이 없습니다.

## 직접 작성한 위젯과 대조해 검증함 {#proven-against-the-hand-written-widgets}

툴킷의 버튼, 체크 박스, 스위치, 칩, 카드를 각각 스킨 컬렉션으로 다시 작성했으며,
테스트는 모든 상태에서 두 가지가 **픽셀 단위까지 동일하게** 렌더링됨을 검증합니다.
스킨으로 그리는 `Draw`는 직접 작성한 것과 벤치마크로 비교 측정합니다.
