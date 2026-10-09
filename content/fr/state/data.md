---
title: "La colonne vertébrale des données"
linkTitle: "Données"
weight: 20
description: "go-widgets/data : des enregistrements typés avec validation, un moteur de requêtes pur, et un stockage qui exécute le même pipeline dans le processus ou via gRPC, en natif comme en wasm."
tags: [état, données, grpc]
---

[`go-widgets/data`](https://github.com/go-widgets/data) est sans interface : il
importe la bibliothèque standard et [`mvvm`]({{< relref "/state/mvvm.md" >}}),
et aucun widget.

| Élément | Ce que c'est |
|---|---|
| `Value`, `Kind` | un scalaire typé comparable : chaîne, entier, flottant, booléen |
| `Record`, `Schema`, `Field`, `Rule` | une ligne typée et sa validation |
| `Query` → `Apply` → `View` | le moteur de requêtes : filtrer, trier, grouper, paginer, agréger |
| `Proxy` | la jonction avec le back-end : `List`, `Query`, `Mutate` |
| `MemoryProxy` | le back-end dans le processus |
| `grpcproxy.Server`, `grpcproxy.Client` | le même contrat via gRPC, transporté sur une WebSocket |
| `Store[R]` | une collection typée, publiée sous forme d'`mvvm.ObservableList` |

## La même réponse, en local ou à distance {#the-same-answer-local-or-remote}

`Store` parle à un `Proxy` sans jamais savoir lequel. `Apply` est une fonction
pure des enregistrements et de la requête, si bien que le client et le serveur exécutent le
même code sur les mêmes lignes — et un `MemoryProxy` et un `grpcproxy.Client`
renvoient des vues identiques à l'octet près. Le test de conformité vérifie exactement cela : une
batterie de requêtes de tri, de filtre, de groupement, de pagination et d'agrégation à travers les deux
proxys et à travers le moteur, chaque résultat mis sous forme canonique, les trois
comparés octet par octet.

Le transport WebSocket se compile pour `js/wasm`, si bien qu'une application de navigateur
atteint le même service par le même client.

```go
store := data.NewStore(mem, codec) // or a grpcproxy.Client: nothing else changes
store.SetQuery(data.Query{Sorts: []data.Sort{{Field: "salary", Desc: true}}, Limit: 20})
store.Load(ctx)
items := store.Items() // bind this to a view
```

`tkbind.BindTable` place un stockage derrière une `Table` de la boîte à outils : un clic sur l'en-tête
trie, un regroupement groupe, et une modification de cellule validée devient une mutation via
le proxy — le même aller-retour, en local ou à distance.
