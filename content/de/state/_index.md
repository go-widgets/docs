---
title: "Zustand lebt in einem View-Model"
linkTitle: "Zustand"
weight: 20
description: "Warum Anwendungszustand in Observables gehalten und an Widgets gebunden wird, statt in Widget-Felder geschrieben zu werden."
tags: [zustand, mvvm]
---

Die Felder eines Widgets – der Text eines Eingabefelds, die Einträge einer Liste, die Zeilen einer Tabelle – sind seine
**Sicht** auf den Zustand, nicht der Zustand selbst. Eine Anwendung, die sie von
irgendeiner Stelle aus zuweist, verstreut ihren Zustand über viele Aufrufstellen, und zwei
dieser Aufrufstellen sind sich irgendwann uneinig darüber, was auf dem Bildschirm steht.

Deshalb lebt der Zustand in einem **View-Model**: einfache Go-Werte, in Observables verpackt,
ohne ein Widget weit und breit, testbar ohne Fenster. Die View bindet jedes Widget
einmal an das View-Model, und von da an hält die Bindung – nicht die Anwendung –
beide synchron, in beide Richtungen.

| | |
|---|---|
| [MVVM]({{< relref "/state/mvvm.md" >}}) | `Observable`, `Command`, `ObservableList`; Binder für das Toolkit und die Terminal-Widgets; Rückgängig und Wiederherstellen |
| [Das Daten-Rückgrat]({{< relref "/state/data.md" >}}) | typisierte Datensätze mit Validierung, Abfragen und ein Speicher, der dieselbe Pipeline lokal oder über gRPC ausführt |
| [`mvvmlint`]({{< relref "/guard-rails/mvvmlint.md" >}}) | die CI-Prüfung, die einen Pull Request scheitern lässt, der ein Zustandsfeld eines Widgets von Hand zuweist |
