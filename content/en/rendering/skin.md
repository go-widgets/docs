---
title: "Skins: a widget's parts and states as data"
linkTitle: "Skins"
weight: 30
description: "go-widgets/skin describes a widget's parts, per-state visuals and signal-driven transitions in a .skin.json file, rendered by one engine."
tags: [rendering, skin, theme]
---

The toolkit's widgets are written in Go: a `Button`'s parts, its hover,
pressed and disabled faces and its click feedback all live in `button.go`.
That is fast and exact, and every visual change is a code change.

[`go-widgets/skin`](https://github.com/go-widgets/skin) moves that description
into data — the idea of Enlightenment's Edje. A `.skin.json` document names a
widget's **parts**, their look in each **state**, and the **programs** that
move them between states when a signal arrives. Re-skinning, or building a new
control, then needs no recompilation: the program loads different bytes.

```go
theme, err := skin.Load(jsonBytes) // parse and validate every collection
if err != nil {
	return err
}
obj, _ := theme.New("button")
obj.Bind(skin.NewMVVMContext().Set("$.label", labelObs))
obj.SetBounds(toolkit.Rect{W: 96, H: 28})

// each frame
obj.OnEvent(ev)               // pointer events become signals, signals states
busy := obj.Tick(time.Now())  // advance the transitions
obj.Draw(p, theme.Palette())
```

A `skin.Object` is a `toolkit.Widget`, so it sits in a `VBox` next to
hand-written widgets. The layouts place widgets; a skin arranges what is
inside one.

## The format

```jsonc
{
  "collections": {
    "button": {
      "min": { "w": 96, "h": 28 },
      "parts": [
        { "name": "bg", "type": "rect",
          "states": {
            "default": { "color": "@surface",     "border": "@border", "radius": 6 },
            "hover":   { "color": "@surface_alt", "border": "@border", "radius": 6 },
            "pressed": { "color": "@accent",      "border": "@border", "radius": 6 } } },
        { "name": "label", "type": "text", "text_from": "$.label", "align": [0.5, 0.5],
          "states": { "default": { "ink": "@on_surface" } } }
      ],
      "programs": [
        { "on": "mouse,in",    "target": ["bg", "label"], "to": "hover", "in": 0.12, "ease": "ease_out_cubic" },
        { "on": "mouse,click", "emit": "clicked" }
      ]
    }
  }
}
```

- A **part** is a `rect`, `text` or `image`, placed by two relative corners
  (`rel1`, `rel2`) against the object or an earlier part, with pixel and
  font-relative (`offset_em`) offsets, so a text part follows the font.
- A **state** sets colours, border, radius and visibility, and may move or
  resize the part.
- A **program** reacts to a signal, moves parts to a state over a duration
  with a named easing, and may `emit` a signal outwards — to an
  [`mvvm.Command`]({{< relref "/state/mvvm.md" >}}), for instance.
- Colours are `@tokens` resolved against the toolkit `Theme` (including every
  GTK `@define-color` entry as `@extra:<name>`), hex, or RGBA arrays.

JSON is decoded by the standard library: no third-party dependency.

## Proven against the hand-written widgets

The toolkit's button, check box, switch, chip and card are each written again
as a skin collection, and the tests assert that the two render **pixel for
pixel identically** in every state. A skinned `Draw` is measured against the
hand-written one in a benchmark.
