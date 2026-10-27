---
title: "mvvmlint: 상태는 뷰모델을 거칩니다"
linkTitle: "mvvmlint"
weight: 10
description: "툴킷 위젯의 상태 필드를 직접 대입하거나, 위젯은 있는데 뷰모델이 없는 풀 리퀘스트를 실패 처리합니다."
tags: [ci, lint, mvvm]
---

[`go-widgets/mvvmlint`](https://github.com/go-widgets/mvvmlint)는 툴킷을 임포트하는
패키지에서만 동작합니다.

1. **위젯의 상태 필드를 직접 대입하는 경우.** `entry.Text = "x"`처럼 `entry`의 정적
   타입이 툴킷 위젯이고 `Text`가 상태 필드이면 지적됩니다. [바인더]({{< relref "/state/mvvm.md" >}})가
   받는 `&entry.Text`는 대입이 아니므로 지적되지 않으며, 복합 리터럴
   (`toolkit.Entry{Text: "x"}`)이나 `OnChange` 같은 콜백을 구성하는 것도
   지적되지 않습니다.
2. **위젯은 있고 뷰모델은 없는 경우.** 툴킷은 임포트하지만 `go-widgets/mvvm`은
   임포트하지 않는 패키지에는 진단이 하나 나옵니다. 뷰모델이 없는 것이 정당한 위젯
   라이브러리에서는 `-requirevm=false`로 이 검사를 끌 수 있습니다.

상태 필드는 툴킷 위젯의 필드(`Text`, `Items`, `Rows`, `Root`, `Selected`, `Value`,
`Checked`, …)이며, `-statefields=`로 목록을 교체할 수 있습니다. 이름이
`*_binding.go`인 파일, 또는 `//mvvmlint:allow`가 들어 있는 파일은 첫 번째 규칙에서
제외됩니다.

```sh
go install github.com/go-widgets/mvvmlint/cmd/mvvmlint@latest
go vet -vettool="$(go env GOPATH)/bin/mvvmlint" ./...
```

```yaml
# .github/workflows/mvvm.yml
name: mvvm
on:
  pull_request:
  push:
    branches: [main]
jobs:
  mvvm:
    uses: go-widgets/mvvmlint/.github/workflows/mvvmlint.yml@main
```

[`app-template`]({{< relref "/getting-started.md" >}})에는 이 작업이 처음부터
포함되어 있습니다.
