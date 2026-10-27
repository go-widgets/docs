---
title: "위젯이 화면에 도달하는 과정"
linkTitle: "렌더링"
weight: 10
description: "위젯은 페인터를 통해 그리고, 페인터가 그것을 픽셀로 만들지 터미널 셀로 만들지 결정하며, 백엔드가 결과를 표시합니다."
tags: [rendering, painter, toolkit]
---

세 개의 계층이 있으며, 각 계층은 바로 아래 계층만 알고 있습니다.

```text
toolkit widget tree          Button, Table, Agenda, VBox ...  (or a skin.Object)
        │  Draw(p painter.Painter, theme *Theme)
        ▼
painter.Painter              FillRect, StrokeRect, Text, PutPixel, ...
        │
        ├─ PixelPainter  → an RGBA []byte   → window, browser canvas, PNG, Android
        └─ CellPainter   → a grid of cells  → a terminal, as 24-bit ANSI
```

위젯은 자신이 어떤 페인터를 받았는지 결코 알지 못합니다. 백엔드는 어떤 위젯이
그렸는지 결코 알지 못합니다. 하나의 위젯 트리를 조건 분기 코드 한 줄 없이 네이티브
창, 브라우저, 터미널에 모두 표시할 수 있는 이유가 바로 이것입니다.

| | |
|---|---|
| [페인터 경계]({{< relref "/rendering/painter.md" >}}) | 기본 동작, 선택적 기능, 두 가지 페인터 |
| [툴킷]({{< relref "/rendering/toolkit.md" >}}) | 위젯 세트, 테마, 레이아웃, 텍스트, 접근성 |
| [스킨]({{< relref "/rendering/skin.md" >}}) | Go 코드 대신 데이터로 위젯의 부품과 상태를 기술 |
