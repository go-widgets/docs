---
title: "The data spine"
linkTitle: "Data"
weight: 20
description: "go-widgets/data: typed records with validation, a pure query engine, and a store that runs the same pipeline in-process or over gRPC, natively and in wasm."
tags: [state, data, grpc]
---

[`go-widgets/data`](https://github.com/go-widgets/data) is headless: it
imports the standard library and [`mvvm`]({{< relref "/state/mvvm.md" >}}),
and no widget.

| Piece | What it is |
|---|---|
| `Value`, `Kind` | a comparable typed scalar: string, int, float, bool |
| `Record`, `Schema`, `Field`, `Rule` | a typed row and its validation |
| `Query` → `Apply` → `View` | the query engine: filter, sort, group, page, aggregate |
| `Proxy` | the back-end seam: `List`, `Query`, `Mutate` |
| `MemoryProxy` | the in-process back-end |
| `grpcproxy.Server`, `grpcproxy.Client` | the same contract over gRPC, carried over a WebSocket |
| `Store[R]` | a typed collection, published as an `mvvm.ObservableList` |

## The same answer, local or remote

`Store` talks to a `Proxy` and never knows which one. `Apply` is a pure
function of the records and the query, so the client and the server run the
same code on the same rows — and a `MemoryProxy` and a `grpcproxy.Client`
return byte-identical views. The conformance test checks exactly that: a
battery of sort, filter, group, page and aggregate queries through both
proxies and through the engine, every result canonicalised, all three
compared byte for byte.

The WebSocket transport compiles to `js/wasm`, so a browser application
reaches the same service through the same client.

```go
store := data.NewStore(mem, codec) // or a grpcproxy.Client: nothing else changes
store.SetQuery(data.Query{Sorts: []data.Sort{{Field: "salary", Desc: true}}, Limit: 20})
store.Load(ctx)
items := store.Items() // bind this to a view
```

`tkbind.BindTable` puts a store behind a toolkit `Table`: a header click
sorts, a group-by groups, and a validated cell edit becomes a mutation through
the proxy — the same round trip, local or remote.
