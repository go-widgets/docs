---
title: "データスパイン"
linkTitle: "データ"
weight: 20
description: "go-widgets/data：検証付きの型付きレコード、純粋なクエリエンジン、そしてネイティブでも wasm でも、プロセス内でも gRPC 経由でも同じパイプラインを実行するストア。"
tags: [state, data, grpc]
---

[`go-widgets/data`](https://github.com/go-widgets/data) はヘッドレスです。インポートするのは標準ライブラリと
[`mvvm`]({{< relref "/state/mvvm.md" >}}) だけで、ウィジェットは含みません。

| 部品 | 内容 |
|---|---|
| `Value`, `Kind` | 比較可能な型付きスカラー：文字列、整数、浮動小数点数、真偽値 |
| `Record`, `Schema`, `Field`, `Rule` | 型付きの行とその検証 |
| `Query` → `Apply` → `View` | クエリエンジン：フィルタ、ソート、グループ化、ページング、集計 |
| `Proxy` | バックエンドとの継ぎ目：`List`、`Query`、`Mutate` |
| `MemoryProxy` | プロセス内のバックエンド |
| `grpcproxy.Server`, `grpcproxy.Client` | 同じコントラクトを gRPC で提供し、WebSocket 上で運びます |
| `Store[R]` | 型付きのコレクション。`mvvm.ObservableList` として公開されます |

## ローカルでもリモートでも同じ答え {#the-same-answer-local-or-remote}

`Store` は `Proxy` とやり取りしますが、それがどちらなのかを知りません。`Apply` はレコードとクエリだけで決まる
純粋関数なので、クライアントとサーバーは同じ行に対して同じコードを実行します。そのため `MemoryProxy` と
`grpcproxy.Client` はバイト単位で同一のビューを返します。適合性テストはまさにそれを検証します。ソート、フィルタ、
グループ化、ページング、集計のクエリ一式を、2 つのプロキシとエンジンの両方に通し、すべての結果を正規化して、
3 つをバイト単位で比較します。

WebSocket トランスポートは `js/wasm` にコンパイルできるため、ブラウザアプリケーションも同じクライアントを通じて
同じサービスに到達できます。

```go
store := data.NewStore(mem, codec) // or a grpcproxy.Client: nothing else changes
store.SetQuery(data.Query{Sorts: []data.Sort{{Field: "salary", Desc: true}}, Limit: 20})
store.Load(ctx)
items := store.Items() // bind this to a view
```

`tkbind.BindTable` はストアをツールキットの `Table` の背後に置きます。ヘッダーのクリックでソートし、グループ化で
グループ化し、検証済みのセル編集はプロキシを通じたミューテーションになります。ローカルでもリモートでも、
同じラウンドトリップです。
