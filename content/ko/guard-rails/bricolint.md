---
title: "bricolint: 직접 그린 UI 금지"
linkTitle: "bricolint"
weight: 20
description: "애플리케이션 코드에서 페인터 기본 동작으로 크롬을 그리거나 프레임마다 위젯을 다시 만드는 풀 리퀘스트를 실패 처리합니다."
tags: [ci, lint, painter]
---

*브리콜라주(Bricolage)*, 즉 즉흥적으로 손수 만든 그리기는 인터페이스가 눌림, 호버,
포커스 피드백, 테마 적용, HiDPI 스케일링, 접근성을 조용히 잃게 되는 경로입니다.
[`go-widgets/bricolint`](https://github.com/go-widgets/bricolint)는 툴킷으로 옮겨 온
애플리케이션이 다시 예전 방식으로 흘러가지 않도록 막아 줍니다.

```go
p.FillRect(bar, theme.Bg)              // flagged: hand-drawn chrome
toolkit.NewBackdrop(theme.Bg).Draw(p)  // a toolkit widget

func (v *view) Draw(p painter.Painter) {
	b := toolkit.NewButton("ok")       // flagged: a new widget every frame
	b.Draw(p)
}
```

1. **애플리케이션 코드의 페인터 기본 동작**: 정적 타입이 페인터 타입인 수신자에 대한
   그리기 메서드(`FillRect`, `StrokeRect`, `FillPath`, `DrawImage`, `Text`,
   `PutPixel`, …) 호출입니다. 서피스를 조회하거나 클리핑하는 것(`Size`, `PushClip`,
   `PushTranslate`)은 그리기가 아니므로 지적되지 않습니다. `-primitives=`로 목록을
   교체할 수 있습니다.
2. **프레임마다 버려지는 위젯**: `Draw`, `Paint`, `Render`라는 이름의 메서드 안에서
   `toolkit.New…`를 호출하는 경우입니다. 그릴 때마다 다시 만드는 위젯은 상호작용
   상태를 유지하지 못합니다. 한 번만 만들어 필드에 보관하고 바인딩으로 구동하십시오.
   `-checkthrow=false`로 이 검사를 끌 수 있습니다.

일부 코드는 정말로 말단(leaf)입니다. 게임 프레임버퍼, 페인터 백엔드, 툴킷 자체의
내부가 그렇고, 이런 코드는 그 사실을 명시적으로 밝힙니다.

```go
p.FillRect(bg, c) //bricolint:allow engine SVG raster blit — a genuine leaf
```

```go
//bricolint:allowfile painter back-end — this file IS the leaf
package pdfsurface
```

**사유는 필수입니다**. 사유가 없는 지시자는 무시되며, 누군가 정당화 내용을 적을
때까지 지적이 계속 발생합니다.

```yaml
jobs:
  bricolint:
    uses: go-widgets/bricolint/.github/workflows/bricolint.yml@main
```
