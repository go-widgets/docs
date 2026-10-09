---
title: "Skins: las partes y los estados de un widget como datos"
linkTitle: "Skins"
weight: 30
description: "go-widgets/skin describe las partes de un widget, su aspecto en cada estado y las transiciones provocadas por señales en un archivo .skin.json, renderizado por un único motor."
tags: [renderizado, skin, tema]
---

Los widgets del toolkit están escritos en Go: las partes de un `Button`, sus
aspectos al pasar el puntero, al pulsarse y deshabilitado, y su respuesta al
clic viven todos en `button.go`. Es rápido y exacto, y cada cambio visual es un
cambio de código.

[`go-widgets/skin`](https://github.com/go-widgets/skin) traslada esa descripción
a datos: la idea del Edje de Enlightenment. Un documento `.skin.json` nombra las
**partes** de un widget, su aspecto en cada **estado** y los **programas** que
las hacen pasar de un estado a otro cuando llega una señal. Cambiar la skin, o
construir un control nuevo, ya no requiere recompilar: el programa carga otros
bytes.

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

Un `skin.Object` es un `toolkit.Widget`, así que se coloca en un `VBox` junto a
widgets escritos a mano. Los layouts colocan widgets; una skin organiza lo que
hay dentro de uno.

## El formato {#the-format}

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

- Una **parte** es un `rect`, un `text` o una `image`, situada mediante dos
  esquinas relativas (`rel1`, `rel2`) respecto al objeto o a una parte
  anterior, con desplazamientos en píxeles y relativos a la fuente
  (`offset_em`), de modo que una parte de texto sigue a la fuente.
- Un **estado** fija colores, borde, radio y visibilidad, y puede mover o
  redimensionar la parte.
- Un **programa** reacciona a una señal, lleva partes a un estado durante un
  tiempo con una curva de easing con nombre, y puede emitir (`emit`) una señal
  hacia fuera: hacia un [`mvvm.Command`]({{< relref "/state/mvvm.md" >}}), por
  ejemplo.
- Los colores son `@tokens` resueltos contra el `Theme` del toolkit (incluida
  cada entrada `@define-color` de GTK como `@extra:<name>`), hexadecimales o
  arrays RGBA.

El JSON lo decodifica la biblioteca estándar: ninguna dependencia de terceros.

## Contrastado con los widgets escritos a mano {#proven-against-the-hand-written-widgets}

El botón, la casilla de verificación, el interruptor, el chip y la tarjeta del
toolkit están reescritos cada uno como una colección de skin, y las pruebas
comprueban que ambos se renderizan **idénticos píxel a píxel** en todos los
estados. Un benchmark mide un `Draw` con skin frente al escrito a mano.
