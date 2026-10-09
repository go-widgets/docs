---
title: "Règles imposées par la CI"
linkTitle: "Règles de CI"
weight: 40
description: "Deux contrôles go/analysis qu'une application exécute comme vérifications de statut obligatoires : l'état passe par MVVM, et pas d'interface dessinée à la main."
tags: [ci, lint]
---

Deux conventions gardent honnête l'interface d'une application, et toutes deux s'éroderaient
un raccourci commode après l'autre si elles étaient seulement écrites quelque part. Chacune est donc un
analyseur [`go/analysis`](https://pkg.go.dev/golang.org/x/tools/go/analysis),
exécuté via `go vet`, avec un workflow réutilisable qu'une application marque comme vérification
**obligatoire**.

| | Refuse |
|---|---|
| [`mvvmlint`]({{< relref "/guard-rails/mvvmlint.md" >}}) | l'affectation à la main d'un champ d'état d'un widget, et un paquet qui a des widgets mais pas de view-model |
| [`bricolint`]({{< relref "/guard-rails/bricolint.md" >}}) | les primitives du painter dans le code applicatif, et un widget reconstruit à chaque image |

Tous deux sont prudents par construction : une règle ne se déclenche que lorsque le **type
statique** du récepteur se résout, via `go/types`, en un type de la boîte à outils ou du painter.
Un champ ou une méthode de même nom sur un type sans rapport n'est jamais signalé. Les
exceptions voulues sont des directives explicites dans le code, si bien que chacune est
écrite là où elle est faite.
