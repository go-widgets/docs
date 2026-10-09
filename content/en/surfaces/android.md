---
title: "Android"
linkTitle: "Android"
weight: 50
description: "go-widgets/android: an installable APK whose whole interface is laid out and painted by a CGO_ENABLED=0 Go process."
tags: [surfaces, android]
---

Android is the one platform where a CGO-free process cannot own a window: every
path to a drawable surface goes through JNI, and JNI needs cgo. There is no
wire protocol to speak instead, the way X11 and Wayland have one.

So [`go-widgets/android`](https://github.com/go-widgets/android) is two
processes:

| | |
|---|---|
| **Java host** (`host/`) | owns the `Activity`, the `SurfaceView`, touch, keys and the lifecycle. Blits pixels; knows nothing about widgets. |
| **Go application** (`cmd/gwapp`) | an ordinary `CGO_ENABLED=0 GOOS=android` executable. Owns layout, widgets, theme, hit-testing and focus, unchanged from every other back-end. |

The Java host stands where the X server stands on Linux. Pixels travel through
a **memfd** the application creates and hands to the host over the socket;
the Go side writes RGBA_8888, which is byte for byte Android's ARGB_8888, so
the blit is a copy with no conversion, of the damaged rectangle only. Input,
insets, IME text and the accessibility tree cross a framed protocol over an
abstract `LocalSocket`.

```go
c, err := android.Dial("my app", nil) // nil theme: toolkit.DefaultDark()
if errors.Is(err, android.ErrUnsupported) {
	return nil // not under a host
}
defer c.Close()
return c.Run(myWidgetTree())
```

`Client` satisfies `window`'s `Backend`, so an application moves between
Android and the desktop back-ends without changing a line above the window —
and [`window.Open`]({{< relref "/surfaces/native-window.md#how-open-chooses" >}})
picks it by itself when the host's socket is there.

## arm64 only

`android/arm64` is the only Android target Go links without cgo; `arm`,
`amd64` and `386` require external linking. CI asserts both halves — the one
that builds and the three that do not — so the day Go lifts the restriction,
the build says so.

## Accessibility

A screen reader would see the `SurfaceView` as one opaque rectangle. The host
gives it a virtual view hierarchy instead: one node per accessible widget,
with the `android.widget.*` class Android chooses its announcements from, the
text, the bounds, and an activation action. The tree is **pulled** when
something reads it, never pushed, so an app with no accessibility service
attached never builds one. An activation comes back as an ordinary click at
the element's centre, through the same code a touch takes.

## Touch and animation

Each touch sample reaches the tree as a touch event, then a mouse event: the
first for gesture-aware widgets, the second for the many that only listen for
clicks. Animated widgets advance on a frame loop that starts when something
begins animating and stops when nothing does.
