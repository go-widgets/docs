---
title: "Status"
weight: 60
description: "What has been proven on a real display, a real device or a real terminal; what is only compiled; and what CI does not cover yet."
tags: [status]
---

These pages were checked against the modules at the tags listed under
[Modules]({{< relref "/modules.md" >}}). Where a module's README and its code
disagree, the pages follow the code.

## Proven live

A codec tested in-process proves the codec. These are the places where the
platform itself was asked:

| What | Where it was proven |
|---|---|
| X11 window | CI, under Xvfb: a known pattern presented, captured and sampled; a click and a key synthesised with `xdotool` and the dispatched `toolkit.Event` asserted; `Show`/`Hide`/`Raise` read back from the server |
| Cocoa window | a macOS CI lane: a window opened and rendered, pixels sampled, a click and a key synthesised; `Show`/`Hide`/`Raise` asked of a real `NSWindow` from another goroutine and read back from AppKit |
| Win32 window | a Windows 11 arm64 VM: a real window rendering a `VBox`, a `Label` and a `Button`, three injected clicks driving its counter from 0 to 3 through the window procedure |
| Wayland window | against a fake compositor that enforces xdg-shell's map and unmap rules |
| wasmbox window | headless Chromium driving the real wasmdesk compositor: the composited pixels read back, a real click routed to the widget |
| Android | an Android 15 arm64 device: damage-only blits measured, memfd dirty memory measured, the accessibility tree benchmarked on the device |
| macOS tray | a real macOS session: the icon leaves the menu bar when the program stops, returns when it starts, and opens its menu |
| Linux tray | CI against a real session bus: the name is claimed, the menu answers `GetLayout`, `Quit` withdraws it |
| Windows tray | the icon and its menu on a Windows 11 VM |
| Terminal demos | a real pty in CI: key bytes sent, the rendered frame asserted |

## Compiled, not run

- Win32 `Show`/`Hide`/`Raise`: the decision table is unit-tested, the calls
  on the window's thread are only compiled.
- The Windows tray's `Attach`.
- `webcanvas`'s DOM loop (`run_js.go`): behind a `js && wasm` build tag, so it
  is outside the measured coverage; the applications that use it exercise it.
- `application`'s native `Run`: opening a real window is excluded from its
  coverage gate; the contract, the event translation and the ready counter
  are covered.

## Not done

- The Windows and Linux trays ignore `MenuItem.Icon`: a row with an icon
  draws without it.
- `window`'s Wayland back-end cannot `Raise`: xdg-shell has no such request,
  and xdg-activation needs a token from the user's own input.
- Android: `android/arm64` only, because it is the only Android target Go
  links without cgo.
- `window.Screens` returns `ErrScreensUnsupported` on `js/wasm`.

## What CI does not cover yet

Measured from each repository's workflows at the tags above:

| Module | Gap |
|---|---|
| `mvvmtk` | no coverage gate; cross-builds amd64 and arm64 only, on six operating systems |
| `application` | cross-builds Linux, macOS and Windows on amd64 only |
| `tray` | cross-builds four OS/architecture pairs |
| `mvvmlint` | no cross-build |
| `bricolint` | its "6 arches" step builds five: loong64 is missing |

`app-template` and `gallery` are browser applications and build for `js/wasm`,
which is their only target.
