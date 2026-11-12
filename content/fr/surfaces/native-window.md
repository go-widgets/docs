---
title: "Une fenêtre native"
linkTitle: "Fenêtre native"
weight: 10
description: "go-widgets/window : une seule API Open/Run sur X11, Wayland, GTK4, Cocoa, Win32, Android, wasmbox et un onglet de navigateur, toutes avec CGO_ENABLED=0."
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
| `js/wasm`, dans wasmdesk | wasmbox | un client du [compositeur wasmdesk]({{< relref "/surfaces/browser.md#a-window-on-the-wasmdesk-desktop" >}}) |
| `js/wasm`, une page ordinaire | onglet de navigateur | un `<canvas>` de la page (`Config.Canvas`, par défaut `"screen"`) via `webcanvas` : pas de compositeur, pas de `SharedArrayBuffer`, pas d'isolation cross-origin |
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
signale les rectangles qu'elle a repeints, et chaque back-end sauf GTK4 ne présente que
ceux-là : X11 via `ShmPutImage` de MIT-SHM, Wayland via les zones endommagées de `wl_shm`,
Cocoa, Win32, Android, wasmbox et l'onglet de navigateur. La première image, un
redimensionnement et un `Expose` X11 présentent encore toute la surface.

## Sur le GPU {#on-the-gpu}

Sous macOS, `Config{GPU: true}` dessine les images sur le GPU.
- **Comment :** les widgets sont enregistrés par
  [gpupainter](https://github.com/go-widgets/gpupainter) dans une liste
  [go-gfx/gpu2d](https://github.com/go-gfx/gpu2d). Son moteur Metal dessine
  l'image et la présente dans une `CAMetalLayer`, sans que l'image revienne au
  CPU.
- **Exactitude :** les pixels sont ceux du chemin CPU, octet pour octet.
- **Coût :** une image entière coûte au processeur 0,59 ms au lieu de 1,58 ms
  en 960 × 640, et 0,99 ms au lieu de 5,96 ms en 1920 × 1200 (Apple M4 Max).
- **Repli :** une fenêtre avec un matériau translucide reste sur le chemin CPU,
  de même qu'une machine où Metal ne démarre pas. Ni l'un ni l'autre n'est une
  erreur. Sous Linux, voir ci-dessous ; les autres back-ends ignorent l'option.

Sous Linux (X11 et Wayland), `Config{GPU: true}` dessine les images avec Vulkan.
- **Comment :** le même enregistrement, dessiné par le moteur Vulkan de
  go-gfx/gpu2d. Ses pixels sont copiés dans le framebuffer de la fenêtre, qui est
  présenté comme toujours : MIT-SHM (ou `PutImage`) sous X11, `wl_shm` sous
  Wayland. L'image traverse le CPU une fois ; rien n'est encore partagé avec le
  serveur d'affichage par dma-buf.
- **Exactitude :** sur Vulkan logiciel (lavapipe), les pixels sont ceux du chemin
  CPU, octet pour octet, prouvé en direct sur les deux protocoles : la fenêtre
  relue depuis le serveur X, et le tampon `wl_shm` validé par la fenêtre, sont
  égaux à ceux du chemin CPU. go-gfx/gpu2d a mesuré son moteur Vulkan sur un GPU
  matériel (AMD Radeon PRO W7700, RADV) : `Draw` est 52 fois plus rapide que le
  `Rasterize` CPU en 1080p ; la fenêtre elle-même, avec sa copie dans le
  framebuffer, n'a pas été mesurée sur du matériel.
- **Repli :** quand Vulkan est indisponible, ou qu'une image est refusée (trop
  grande pour le périphérique, périphérique perdu), la fenêtre dessine sur le CPU
  et continue. `GPUError()` sur le back-end en donne la raison, et toute image
  suivante est une image CPU.
- **Coût (logiciel seulement) :** lavapipe est un rastériseur CPU, donc le chemin
  GPU coûte environ dix fois le temps CPU du chemin CPU : 25,8 ms au lieu de
  2,48 ms par image en 960 × 640, et 89,7 ms au lieu de 8,46 ms en 1920 × 1200
  (X11, lavapipe sur 16 threads dans un conteneur arm64 sur un Apple M4 Max). Ce
  que cela donne sur un vrai GPU n'est pas encore mesuré.

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
| GTK, Android, wasmbox, onglet de navigateur | `ErrNotSupported` | `ErrNotSupported` | `ErrNotSupported` |

Le focus, c'est à la plateforme de l'accorder : avec la prévention du vol de focus, `Raise`
peut seulement marquer la fenêtre comme demandant l'attention, sans pouvoir le savoir.
