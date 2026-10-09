---
title: "Où elle peint"
linkTitle: "Surfaces"
weight: 30
description: "Toutes les surfaces sur lesquelles un arbre go-widgets peut s'afficher, et ce qui décide entre elles."
tags: [surfaces, back-ends]
---

| Surface | Module | Painter |
|---|---|---|
| [Une fenêtre native]({{< relref "/surfaces/native-window.md" >}}) sur X11, Wayland, macOS, Windows | `window` | pixels |
| [Une application]({{< relref "/surfaces/application.md" >}}) avec une icône de zone de notification et l'apparence du système | `application`, `tray` | pixels |
| [Un onglet de navigateur]({{< relref "/surfaces/browser.md" >}}) : un simple `<canvas>` | `webcanvas` | pixels |
| [Un bureau dans le navigateur]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}}) : une fenêtre du compositeur wasmdesk | `window` (wasmbox) | pixels |
| [Un terminal]({{< relref "/surfaces/terminal.md" >}}) | `tui` | cellules |
| [Un APK Android]({{< relref "/surfaces/android.md" >}}) | `android`, `window` | pixels |
| [Une image]({{< relref "/surfaces/snapshots.md" >}}) : SVG ou PNG | `svg` | pixels |

L'arbre de widgets ne change pas de l'une à l'autre. Ce qui change, c'est qui possède le
tampon et qui fournit les entrées — un serveur d'affichage, un navigateur, un terminal, un
hôte Java — et chaque back-end transforme ces entrées en un même `toolkit.Event`.

## Choisie selon ce qui est présent {#chosen-by-what-is-there}

`window.Open` ne prend aucun argument de back-end. Il examine l'environnement — la
plateforme pour laquelle il a été compilé, `$WAYLAND_DISPLAY`, `$DISPLAY`, le socket qu'exporte un
hôte Android — et choisit celui qui peut fonctionner. Un binaire compilé pour
Linux ouvre une fenêtre Wayland dans une session Wayland et une fenêtre X11 sous
X ; le même binaire Android contacte son hôte dans un APK et ouvre une fenêtre
Linux ordinaire sous Termux. Voir
[comment `Open` choisit]({{< relref "/surfaces/native-window.md#how-open-chooses" >}}).
