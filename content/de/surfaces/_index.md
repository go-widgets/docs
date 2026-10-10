---
title: "Wohin es zeichnet"
linkTitle: "Oberflächen"
weight: 30
description: "Jede Oberfläche, auf der ein go-widgets-Baum angezeigt werden kann, und was zwischen ihnen auswählt."
tags: [oberflächen, back-ends]
---

| Oberfläche | Modul | Painter |
|---|---|---|
| [Ein natives Fenster]({{< relref "/surfaces/native-window.md" >}}) unter X11, Wayland, macOS, Windows | `window` | Pixel |
| [Eine Anwendung]({{< relref "/surfaces/application.md" >}}) mit Tray-Symbol und dem Erscheinungsbild des Systems | `application`, `tray` | Pixel |
| [Ein Browser-Tab]({{< relref "/surfaces/browser.md" >}}): ein einfaches `<canvas>` | `webcanvas` oder `window` | Pixel |
| [Ein Browser-Desktop]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}}): ein Fenster des wasmdesk-Compositors | `window` (wasmbox) | Pixel |
| [Ein Terminal]({{< relref "/surfaces/terminal.md" >}}) | `tui` | Zellen |
| [Eine Android-APK]({{< relref "/surfaces/android.md" >}}) | `android`, `window` | Pixel |
| [Ein Bild]({{< relref "/surfaces/snapshots.md" >}}): SVG oder PNG | `svg` | Pixel |

Der Widget-Baum ändert sich zwischen ihnen nicht. Was sich ändert, ist, wem der
Puffer gehört und wer die Eingaben liefert – ein Display-Server, ein Browser, ein Terminal, ein
Java-Host –, und jedes Back-End macht aus diesen Eingaben dasselbe `toolkit.Event`.

## Ausgewählt nach dem, was vorhanden ist {#chosen-by-what-is-there}

`window.Open` nimmt kein Back-End-Argument. Es betrachtet die Umgebung – die
Plattform, für die es gebaut wurde, `$WAYLAND_DISPLAY`, `$DISPLAY`, den Socket, den ein
Android-Host exportiert – und wählt dasjenige, das funktionieren kann. Ein für
Linux gebautes Binary öffnet unter einer Wayland-Sitzung ein Wayland-Fenster und unter
X ein X11-Fenster; dasselbe Android-Binary verbindet sich in einer APK mit seinem Host und öffnet unter Termux ein gewöhnliches
Linux-Fenster. Siehe
[wie `Open` auswählt]({{< relref "/surfaces/native-window.md#how-open-chooses" >}}).
