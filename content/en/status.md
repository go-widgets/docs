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

## What CI covers {#what-ci-does-not-cover-yet}

On 2026-10-10 the five gaps this page listed were closed:

| Module | Was | Now |
|---|---|---|
| `mvvmtk` | no coverage gate; amd64 and arm64 only | the exact 100% gate the toolkit uses (it stood at 100.0% already); Linux on all six |
| `application` | Linux, macOS and Windows on amd64 only | Linux on all six; macOS and Windows on amd64 and arm64 |
| `tray` | five OS/architecture pairs | Linux on all six, macOS on two, Windows on one |
| `mvvmlint` | no cross-build | Linux on all six, macOS and Windows |
| `bricolint` | a "6 arches" step that built five | loong64 added |

So all seventeen libraries and tools now cross-build for the six 64-bit
Linux architectures, and all nineteen gate at 100% statement coverage.
`app-template` and `gallery` are browser applications and build for `js/wasm`,
which is their only target.

## Security {#security}

Measured with `govulncheck` (symbol level: only code the module can reach) on
2026-10-10:

- 15 of the 19 modules reached vulnerable code: in the standard library of
  go1.27.1 (`html/template`, `net/http`, `crypto/tls`, `mime/multipart`; fixed
  in go1.27.2) and in `golang.org/x/net` v0.58.0 (`http2`, through gRPC in
  `data`; fixed in v0.60.0).
- Every module's CI now builds with go1.27.2, and the dependency updates that
  bring `x/net` v0.60.0 are the next releases.
- The workflows of all 23 repositories (31 files) were audited with
  `actionlint` and `wfaudit`: no privileged trigger, no write grant at the
  workflow level, no token left in an uploaded checkout, no untrusted
  expression in a shell step. Every workflow declares its permissions, so
  none depends on a repository's default token.
