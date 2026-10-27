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

## 직접 해 보기 {#trying-it}

저장소의 데모는 같은 위젯 세 개를 두 가지 페인터로 그립니다.

```sh
go run ./cmd/wui-demo --out demo.png     # pixels, written as a PNG
go run ./cmd/tui-demo --theme dark       # cells, written as ANSI to stdout
```

그리고 [go-widgets.github.io/painter](https://go-widgets.github.io/painter/)에서는
픽셀 페인터를 브라우저에서 볼 수 있습니다.
