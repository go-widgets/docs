---
title: "Das Daten-Rückgrat"
linkTitle: "Daten"
weight: 20
description: "go-widgets/data: typisierte Datensätze mit Validierung, eine reine Abfrage-Engine und ein Speicher, der dieselbe Pipeline im Prozess oder über gRPC ausführt, nativ und in wasm."
tags: [zustand, daten, grpc]
---

[`go-widgets/data`](https://github.com/go-widgets/data) ist headless: Es
importiert die Standardbibliothek und [`mvvm`]({{< relref "/state/mvvm.md" >}}),
aber kein Widget.

| Baustein | Was es ist |
|---|---|
| `Value`, `Kind` | ein vergleichbarer typisierter Skalar: String, Int, Float, Bool |
| `Record`, `Schema`, `Field`, `Rule` | eine typisierte Zeile und ihre Validierung |
| `Query` → `Apply` → `View` | die Abfrage-Engine: filtern, sortieren, gruppieren, paginieren, aggregieren |
| `Proxy` | die Back-End-Schnittstelle: `List`, `Query`, `Mutate` |
| `MemoryProxy` | das Back-End im Prozess |
| `grpcproxy.Server`, `grpcproxy.Client` | derselbe Vertrag über gRPC, transportiert über einen WebSocket |
| `Store[R]` | eine typisierte Sammlung, veröffentlicht als `mvvm.ObservableList` |

## Dieselbe Antwort, lokal oder entfernt {#the-same-answer-local-or-remote}

`Store` spricht mit einem `Proxy` und weiß nie, mit welchem. `Apply` ist eine reine
Funktion der Datensätze und der Abfrage, sodass Client und Server denselben
Code auf denselben Zeilen ausführen – und ein `MemoryProxy` und ein `grpcproxy.Client`
liefern byte-identische Views. Genau das prüft der Konformitätstest: eine
Batterie von Sortier-, Filter-, Gruppierungs-, Paginierungs- und Aggregationsabfragen über beide
Proxys und über die Engine, jedes Ergebnis kanonisiert, alle drei
Byte für Byte verglichen.

Der WebSocket-Transport kompiliert nach `js/wasm`, sodass eine Browser-Anwendung
denselben Dienst über denselben Client erreicht.

```go
store := data.NewStore(mem, codec) // or a grpcproxy.Client: nothing else changes
store.SetQuery(data.Query{Sorts: []data.Sort{{Field: "salary", Desc: true}}, Limit: 20})
store.Load(ctx)
items := store.Items() // bind this to a view
```

`tkbind.BindTable` stellt einen Store hinter eine Toolkit-`Table`: Ein Klick auf die Kopfzeile
sortiert, ein Group-by gruppiert, und eine validierte Zellbearbeitung wird zu einer Mutation über
den Proxy – derselbe Hin- und Rückweg, lokal oder entfernt.
