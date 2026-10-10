---
title: "État"
weight: 60
description: "Ce qui a été prouvé sur un vrai écran, un vrai appareil ou un vrai terminal ; ce qui est seulement compilé ; et ce que la CI ne couvre pas encore."
tags: [état]
---

Ces pages ont été vérifiées par rapport aux modules aux étiquettes listées dans
[Modules]({{< relref "/modules.md" >}}). Quand le README d'un module et son code
divergent, les pages suivent le code.

## Prouvé en conditions réelles {#proven-live}

Un codec testé dans le processus prouve le codec. Voici les endroits où l'on a interrogé
la plateforme elle-même :

| Quoi | Où cela a été prouvé |
|---|---|
| Fenêtre X11 | CI, sous Xvfb : un motif connu présenté, capturé et échantillonné ; un clic et une touche synthétisés avec `xdotool` et le `toolkit.Event` distribué vérifié ; `Show`/`Hide`/`Raise` relus auprès du serveur |
| Fenêtre Cocoa | une voie de CI macOS : une fenêtre ouverte et rendue, des pixels échantillonnés, un clic et une touche synthétisés ; `Show`/`Hide`/`Raise` demandés à une vraie `NSWindow` depuis une autre goroutine et relus auprès d'AppKit |
| Fenêtre Win32 | une VM Windows 11 arm64 : une vraie fenêtre rendant une `VBox`, un `Label` et un `Button`, trois clics injectés faisant passer son compteur de 0 à 3 via la procédure de fenêtre |
| Fenêtre Wayland | face à un faux compositeur qui impose les règles de map et d'unmap de xdg-shell |
| Fenêtre wasmbox | un Chromium headless pilotant le vrai compositeur wasmdesk : les pixels composités relus, un vrai clic acheminé jusqu'au widget |
| Android | un appareil Android 15 arm64 : copies limitées aux zones endommagées mesurées, mémoire sale du memfd mesurée, arbre d'accessibilité évalué sur l'appareil |
| Zone de notification macOS | une vraie session macOS : l'icône quitte la barre des menus quand le programme s'arrête, revient quand il démarre, et ouvre son menu |
| Zone de notification Linux | CI face à un vrai bus de session : le nom est réclamé, le menu répond à `GetLayout`, `Quit` le retire |
| Zone de notification Windows | l'icône et son menu sur une VM Windows 11 |
| Démos en terminal | un vrai pty en CI : octets de touches envoyés, image rendue vérifiée |

## Compilé, non exécuté {#compiled-not-run}

- `Show`/`Hide`/`Raise` sous Win32 : la table de décision est testée unitairement, les appels
  sur le thread de la fenêtre sont seulement compilés.
- L'`Attach` de la zone de notification Windows.
- La boucle DOM de `webcanvas` (`run_js.go`) : derrière une étiquette de compilation `js && wasm`, elle
  est donc hors de la couverture mesurée ; les applications qui l'utilisent l'exercent.
- Le `Run` natif d'`application` : l'ouverture d'une vraie fenêtre est exclue de sa
  barrière de couverture ; le contrat, la traduction des événements et le compteur de disponibilité
  sont couverts.

## Non fait {#not-done}

- Les zones de notification Windows et Linux ignorent `MenuItem.Icon` : une ligne avec une icône
  se dessine sans elle.
- Le back-end Wayland de `window` ne sait pas faire `Raise` : xdg-shell n'a pas de telle requête,
  et xdg-activation exige un jeton issu d'une action de l'utilisateur lui-même.
- Android : `android/arm64` uniquement, car c'est la seule cible Android que Go
  sait lier sans cgo.
- `window.Screens` renvoie `ErrScreensUnsupported` sur `js/wasm`.

## Ce que la CI couvre {#what-ci-does-not-cover-yet}

Le 2026-10-10, les cinq manques que cette page listait ont été comblés :

| Module | Avant | Maintenant |
|---|---|---|
| `mvvmtk` | pas de barrière de couverture ; amd64 et arm64 seulement | exactement la barrière de 100 % qu'utilise le toolkit (il était déjà à 100,0 %) ; Linux sur les six |
| `application` | Linux, macOS et Windows sur amd64 seulement | Linux sur les six ; macOS et Windows sur amd64 et arm64 |
| `tray` | cinq paires système/architecture | Linux sur les six, macOS sur deux, Windows sur une |
| `mvvmlint` | pas de compilation croisée | Linux sur les six, macOS et Windows |
| `bricolint` | une étape « 6 arches » qui en compilait cinq | loong64 ajouté |

Les dix-sept bibliothèques et outils font donc tous désormais une compilation croisée pour les six
architectures Linux 64 bits, et les dix-neuf ont tous une barrière à 100 % de couverture des instructions.
`app-template` et `gallery` sont des applications de navigateur et se compilent pour `js/wasm`,
qui est leur seule cible.

## Sécurité {#security}

Mesuré avec `govulncheck` (au niveau des symboles : seul le code que le module peut atteindre) le
2026-10-10 :

- 15 des 19 modules atteignaient du code vulnérable : dans la bibliothèque standard de
  go1.27.1 (`html/template`, `net/http`, `crypto/tls`, `mime/multipart` ; corrigé
  dans go1.27.2) et dans `golang.org/x/net` v0.58.0 (`http2`, via gRPC dans
  `data` ; corrigé dans v0.60.0).
- La CI de chaque module compile désormais avec go1.27.2, et les mises à jour de dépendances qui
  apportent `x/net` v0.60.0 sont les prochaines versions.
- Les workflows des 23 dépôts (31 fichiers) ont été audités avec
  `actionlint` et `wfaudit` : aucun déclencheur privilégié, aucune permission d'écriture au
  niveau du workflow, aucun jeton laissé dans un checkout téléversé, aucune
  expression non fiable dans une étape shell. Chaque workflow déclare ses permissions, si bien
  qu'aucun ne dépend du jeton par défaut d'un dépôt.
