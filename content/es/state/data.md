---
title: "La columna vertebral de datos"
linkTitle: "Datos"
weight: 20
description: "go-widgets/data: registros tipados con validación, un motor de consultas puro y un almacén que ejecuta la misma cadena de procesamiento dentro del proceso o por gRPC, en nativo y en wasm."
tags: [gestión del estado, datos, grpc]
---

[`go-widgets/data`](https://github.com/go-widgets/data) no tiene interfaz:
importa la biblioteca estándar y [`mvvm`]({{< relref "/state/mvvm.md" >}}),
y ningún widget.

| Pieza | Qué es |
|---|---|
| `Value`, `Kind` | un escalar tipado comparable: string, int, float, bool |
| `Record`, `Schema`, `Field`, `Rule` | una fila tipada y su validación |
| `Query` → `Apply` → `View` | el motor de consultas: filtrar, ordenar, agrupar, paginar, agregar |
| `Proxy` | la costura del backend: `List`, `Query`, `Mutate` |
| `MemoryProxy` | el backend dentro del proceso |
| `grpcproxy.Server`, `grpcproxy.Client` | el mismo contrato por gRPC, transportado sobre un WebSocket |
| `Store[R]` | una colección tipada, publicada como `mvvm.ObservableList` |

## La misma respuesta, en local o en remoto {#the-same-answer-local-or-remote}

`Store` habla con un `Proxy` y nunca sabe con cuál. `Apply` es una función pura
de los registros y la consulta, de modo que el cliente y el servidor ejecutan el
mismo código sobre las mismas filas, y un `MemoryProxy` y un `grpcproxy.Client`
devuelven vistas idénticas byte a byte. La prueba de conformidad comprueba
exactamente eso: una batería de consultas de ordenación, filtrado, agrupación,
paginación y agregación a través de ambos proxies y del motor, cada resultado
canonicalizado, y los tres comparados byte a byte.

El transporte WebSocket compila para `js/wasm`, así que una aplicación de
navegador llega al mismo servicio a través del mismo cliente.

```go
store := data.NewStore(mem, codec) // or a grpcproxy.Client: nothing else changes
store.SetQuery(data.Query{Sorts: []data.Sort{{Field: "salary", Desc: true}}, Limit: 20})
store.Load(ctx)
items := store.Items() // bind this to a view
```

`tkbind.BindTable` coloca un almacén detrás de un `Table` del toolkit: un clic
en la cabecera ordena, una agrupación agrupa, y la edición validada de una celda
se convierte en una mutación a través del proxy: el mismo viaje de ida y vuelta,
en local o en remoto.
