---
title: "Une fenêtre native"
linkTitle: "Fenêtre native"
weight: 10
description: "go-widgets/window : une seule API Open/Run sur X11, Wayland, Cocoa, Win32, GTK4, Android et wasmbox, toutes avec CGO_ENABLED=0."
tags: [surfaces, window, x11, wayland, macos, windows]
---

[`go-widgets/window`](https://github.com/go-widgets/window) ouvre une vraie fenêtre
et y fait tourner un arbre de widgets : mise en page, dessin, présentation, traduction des entrées en
`toolkit.Event`, et ainsi de suite jusqu'à la fermeture de la fenêtre.

```go
w, err := window.Open(window.Config{Title: "Demo"})
if err != nil {
	return err
}
defer w.Close()
return w.Run(root) // root is any toolkit.Widget
```

`Config` prend aussi la taille initiale en **points logiques** (zéro demande une
taille lisible par défaut), l'instance et la classe `WM_CLASS`, un `Theme`, et un
`RenderScale`.

## Comment Open choisit {#how-open-chooses}

| Plateforme et environnement | Back-end | Comment il atteint l'écran |
|---|---|---|
| Linux, `$GO_WIDGETS_GTK` défini | GTK4 | GTK possède la fenêtre ; le framebuffer est une `GtkPicture`, et les contrôles natifs sont de vrais widgets GTK posés au-dessus. Exige le runtime libgtk-4. |
| Linux, `$WAYLAND_DISPLAY` défini | Wayland | xdg-shell sur le socket unix du compositeur |
| Linux, sinon `$DISPLAY` | X11 | le protocole X11 de base sur le socket unix, avec MIT-SHM |
| macOS | Cocoa | `NSWindow` et `NSView` via [go-macos/objc](https://github.com/go-macos/objc) (purego) |
| Windows | Win32 | un `HWND` de premier niveau via des appels système à user32 et gdi32, `StretchDIBits` |
| Android, `$GW_ANDROID_SOCKET` défini | hôte Android | un protocole tramé vers [l'hôte Java]({{< relref "/surfaces/android.md" >}}), les pixels dans un memfd partagé |
| Android, sinon | Wayland ou X11 | un shell sous Termux a toujours un serveur d'affichage à contacter |
| `js/wasm` | wasmbox | un client du [compositeur wasmdesk]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}}) |
| tout le reste | — | `window.ErrUnsupported`, pour qu'une compilation croisée compile quand même |

Chaque back-end est `CGO_ENABLED=0`. Celui de X11 est le protocole de base écrit
de zéro sur le socket — pas de Xlib, pas de XCB — avec les deux ordres d'octets, le
cookie Xauthority, la correspondance des keysyms, `PutImage` découpé en tuiles sous la taille de
requête du serveur, et la voie rapide MIT-SHM, le segment partagé étant transmis par
`SCM_RIGHTS`. Windows est atteint par les DLL du processus lui-même avec
`syscall.NewLazyDLL` et une procédure de fenêtre `syscall.NewCallback` ; macOS
par le runtime Objective-C avec purego.

La partie indépendante de la plateforme de chaque back-end — correspondance des événements, calculs de
coordonnées, empaquetage des pixels, rectangles endommagés — vit dans un codec couvert à 100 %,
sur chaque système d'exploitation. La fine couche de liaison avec la plateforme est éprouvée en conditions réelles : voir
[État]({{< relref "/status.md" >}}).

## Seulement ce qui a changé {#only-what-changed}

Une racine qui implémente `DamageRenderer` (comme le fait `toolkit/scene.HostRoot`)
signale les rectangles qu'elle a repeints, et les back-ends Cocoa, Win32 et wasmbox
ne présentent que ceux-là.

## HiDPI {#hidpi}

Par défaut, un pixel du framebuffer correspond à un point logique : l'interface est mise en page
et peinte à une taille lisible, et le compositeur la suréchantillonne. Sous Windows,
la fenêtre déclare une prise en charge du DPI par moniteur et le système met à l'échelle l'image
logique vers la zone client physique. `RenderScale` demande un framebuffer à
la résolution réelle de l'écran, ce qui n'est correct que pour une racine qui se met en page
en pixels de l'appareil.

## Écrans, affichage et masquage {#screens-and-showing-and-hiding}

`window.Screens()` liste les écrans connectés, le principal en premier, en points
logiques, panneaux du bureau exclus, et peut être appelé avant `Open`.
Le nom d'un écran est celui de la dalle elle-même (`"DELL U2720Q"`, d'après son EDID), avec repli
sur le connecteur (`"HDMI-1"`) quand elle n'en publie aucun — et aussi quand deux
dalles connectées publient le même, car un nom qui ne permet pas de les distinguer
n'est pas un nom.

`window.Show`, `Hide` et `Raise` retirent de l'écran une fenêtre ouverte sans
la fermer, puis la ramènent — ce dont a besoin une [application de zone de notification]({{< relref "/surfaces/application.md#the-tray" >}}).

| Back-end | Show | Hide | Raise |
|---|---|---|---|
| X11 | `MapWindow` | retrait ICCCM, pour qu'elle quitte la barre des tâches | map, raise, `_NET_ACTIVE_WINDOW` |
| Wayland | nouveau commit | tampon nul et commit | `ErrNotSupported` : xdg-shell n'a pas de telle requête |
| macOS | `orderFront:` | `orderOut:` | activer l'application, `makeKeyAndOrderFront:` |
| Windows | `SW_SHOWNA` | `SW_HIDE` | `SW_RESTORE` et `SetForegroundWindow` |
| GTK, Android, wasmbox | `ErrNotSupported` | `ErrNotSupported` | `ErrNotSupported` |

Le focus, c'est à la plateforme de l'accorder : avec la prévention du vol de focus, `Raise`
peut seulement marquer la fenêtre comme demandant l'attention, sans pouvoir le savoir.
