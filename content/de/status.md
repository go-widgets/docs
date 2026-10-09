---
title: "Stand"
weight: 60
description: "Was auf einem echten Display, einem echten Gerät oder in einem echten Terminal nachgewiesen wurde; was nur kompiliert wird; und was die CI noch nicht abdeckt."
tags: [stand]
---

Diese Seiten wurden mit den Modulen in den unter
[Module]({{< relref "/modules.md" >}}) aufgeführten Tags abgeglichen. Wo das README eines Moduls und sein Code
sich widersprechen, folgen die Seiten dem Code.

## Live nachgewiesen {#proven-live}

Ein im Prozess getesteter Codec beweist den Codec. Dies sind die Stellen, an denen
die Plattform selbst gefragt wurde:

| Was | Wo es nachgewiesen wurde |
|---|---|
| X11-Fenster | CI, unter Xvfb: ein bekanntes Muster dargestellt, aufgenommen und abgetastet; ein Klick und eine Taste mit `xdotool` synthetisiert und das ausgelöste `toolkit.Event` geprüft; `Show`/`Hide`/`Raise` vom Server zurückgelesen |
| Cocoa-Fenster | eine macOS-CI-Spur: ein Fenster geöffnet und gerendert, Pixel abgetastet, ein Klick und eine Taste synthetisiert; `Show`/`Hide`/`Raise` von einer anderen Goroutine aus an ein echtes `NSWindow` gerichtet und von AppKit zurückgelesen |
| Win32-Fenster | eine Windows-11-arm64-VM: ein echtes Fenster, das eine `VBox`, ein `Label` und einen `Button` rendert, drei injizierte Klicks treiben seinen Zähler über die Fensterprozedur von 0 auf 3 |
| Wayland-Fenster | gegen einen Fake-Compositor, der die Map- und Unmap-Regeln von xdg-shell durchsetzt |
| wasmbox-Fenster | Headless-Chromium steuert den echten wasmdesk-Compositor: die komponierten Pixel zurückgelesen, ein echter Klick an das Widget geleitet |
| Android | ein Android-15-arm64-Gerät: Blits nur geschädigter Bereiche gemessen, memfd-Dirty-Memory gemessen, der Barrierefreiheitsbaum auf dem Gerät gebenchmarkt |
| macOS-Tray | eine echte macOS-Sitzung: Das Symbol verlässt die Menüleiste, wenn das Programm endet, kehrt zurück, wenn es startet, und öffnet sein Menü |
| Linux-Tray | CI gegen einen echten Session-Bus: Der Name wird beansprucht, das Menü beantwortet `GetLayout`, `Quit` zieht es zurück |
| Windows-Tray | das Symbol und sein Menü auf einer Windows-11-VM |
| Terminal-Demos | ein echtes pty in der CI: Tasten-Bytes gesendet, der gerenderte Frame geprüft |

## Kompiliert, nicht ausgeführt {#compiled-not-run}

- Win32 `Show`/`Hide`/`Raise`: Die Entscheidungstabelle ist per Unit-Test geprüft, die Aufrufe
  auf dem Thread des Fensters werden nur kompiliert.
- `Attach` des Windows-Trays.
- Die DOM-Schleife von `webcanvas` (`run_js.go`): hinter einem `js && wasm`-Build-Tag, also
  außerhalb der gemessenen Abdeckung; die Anwendungen, die sie nutzen, üben sie aus.
- Das native `Run` von `application`: Das Öffnen eines echten Fensters ist von seiner
  Coverage-Schranke ausgenommen; der Vertrag, die Ereignisübersetzung und der Bereitschaftszähler
  sind abgedeckt.

## Nicht umgesetzt {#not-done}

- Die Trays unter Windows und Linux ignorieren `MenuItem.Icon`: Eine Zeile mit einem Symbol
  wird ohne es gezeichnet.
- Das Wayland-Back-End von `window` kann kein `Raise`: xdg-shell hat keine solche Anfrage,
  und xdg-activation braucht ein Token aus der eigenen Eingabe des Benutzers.
- Android: nur `android/arm64`, weil es das einzige Android-Ziel ist, das Go
  ohne cgo linkt.
- `window.Screens` gibt unter `js/wasm` `ErrScreensUnsupported` zurück.

## Was die CI noch nicht abdeckt {#what-ci-does-not-cover-yet}

Gemessen an den Workflows jedes Repositorys in den oben genannten Tags:

| Modul | Lücke |
|---|---|
| `mvvmtk` | keine Coverage-Schranke; Cross-Builds nur für amd64 und arm64, auf sechs Betriebssystemen |
| `application` | Cross-Builds für Linux, macOS und Windows nur auf amd64 |
| `tray` | Cross-Builds für vier Paare aus Betriebssystem und Architektur |
| `mvvmlint` | kein Cross-Build |
| `bricolint` | sein Schritt „6 arches" baut fünf: loong64 fehlt |

`app-template` und `gallery` sind Browser-Anwendungen und bauen für `js/wasm`,
ihr einziges Ziel.
