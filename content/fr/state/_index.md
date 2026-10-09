---
title: "L'état vit dans un view-model"
linkTitle: "Gestion de l'état"
weight: 20
description: "Pourquoi l'état d'une application est conservé dans des observables liés aux widgets, plutôt qu'écrit dans les champs des widgets."
tags: [état, mvvm]
---

Les champs d'un widget — le texte d'un champ de saisie, les éléments d'une liste, les lignes d'un tableau — sont sa
**vue** de l'état, pas l'état. Une application qui les affecte depuis
l'endroit où elle se trouve disperse son état entre les sites d'appel, et deux de
ces sites d'appel finissent par ne plus s'accorder sur ce qui est à l'écran.

L'état vit donc dans un **view-model** : de simples valeurs Go enveloppées dans des observables,
sans aucun widget en vue, testables sans fenêtre. La vue lie chaque widget
au view-model une fois, et à partir de là c'est la liaison — pas l'application — qui
les garde synchronisés, dans les deux sens.

| | |
|---|---|
| [MVVM]({{< relref "/state/mvvm.md" >}}) | `Observable`, `Command`, `ObservableList` ; des binders pour les widgets de la boîte à outils et du terminal ; annuler et rétablir |
| [La colonne vertébrale des données]({{< relref "/state/data.md" >}}) | des enregistrements typés avec validation, des requêtes, et un stockage qui exécute le même pipeline en local ou via gRPC |
| [`mvvmlint`]({{< relref "/guard-rails/mvvmlint.md" >}}) | le contrôle de CI qui fait échouer une pull request affectant à la main un champ d'état d'un widget |
