---
title: "El estado vive en un view-model"
linkTitle: "Gestión del estado"
weight: 20
description: "Por qué el estado de la aplicación se guarda en observables enlazados a los widgets, en lugar de escribirse en los campos de los widgets."
tags: [gestión del estado, mvvm]
---

Los campos de un widget —el texto de una entrada, los elementos de una lista,
las filas de una tabla— son su **vista** del estado, no el estado. Una
aplicación que los asigna desde dondequiera que se encuentre dispersa su estado
entre los puntos de llamada, y dos de esos puntos acaban por no ponerse de
acuerdo sobre lo que hay en pantalla.

Por eso el estado vive en un **view-model**: valores Go corrientes envueltos en
observables, sin ningún widget a la vista, que se pueden probar sin ventana. La
vista enlaza cada widget al view-model una sola vez, y a partir de ahí es el
enlace —no la aplicación— el que los mantiene sincronizados, en ambos sentidos.

| | |
|---|---|
| [MVVM]({{< relref "/state/mvvm.md" >}}) | `Observable`, `Command`, `ObservableList`; binders para el toolkit y los widgets de terminal; deshacer y rehacer |
| [La columna vertebral de datos]({{< relref "/state/data.md" >}}) | registros tipados con validación, consultas y un almacén que ejecuta la misma cadena de procesamiento en local o por gRPC |
| [`mvvmlint`]({{< relref "/guard-rails/mvvmlint.md" >}}) | la comprobación de CI que rechaza una pull request que asigna a mano un campo de estado de un widget |
