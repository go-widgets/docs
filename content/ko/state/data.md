---
title: "데이터 스파인"
linkTitle: "데이터"
weight: 20
description: "go-widgets/data: 검증이 있는 타입 레코드, 순수 쿼리 엔진, 그리고 같은 파이프라인을 프로세스 내부 또는 gRPC로, 네이티브와 wasm 모두에서 실행하는 스토어."
tags: [state, data, grpc]
---

[`go-widgets/data`](https://github.com/go-widgets/data)는 헤드리스(headless)입니다.
표준 라이브러리와 [`mvvm`]({{< relref "/state/mvvm.md" >}})만 가져오며, 위젯은 전혀
가져오지 않습니다.

| 구성 요소 | 설명 |
|---|---|
| `Value`, `Kind` | 비교 가능한 타입 스칼라: string, int, float, bool |
| `Record`, `Schema`, `Field`, `Rule` | 타입이 있는 행과 그 검증 |
| `Query` → `Apply` → `View` | 쿼리 엔진: 필터, 정렬, 그룹, 페이지, 집계 |
| `Proxy` | 백엔드 경계: `List`, `Query`, `Mutate` |
| `MemoryProxy` | 프로세스 내부 백엔드 |
| `grpcproxy.Server`, `grpcproxy.Client` | 같은 계약을 gRPC로, WebSocket 위에서 전달 |
| `Store[R]` | 타입이 있는 컬렉션으로, `mvvm.ObservableList`로 게시됨 |

## 같은 답, 로컬이든 원격이든 {#the-same-answer-local-or-remote}

`Store`는 `Proxy`와 통신하며 그것이 어느 쪽인지 알지 못합니다. `Apply`는 레코드와
쿼리에 대한 순수 함수이므로 클라이언트와 서버가 같은 행에 대해 같은 코드를
실행하고, `MemoryProxy`와 `grpcproxy.Client`는 바이트 단위로 동일한 뷰를 반환합니다.
적합성 테스트가 바로 이것을 검사합니다. 정렬, 필터, 그룹, 페이지, 집계 쿼리 모음을
두 프록시와 엔진 모두에 통과시키고, 모든 결과를 정규화한 다음, 세 가지를 바이트
단위로 비교합니다.

WebSocket 전송은 `js/wasm`으로 컴파일되므로, 브라우저 애플리케이션도 같은
클라이언트를 통해 같은 서비스에 접근합니다.

```go
store := data.NewStore(mem, codec) // or a grpcproxy.Client: nothing else changes
store.SetQuery(data.Query{Sorts: []data.Sort{{Field: "salary", Desc: true}}, Limit: 20})
store.Load(ctx)
items := store.Items() // bind this to a view
```

`tkbind.BindTable`은 스토어를 툴킷 `Table` 뒤에 놓습니다. 헤더를 클릭하면 정렬하고,
group-by는 그룹화하며, 검증을 통과한 셀 편집은 프록시를 통한 변경(mutation)이
됩니다. 로컬이든 원격이든 같은 왕복입니다.
