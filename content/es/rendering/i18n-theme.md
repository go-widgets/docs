---
title: "Idiomas y modo claro/oscuro de su aplicación"
linkTitle: "Idiomas y tema"
weight: 25
description: "toolkit v0.403.0: Catalog y NegotiateLocale para los textos de la aplicación, i18ntest para mantenerlos completos, ThemeManager para Oscuro, Claro y Sistema."
tags: [rendering, toolkit, i18n, tema]
---

Desde toolkit v0.403.0 el paquete ofrece tres piezas pequeñas para una
aplicación que habla varios idiomas y ofrece un aspecto claro y otro oscuro.

## Catalog

Las palabras propias del toolkit siguen con `SetLocalizer` y `Tr`. Un
`Catalog` guarda las palabras que escribe la *aplicación*: una tabla
`clave -> texto` por locale y un locale de reserva que responde por los demás.

```go
cat := toolkit.NewCatalog("en").
	Add("en", map[string]string{"save": "Save", "count": "%d items"}).
	Add("fr", map[string]string{"save": "Enregistrer", "count": "%d éléments"})
loc := cat.Negotiate(browserLanguages) // "fr" para []string{"fr-CA", "en"}
toolkit.SetLocalizer(cat.Localizer(loc))
```

- `Add(locale, msgs)` fusiona en la tabla del locale y devuelve el catálogo,
  así que las llamadas se encadenan. Las etiquetas se comparan en minúsculas
  con `-` (`fr_CA` y `FR-ca` son ambas `fr-ca`).
- `Lookup(locale, key)` responde desde la tabla del locale o, para una
  etiqueta regional sin tabla (`fr-ca`), desde la de su idioma (`fr`); no usa
  la reserva, y `ok` es falso para una clave ausente.
- `Text(locale, key)` es `Lookup`, luego el texto del locale de reserva, si no
  `""`. Una tabla a medio traducir muestra inglés, nunca una clave.
- `Localizer(locale)` es la función que toma `SetLocalizer`.
- `Fallback()`, `Locales()` y `Keys()` listan el contenido, ordenado.

## NegotiateLocale

`NegotiateLocale(requested, available, fallback)` elige la entrada de
`available` que mejor responde a `requested`, la más deseada primero (el
`navigator.languages` de un navegador). Para cada etiqueta pedida gana una
coincidencia exacta (sin distinguir mayúsculas, `_` leído como `-`), si no un
locale disponible del mismo idioma (`fr-CA` lo responde `fr`, `fr` lo responde
`fr-FR`); gana la primera etiqueta pedida que obtiene respuesta. Ninguna:
`fallback`. Un peso `;q=` se lee sin su valor. `Catalog.Negotiate(requested)`
hace lo mismo sobre los locales del catálogo.

## i18ntest

Un catálogo a medio traducir es un fallo que solo se ve en un idioma.
`i18ntest.Check` lo convierte en una prueba que falla:

```go
func TestCatalogIsComplete(t *testing.T) { i18ntest.Check(t, catalog) }
```

Falla una vez por locale al que le faltan claves (nombrándolas), una vez por
traducción cuyos verbos `fmt` difieren de los de la reserva (un `%d` perdido o
reordenado imprimiría basura) y si el catálogo no tiene ningún locale.
`Missing()` y `VerbMismatches()` dan las mismas listas a su propio código.

## ThemeManager

Una `ThemeFamily` es un tema en dos variantes (`Name`, `Light`, `Dark`).
`default`, `adwaita`, `fluent` y `whitesur` están registradas;
`RegisterThemeFamily` añade las suyas, `ThemeFamilyNamed` y `ThemeFamilies`
leen el registro. Un `ThemeMode` es la elección del usuario: `ThemeSystem`
(valor cero, sigue al sistema), `ThemeLight` o `ThemeDark`; `String` da
`"system"`, `"light"` o `"dark"`, la forma que se guarda, y `ParseThemeMode`
la vuelve a leer.

```go
m := toolkit.NewThemeManager(family) // ThemeSystem, sistema claro
m.Follow(colorScheme)                // ColorScheme: Dark() y Watch(f)
unsub := m.OnChange(func() { repaint(m.Theme()) })
m.SetMode(toolkit.ThemeDark)
```

`Theme()` construye la variante vigente; `Dark()` dice cuál es. `SetMode` y
`SetFamily` avisan a los suscriptores cuando algo cambia. `Follow` lee un
`ColorScheme` (en un navegador, `prefers-color-scheme`; en una prueba, un
doble) y lo sigue en vivo, así que `ThemeSystem` se repinta sin recargar;
`Close` deja de seguirlo. Un gestor es seguro entre goroutines y los
suscriptores se ejecutan fuera de su cerrojo, por lo que pueden volver a
llamarlo.
