---
title: "数据主干"
linkTitle: "数据"
weight: 20
description: "go-widgets/data：带校验的类型化记录、纯函数式查询引擎，以及一个在进程内或通过 gRPC 运行同一管线的存储，原生和 wasm 均可使用。"
tags: [state, data, grpc]
---

[`go-widgets/data`](https://github.com/go-widgets/data) 是无界面的：它只导入标准库和
[`mvvm`]({{< relref "/state/mvvm.md" >}})，不导入任何控件。

| 组成部分 | 它是什么 |
|---|---|
| `Value`、`Kind` | 可比较的类型化标量：字符串、整数、浮点数、布尔值 |
| `Record`、`Schema`、`Field`、`Rule` | 类型化的行及其校验 |
| `Query` → `Apply` → `View` | 查询引擎：过滤、排序、分组、分页、聚合 |
| `Proxy` | 后端接缝：`List`、`Query`、`Mutate` |
| `MemoryProxy` | 进程内后端 |
| `grpcproxy.Server`、`grpcproxy.Client` | 通过 gRPC 提供的同一契约，经由 WebSocket 承载 |
| `Store[R]` | 类型化集合，以 `mvvm.ObservableList` 的形式发布 |

## 本地或远程，答案相同 {#the-same-answer-local-or-remote}

`Store` 与某个 `Proxy` 对话，却从不知道是哪一个。`Apply` 是记录和查询的纯函数，
因此客户端和服务端在相同的行上运行同样的代码——`MemoryProxy` 和 `grpcproxy.Client`
返回逐字节相同的视图。一致性测试检查的正是这一点：一组排序、过滤、分组、分页和聚合查询，
分别经过两个代理和引擎，每个结果都经过规范化，三者逐字节比较。

WebSocket 传输层可编译为 `js/wasm`，因此浏览器应用通过同一个客户端访问同一项服务。

```go
store := data.NewStore(mem, codec) // or a grpcproxy.Client: nothing else changes
store.SetQuery(data.Query{Sorts: []data.Sort{{Field: "salary", Desc: true}}, Limit: 20})
store.Load(ctx)
items := store.Items() // bind this to a view
```

`tkbind.BindTable` 把存储放在工具包的 `Table` 之后：点击表头即排序，按分组依据即分组，
通过校验的单元格编辑会变成经由代理的变更——无论本地还是远程，都是同样的往返。
