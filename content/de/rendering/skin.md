---
title: "Skins: die Teile und Zustände eines Widgets als Daten"
linkTitle: "Skins"
weight: 30
description: "go-widgets/skin beschreibt die Teile eines Widgets, seine Darstellung pro Zustand und signalgesteuerte Übergänge in einer .skin.json-Datei, gerendert von einer einzigen Engine."
tags: [rendering, skin, theme]
---

Die Widgets des Toolkits sind in Go geschrieben: Die Teile eines `Button`, sein Aussehen bei Hover,
gedrückt und deaktiviert sowie sein Klick-Feedback liegen alle in `button.go`.
Das ist schnell und exakt, und jede visuelle Änderung ist eine Codeänderung.

[`go-widgets/skin`](https://github.com/go-widgets/skin) verlagert diese Beschreibung
in Daten – die Idee von Edje aus Enlightenment. Ein `.skin.json`-Dokument benennt die
**Teile** (parts) eines Widgets, ihr Aussehen in jedem **Zustand** (state) und die **Programme** (programs), die
sie zwischen Zuständen bewegen, wenn ein Signal eintrifft. Ein neuer Skin oder ein neues
Bedienelement braucht dann keine Neukompilierung: Das Programm lädt andere Bytes.

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

Ein `skin.Object` ist ein `toolkit.Widget` und sitzt deshalb in einer `VBox` neben
handgeschriebenen Widgets. Die Layouts platzieren Widgets; ein Skin ordnet an, was
in einem Widget liegt.

## Das Format {#the-format}

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

- Ein **Teil** ist ein `rect`, `text` oder `image`, platziert über zwei relative Ecken
  (`rel1`, `rel2`) gegenüber dem Objekt oder einem früheren Teil, mit Pixel- und
  schriftrelativen (`offset_em`) Versätzen, sodass ein Textteil der Schrift folgt.
- Ein **Zustand** setzt Farben, Rahmen, Radius und Sichtbarkeit und kann den Teil verschieben oder
  in der Größe ändern.
- Ein **Programm** reagiert auf ein Signal, bewegt Teile über eine Dauer hinweg mit einer benannten
  Easing-Funktion in einen Zustand und kann ein Signal nach außen senden (`emit`) – zum Beispiel an ein
  [`mvvm.Command`]({{< relref "/state/mvvm.md" >}}).
- Farben sind `@tokens`, aufgelöst gegen das `Theme` des Toolkits (einschließlich jedes
  GTK-`@define-color`-Eintrags als `@extra:<name>`), Hex-Werte oder RGBA-Arrays.

JSON wird von der Standardbibliothek dekodiert: keine Abhängigkeit von Dritten.

## Gegen die handgeschriebenen Widgets nachgewiesen {#proven-against-the-hand-written-widgets}

Button, Checkbox, Switch, Chip und Card des Toolkits sind jeweils noch einmal
als Skin-Collection geschrieben, und die Tests prüfen, dass beide in jedem Zustand **pixelgenau
identisch** rendern. Ein Benchmark misst ein Skin-`Draw` gegen das
handgeschriebene.
