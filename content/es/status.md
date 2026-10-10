---
title: "Estado"
weight: 60
description: "Qué se ha demostrado en una pantalla real, un dispositivo real o un terminal real; qué solo se compila; y qué no cubre todavía la CI."
tags: [estado]
---

Estas páginas se contrastaron con los módulos en las etiquetas indicadas en
[Módulos]({{< relref "/modules.md" >}}). Cuando el README de un módulo y su
código no coinciden, las páginas siguen al código.

## Demostrado en vivo {#proven-live}

Un códec probado dentro del proceso demuestra el códec. Estos son los casos en
los que se interrogó a la propia plataforma:

| Qué | Dónde se demostró |
|---|---|
| Ventana X11 | CI, bajo Xvfb: un patrón conocido presentado, capturado y muestreado; un clic y una tecla sintetizados con `xdotool` y el `toolkit.Event` despachado comprobado; `Show`/`Hide`/`Raise` releídos desde el servidor |
| Ventana Cocoa | una línea de CI en macOS: una ventana abierta y renderizada, píxeles muestreados, un clic y una tecla sintetizados; `Show`/`Hide`/`Raise` solicitados a un `NSWindow` real desde otra goroutine y releídos desde AppKit |
| Ventana Win32 | una VM Windows 11 arm64: una ventana real que renderiza un `VBox`, un `Label` y un `Button`, con tres clics inyectados que llevan su contador de 0 a 3 a través del procedimiento de ventana |
| Ventana Wayland | frente a un compositor simulado que aplica las reglas de mapeo y desmapeo de xdg-shell |
| Ventana wasmbox | Chromium headless controlando el compositor wasmdesk real: los píxeles compuestos releídos, un clic real encaminado al widget |
| Android | un dispositivo Android 15 arm64: copias limitadas a las zonas dañadas medidas, memoria sucia del memfd medida, el árbol de accesibilidad evaluado en el dispositivo |
| Bandeja de macOS | una sesión macOS real: el icono sale de la barra de menús cuando el programa se detiene, vuelve cuando arranca y abre su menú |
| Bandeja de Linux | CI frente a un bus de sesión real: el nombre se reclama, el menú responde a `GetLayout`, `Quit` lo retira |
| Bandeja de Windows | el icono y su menú en una VM Windows 11 |
| Demos de terminal | un pty real en la CI: bytes de teclas enviados, el fotograma renderizado comprobado |

## Compilado, no ejecutado {#compiled-not-run}

- `Show`/`Hide`/`Raise` de Win32: la tabla de decisión tiene pruebas unitarias;
  las llamadas en el hilo de la ventana solo se compilan.
- El `Attach` de la bandeja de Windows.
- El bucle DOM de `webcanvas` (`run_js.go`): tras una etiqueta de compilación
  `js && wasm`, por lo que queda fuera de la cobertura medida; lo ejercitan las
  aplicaciones que lo usan.
- El `Run` nativo de `application`: abrir una ventana real está excluido de su
  barrera de cobertura; el contrato, la traducción de eventos y el contador de
  disponibilidad están cubiertos.

## No hecho {#not-done}

- Las bandejas de Windows y Linux ignoran `MenuItem.Icon`: una fila con icono
  se dibuja sin él.
- El backend Wayland de `window` no puede hacer `Raise`: xdg-shell no tiene esa
  petición, y xdg-activation necesita un token procedente de la propia entrada
  del usuario.
- Android: solo `android/arm64`, porque es el único destino Android que Go
  enlaza sin cgo.
- `window.Screens` devuelve `ErrScreensUnsupported` en `js/wasm`.

## Lo que cubre la CI {#what-ci-does-not-cover-yet}

El 2026-10-10 se cerraron las cinco carencias que esta página enumeraba:

| Módulo | Antes | Ahora |
|---|---|---|
| `mvvmtk` | sin barrera de cobertura; solo amd64 y arm64 | exactamente la barrera del 100 % que usa el toolkit (ya estaba en el 100,0 %); Linux en las seis |
| `application` | Linux, macOS y Windows solo en amd64 | Linux en las seis; macOS y Windows en amd64 y arm64 |
| `tray` | cinco pares sistema operativo/arquitectura | Linux en las seis, macOS en dos, Windows en una |
| `mvvmlint` | sin compilación cruzada | Linux en las seis, macOS y Windows |
| `bricolint` | un paso «6 arches» que compilaba cinco | loong64 añadida |

Así, las diecisiete bibliotecas y herramientas se compilan ya de forma cruzada
para las seis arquitecturas Linux de 64 bits, y los diecinueve módulos tienen
una barrera del 100 % de cobertura de sentencias.
`app-template` y `gallery` son aplicaciones de navegador y se compilan para
`js/wasm`, que es su único destino.

## Seguridad {#security}

Medido con `govulncheck` (a nivel de símbolo: solo el código que el módulo
puede alcanzar) el 2026-10-10:

- 15 de los 19 módulos alcanzaban código vulnerable: en la biblioteca estándar
  de go1.27.1 (`html/template`, `net/http`, `crypto/tls`, `mime/multipart`;
  corregido en go1.27.2) y en `golang.org/x/net` v0.58.0 (`http2`, a través de
  gRPC en `data`; corregido en v0.60.0).
- La CI de todos los módulos compila ya con go1.27.2, y las actualizaciones de
  dependencias que traen `x/net` v0.60.0 son las próximas versiones.
- Los workflows de los 23 repositorios (31 archivos) se auditaron con
  `actionlint` y `wfaudit`: ningún disparador privilegiado, ningún permiso de
  escritura a nivel de workflow, ningún token olvidado en un checkout subido,
  ninguna expresión no confiable en un paso de shell. Todos los workflows
  declaran sus permisos, de modo que ninguno depende del token por defecto de
  un repositorio.
