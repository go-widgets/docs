---
title: "Regeln, die die CI durchsetzt"
linkTitle: "CI-Regeln"
weight: 40
description: "Zwei go/analysis-Prüfungen, die eine Anwendung als verpflichtende Statusprüfungen ausführt: Zustand über MVVM und keine handgezeichnete UI."
tags: [ci, lint]
---

Zwei Konventionen halten die UI einer Anwendung ehrlich, und beide würden Abkürzung für
Abkürzung erodieren, wenn sie nur aufgeschrieben wären. Deshalb ist jede ein
[`go/analysis`](https://pkg.go.dev/golang.org/x/tools/go/analysis)-Analyzer,
der über `go vet` läuft, mit einem wiederverwendbaren Workflow, den eine Anwendung als
**verpflichtende** Prüfung markiert.

| | Lehnt ab |
|---|---|
| [`mvvmlint`]({{< relref "/guard-rails/mvvmlint.md" >}}) | das Zuweisen eines Zustandsfelds eines Widgets von Hand und ein Paket mit Widgets, aber ohne View-Model |
| [`bricolint`]({{< relref "/guard-rails/bricolint.md" >}}) | Painter-Primitive im Anwendungscode und ein Widget, das in jedem Frame neu gebaut wird |

Beide sind von Grund auf konservativ: Eine Regel schlägt nur an, wenn der **statische
Typ** des Empfängers über `go/types` zu einem Toolkit- oder Painter-Typ aufgelöst wird.
Ein gleichnamiges Feld oder eine gleichnamige Methode eines fremden Typs wird nie gemeldet. Die
beabsichtigten Ausnahmen sind explizite Direktiven im Code, sodass jede dort
aufgeschrieben ist, wo sie gemacht wird.
