---
title: "페인터 경계"
linkTitle: "페인터"
weight: 10
description: "모든 위젯을 그릴 때 쓰는 기본 동작, 페인터가 추가할 수 있는 선택적 기능, 그리고 픽셀 페인터와 셀 페인터."
tags: [rendering, painter]
---

[`go-widgets/painter`](https://github.com/go-widgets/painter)는 모든 위젯이 그리기에
사용하는 단 하나의 인터페이스를 정의합니다.

```go
type Painter interface {
	FillRect(r Rect, c RGBA)
	StrokeRect(r Rect, c RGBA, lineW int)
	FillRoundRect(r Rect, radius int, c RGBA)
	StrokeRoundRect(r Rect, radius int, c RGBA, lineW int)
	PutPixel(x, y int, c RGBA)
	Text(x, y int, s string, ink RGBA)
	Size() (w, h int)
}
```

이 집합은 의도적으로 작게 유지했습니다. 모든 백엔드가 전부 구현해야 하고, 터미널 셀
격자는 모서리를 둥글게 하거나 선 굵기를 바꿀 수 없기 때문입니다. 그래서 계약은
그럴 수 없을 때 어떻게 되는지를 명시합니다. `StrokeRect`의 굵기는 셀 격자가 무시하는
힌트이고, `FillRoundRect`는 셀 격자에서 사각형 채우기로 대체됩니다.

## 선택적 기능 {#optional-capabilities}

모든 서피스가 할 수 있는 것은 아닌 기능은 기본 인터페이스에 들어 있지 않습니다.
그 기능이 필요한 위젯은 타입 단언을 하고, 답이 아니오일 때는 적절한 대체 모양을
그립니다.

| 인터페이스 | 메서드 | 용도 |
|---|---|---|
| `Clipper` | `PushClip`, `PopClip` | 스크롤되는 자식이 부모의 경계 안에 머물게 함 |
| `Translator` | `PushTranslate`, `PopTranslate` | 자식을 자신의 좌표계로 그림 |
| `PathPainter` | `FillPath`, `StrokePath` | 곡선, 차트, 안티앨리어싱된 도형 |
| `ImagePainter` | `DrawImage` | 아이콘, 썸네일, 그림 |
| `MaskPainter` | `DrawMask` | 글리프와 단일 색상의 기타 커버리지 마스크 |
| `FacePainter` | `TextFace` | 특정 글꼴 페이스와 크기의 텍스트 |

```go
if c, ok := p.(painter.Clipper); ok {
	c.PushClip(bounds)
	defer c.PopClip()
}
```

## 두 가지 페인터 {#the-two-painters}

| | 기록하는 대상 | 표시하는 주체 |
|---|---|---|
| `NewPixelPainter(buf, w, h)` | 호출자가 소유한 RGBA `[]byte`(BGRA 서피스에는 `NewPixelPainterBGRA`) | 네이티브 창, 브라우저 캔버스, Android 서피스, PNG 또는 SVG |
| `NewCellPainter(w, h)` | 24비트 ANSI 직렬화기를 갖춘 셀(룬, 전경색, 배경색) 격자 | [`tui`]({{< relref "/surfaces/terminal.md" >}})를 통한 터미널 |

버퍼는 계속 호출자의 소유입니다. `PixelPainter`는 기본 동작을 쓰기 작업으로 바꿀
뿐이므로, 같은 `[]byte`를 복사 없이 창 백엔드, `<canvas>`, 이미지 인코더에 넘길 수
있습니다.

## SVG로 기록하기 {#recording-to-svg}

`svgrec.Recorder`(패키지 `github.com/go-widgets/painter/svgrec`)는 세 번째
페인터입니다. 같은 호출을 **네이티브 SVG 문서**로 바꾸며, `<rect>`, `<path>`,
`<clipPath>` 요소로 이루어지고 `<image>`는 래스터가 실제로 그려지는 곳에만
쓰입니다. 결과는 무손실이며 크기를 바꿀 수 있고 결정적입니다. 같은 호출은 같은
바이트를 만듭니다.

```go
r, _ := svgrec.New(1000, 700)
widget.Draw(r, painter.LightTheme())
doc, err := r.SVG() // err is the first call that was refused, if any
```

`Clipper`, `Translator`, `PathPainter`, `ImagePainter`, `MaskPainter`를
구현합니다. SVG에 충실한 대응물이 없는 곳에서는 조용히 근사하는 대신 대응표가
그 사실을 밝힙니다.

| 호출 | SVG |
|---|---|
| `FillRect`, `PutPixel` | `<rect>` |
| `FillRoundRect` | `<rect rx>`; 모서리의 안티앨리어싱이 픽셀 페인터와 다릅니다 |
| `StrokeRect`, `StrokeRoundRect` | `evenodd` 프레임, 링은 픽셀 페인터처럼 안쪽으로 그려집니다 |
| `Text` | 글리프 윤곽선(아래 참고) |
| `PushClip` / `PushTranslate` | `<clipPath>`와 중첩된 `<g>`; 평행 이동은 좌표에 적용됩니다 |
| `FillPath`, `StrokePath` | `<path>`; 선 두께는 사용자 단위이므로 viewBox에 따라 크기가 변하며, 길이가 0인 하위 경로는 픽셀 페인터에서는 아무것도 그리지 않지만 SVG에서는 점을 그립니다 |
| `DrawImage`, `DrawMask` | PNG 데이터 URI를 가진 `<image>` |

**텍스트는 `<text>`가 아니라 윤곽선이 됩니다.** 내장 5x7 글꼴은 픽셀 격자이므로
각 문자열은 병합된 런으로 이루어진 하나의 `<path>`가 되며, 보조 기술을 위해
텍스트를 `<title>`에 담은 `<g>`로 감싸집니다. 부분 집합 글꼴을 내장하려면 글꼴
작성기와 이를 존중하는 뷰어가 필요하고, 그렇지 않은 뷰어는 다른 글꼴로 대체하여
레이아웃이 어긋납니다. 대가로 텍스트를 선택할 수 없습니다. TrueType 글꼴로 조판한
텍스트는 아직 윤곽선으로 기록되지 않으며, `FacePainter`는 제공되지 않으므로
래스터 `DrawMask`로 도착합니다.

페인터 메서드는 오류를 반환하지 않으므로, 거부된 호출(잘못된 UTF-8, XML이 담을 수
없는 문자, 1<<24를 넘는 좌표, `Limits`를 초과하는 호출이나 문서)은 버려지고 첫
번째 오류는 `Err()`와 `SVG()`가 보고합니다. 퍼즈 타깃은 모든 출력을
`encoding/xml`로 다시 파싱하고, 작성기가 직접 만들지 않은 요소나 속성이 있으면
실패합니다.

픽셀 페인터와 비교하면(순수 Go `go-gfx/gfx/svg`로 래스터화), 샘플 위젯, 사각형,
비트맵 텍스트, 클립, 평행 이동은 픽셀 단위로 일치하고, 경로는 채널당 최대 1만큼
다르며, 안티앨리어싱된 둥근 모서리는 최악의 장면에서 픽셀의 7.5 %에 대해 채널당
최대 61만큼 다릅니다.

## 직접 해 보기 {#trying-it}

저장소의 데모는 같은 위젯 세 개를 두 가지 페인터로 그립니다.

```sh
go run ./cmd/wui-demo --out demo.png     # pixels, written as a PNG
go run ./cmd/tui-demo --theme dark       # cells, written as ANSI to stdout
```

그리고 [go-widgets.github.io/painter](https://go-widgets.github.io/painter/)에서는
픽셀 페인터를 브라우저에서 볼 수 있습니다.
