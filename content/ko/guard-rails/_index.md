---
title: "CI가 강제하는 규칙"
linkTitle: "CI 규칙"
weight: 40
description: "애플리케이션이 필수 상태 검사로 실행하는 두 가지 go/analysis 검사: MVVM을 통한 상태 관리, 그리고 직접 그린 UI 금지."
tags: [ci, lint]
---

애플리케이션 UI를 건전하게 유지하는 두 가지 관례가 있는데, 문서로만 적어 두면 편리한
지름길 하나씩에 조금씩 무너질 것입니다. 그래서 각각을
[`go/analysis`](https://pkg.go.dev/golang.org/x/tools/go/analysis) 분석기로 만들어
`go vet`로 실행하고, 애플리케이션이 **필수** 검사로 지정할 수 있는 재사용 워크플로를
제공합니다.

| | 거부하는 것 |
|---|---|
| [`mvvmlint`]({{< relref "/guard-rails/mvvmlint.md" >}}) | 위젯의 상태 필드를 직접 대입하는 것, 그리고 위젯은 있는데 뷰모델이 없는 패키지 |
| [`bricolint`]({{< relref "/guard-rails/bricolint.md" >}}) | 애플리케이션 코드의 페인터 기본 동작, 그리고 프레임마다 새로 만들어지는 위젯 |

두 검사 모두 구조적으로 보수적입니다. 수신자의 **정적 타입**이 `go/types`를 통해
툴킷 또는 페인터 타입으로 확인될 때만 규칙이 적용됩니다. 관련 없는 타입에 이름이
같은 필드나 메서드가 있어도 지적되지 않습니다. 의도된 예외는 코드 안의 명시적
지시자(directive)로 표시하므로, 각 예외가 만들어지는 자리에 기록됩니다.
