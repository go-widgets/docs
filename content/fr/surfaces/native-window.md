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
signale les rectangles qu'elle a repeints, et chaque back-end ne présente que
ceux-là : X11 via `ShmPutImage` de MIT-SHM, Wayland via les zones endommagées de `wl_shm`,
GTK4 via une mise à jour partielle de texture, Cocoa, Win32, Android, wasmbox et l'onglet
de navigateur. La première image, un redimensionnement et un `Expose` X11 présentent
encore toute la surface.

Sous GTK4, cela demande libgtk-4 4.16 ou plus récent : l'image devient une texture
`GdkMemoryTextureBuilder` avec `set_update_texture` et `set_update_region`, si bien que GTK
ne refait le rendu et l'envoi que de la région endommagée. Un runtime plus ancien garde
l'envoi complet, choisi à l'exécution. Le tampon d'image vit dans de la mémoire GLib que la texture partage : une présentation ne copie
aucun octet côté processeur (auparavant l'image entière, 128000 octets pour 200x160, à chaque fois).
La fenêtre GTK4 suit les redimensionnements : l'image remplit la fenêtre, le tampon est réalloué,
une image entière est présentée à la nouvelle taille, puis les dégâts reprennent.

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
  `Rasterize` CPU sur un seul cœur en 1080p ; la fenêtre elle-même, avec sa copie dans le
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
| GTK4 | `gtk_widget_set_visible(true)` | `gtk_widget_set_visible(false)` | `gtk_window_present` |
| Android, wasmbox, onglet de navigateur | `ErrNotSupported` | `ErrNotSupported` | `ErrNotSupported` |

Le focus, c'est à la plateforme de l'accorder : avec la prévention du vol de focus, `Raise`
peut seulement marquer la fenêtre comme demandant l'attention, sans pouvoir le savoir
(un compositeur Wayland sous GTK4 fait exactement cela).

`Config.HideOnClose` est respecté par X11, Wayland, GTK4, macOS et Windows : le contrôle
de fermeture de la fenêtre la masque alors au lieu de la fermer, et `Show` ou `Raise`
la ramène.

## Menu contextuel et image sur les contrôles natifs {#context-menu-and-picture-on-native-controls}

`NativeControl.Menu` et `Image` / `ImageOnly` sont appliqués par chaque backend natif, pas
seulement Cocoa. Sous GTK4, le menu est un popover qu'ouvre un clic secondaire et l'image
se place dans le bouton, seule ou à côté du libellé. Sous Win32, le menu est un vrai `HMENU`
affiché avec `TrackPopupMenu` sur `WM_CONTEXTMENU`, et l'image est un bitmap sur le bouton.
Une entrée désactivée ou un séparateur ne peut pas être choisi. L'un et l'autre ne sont
réappliqués que lorsqu'ils ont changé, car les descripteurs sont reconstruits à chaque image.

Le PNG est borné : 1 Mio et 1024 pixels de côté au plus, vérifiés d'après l'en-tête avant
d'allouer le moindre pixel ; une image illisible laisse le bouton tel qu'il était.
`painter.IconSize` et `painter.IconBGRAOver` l'ajustent au bouton moins une marge de
4 pixels, sans jamais l'agrandir. Sous Windows, l'image à côté du libellé demande les
contrôles communs à styles visuels ; sans manifeste, le bouton n'affiche que l'image.
Les champs de saisie Windows gardent leur propre menu système.

## Contrôles natifs sous Windows {#native-controls-on-windows}

Un `toolkit.NativeControl` devient une vraie fenêtre fille de la fenêtre Win32,
une par `Key`, réconciliée à chaque image : une valeur n'est poussée dans un
contrôle que si elle diffère de ce qu'il a rapporté en dernier, de sorte que les
saisies de la personne ne sont jamais combattues. Outre les types texte, bouton,
curseur, progression et incrémenteur, cinq autres sont hébergés :

| Type | Classe de fenêtre | Valeur |
|---|---|---|
| `NativeList` | `LISTBOX` | `Number` est la ligne choisie, `-1` si aucune |
| `NativeLink` | `SysLink` | `OnActivate` est appelé au clic ou sur Entrée |
| `NativeDate` | `SysDateTimePick32` | `Text` est `AAAA-MM-JJ` |
| `NativeColor` | `GoWidgetsColorWell` | `Text` est `#RRGGBB` ; un clic ouvre la boîte de couleur du système |
| `NativeSegmented` | `SysTabControl32` | `Text` est le titre du segment choisi |

Les lignes d'une liste et les segments sont fixés à la création du contrôle ;
pour les changer, changez la `Key`. Un lien n'ouvre jamais rien lui-même : il
appelle seulement `OnActivate`, de sorte que les adresses suivies (http et https
uniquement) restent la décision de `application.OpenURL`. `SysLink` n'existe que
dans les contrôles communs à styles visuels, qu'un programme sans manifeste n'obtient
pas ; la fenêtre crée le lien sous un manifeste à elle et, s'il n'est pas
disponible, affiche un bouton plat qui s'active de la même façon.

## Tableaux, arbres et onglets natifs

Trois types de plus portent des données structurées. On les construit avec
`NewNativeTable(columns, rows, selected)`, `NewNativeTree(nodes, selectedPath)`
et `NewNativeTabs(titles, selected)` ; `NativeControl` a gagné `Columns`,
`Rows`, `Tree` et `Revision` pour eux, et `Native.SetRows` remplace ensuite les
lignes d'un tableau.

| Type | Cocoa | GTK4 | Win32 | Sélection |
|---|---|---|---|---|
| `NativeTable` | `NSTableView` | `GtkColumnView` | `SysListView32` (`LVS_REPORT`, `LVS_OWNERDATA`) | `Number` / `OnNumber` est la ligne, `-1` pour aucune |
| `NativeTree` | `NSOutlineView` | `GtkListView` sur un `GtkTreeListModel` | `SysTreeView32` | `Text` / `OnText` est un chemin comme `2.0.1` |
| `NativeTabs` | `NSTabView` | `GtkNotebook` | `SysTabControl32` | `Number` / `OnNumber` est l'onglet, `-1` pour aucun |

Le tri n'est pas implémenté : les lignes apparaissent dans l'ordre donné. Un
contrôle d'onglets n'est que la barre d'onglets ; le contenu sous un onglet est
disposé par la boîte à outils comme d'habitude. Les repliements peints
(`Table`, `TreeView` avec `HideRoot`, `TabBar`) dessinent les mêmes données là
où aucun contrôle natif n'est hébergé. Les moteurs de fenêtre qui hébergent ces
classes (hébergés par `go-widgets/window`) arrivent dans la prochaine version de
la fenêtre ; côté Cocoa, il faut les nouveaux types `GridView`, `OutlineView` et
`TabView` de `go-macos/appkit`.

Les données sont bornées : au plus `MaxNativeRows` (100000) lignes,
`MaxNativeColumns` (64) colonnes, `MaxNativeTreeDepth` (32) niveaux,
`MaxNativeTreeNodes` (100000) nœuds et `MaxNativeTabs` (64) onglets ; une valeur
plus grande est refusée, pas tronquée.

La réconciliation ne traite que les changements. L'image est indexée sur
`Native.Revision()` : tant que la révision ne change pas, les lignes ne sont
pas parcourues du tout, et elles ne sont poussées vers le contrôle que lorsqu'elle
bouge. Sur Cocoa, une image de 10 000 lignes coûte 160 us avec une révision
inchangée et 687 us quand les lignes changent (construction des 10 000 lignes
comprise) ; `SetGrid` seul prend 1,3 us, la source de données étant virtuelle.
Sur Win32, une révision inchangée coûte 0,26 ns et aucune allocation, contre
43,8 us pour parcourir 10 000 lignes (mesuré sur un M4 Max pour Cocoa, un cœur).
