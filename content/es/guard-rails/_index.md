---
title: "Reglas que impone la CI"
linkTitle: "Reglas de CI"
weight: 40
description: "Dos comprobaciones go/analysis que una aplicación ejecuta como comprobaciones de estado obligatorias: el estado pasa por MVVM, y nada de interfaz dibujada a mano."
tags: [ci, lint]
---

Dos convenciones mantienen honesta la interfaz de una aplicación, y ambas se
irían erosionando, un atajo cómodo tras otro, si solo estuvieran escritas. Por
eso cada una es un analizador
[`go/analysis`](https://pkg.go.dev/golang.org/x/tools/go/analysis),
ejecutado mediante `go vet`, con un workflow reutilizable que una aplicación
marca como comprobación **obligatoria**.

| | Rechaza |
|---|---|
| [`mvvmlint`]({{< relref "/guard-rails/mvvmlint.md" >}}) | asignar a mano un campo de estado de un widget, y un paquete con widgets pero sin view-model |
| [`bricolint`]({{< relref "/guard-rails/bricolint.md" >}}) | primitivas del painter en el código de la aplicación, y un widget construido de nuevo en cada fotograma |

Ambas son conservadoras por construcción: una regla solo salta cuando el **tipo
estático** del receptor se resuelve, mediante `go/types`, a un tipo del toolkit o
del painter. Un campo o un método con el mismo nombre en un tipo no relacionado
nunca se señala. Las excepciones previstas son directivas explícitas en el
código, de modo que cada una queda escrita allí donde se hace.
