---
title: "Your app's languages and light/dark mode"
linkTitle: "Languages and theme"
weight: 25
description: "toolkit v0.403.0: Catalog and NegotiateLocale for the application's own words, i18ntest to keep them complete, ThemeManager for Dark, Light and System."
tags: [rendering, toolkit, i18n, theme]
---

Since toolkit v0.403.0 the package has three small pieces for an application
that speaks several languages and offers a light and a dark look.

## Catalog

The toolkit's own words stay with `SetLocalizer` and `Tr`. A `Catalog` holds
the words the *application* writes: one `key -> text` table per locale, and a
fallback locale that answers for the others.

```go
cat := toolkit.NewCatalog("en").
	Add("en", map[string]string{"save": "Save", "count": "%d items"}).
	Add("fr", map[string]string{"save": "Enregistrer", "count": "%d éléments"})
loc := cat.Negotiate(browserLanguages) // "fr" for []string{"fr-CA", "en"}
toolkit.SetLocalizer(cat.Localizer(loc))
```

- `Add(locale, msgs)` merges into that locale's table and returns the catalog,
  so calls chain. Tags are compared in lower case with `-` separators
  (`fr_CA` and `FR-ca` are both `fr-ca`).
- `Lookup(locale, key)` answers from the locale's own table or, for a regional
  tag with no table (`fr-ca`), from its language's (`fr`); it does not use the
  fallback, and `ok` is false for a missing key.
- `Text(locale, key)` is `Lookup`, then the fallback locale's text, else `""`.
  A half-translated table therefore shows English, never a key.
- `Localizer(locale)` is the function `SetLocalizer` takes.
- `Fallback()`, `Locales()` and `Keys()` list what the catalog holds, sorted.

## NegotiateLocale

`NegotiateLocale(requested, available, fallback)` picks the entry of
`available` that best answers `requested`, most wanted first (a browser's
`navigator.languages`). For each requested tag, an exact match (case
ignored, `_` read as `-`) wins, else an available locale of the same language
(`fr-CA` is answered by `fr`, `fr` by `fr-FR`); the first requested tag that is
answered wins. None: `fallback`. A `;q=` weight is read without its value.
`Catalog.Negotiate(requested)` is the same over the catalog's own locales.

## i18ntest

A catalog that is translated halfway is a bug you only see in one language.
`i18ntest.Check` turns it into a failing test:

```go
func TestCatalogIsComplete(t *testing.T) { i18ntest.Check(t, catalog) }
```

It fails once per locale that lacks keys (naming them), once per translation
whose `fmt` verbs differ from the fallback's (a dropped or reordered `%d`
would print garbage), and when the catalog has no locale at all. `Missing()`
and `VerbMismatches()` on the catalog give the same lists to your own code.

## ThemeManager

A `ThemeFamily` is a theme in its two variants (`Name`, `Light`, `Dark`).
`default`, `adwaita`, `fluent` and `whitesur` are registered;
`RegisterThemeFamily` adds your own, `ThemeFamilyNamed` and `ThemeFamilies`
read the registry. A `ThemeMode` is the viewer's choice: `ThemeSystem` (the
zero value, it follows the system), `ThemeLight` or `ThemeDark`; `String`
gives `"system"`, `"light"` or `"dark"`, the form to store, and
`ParseThemeMode` reads it back.

```go
m := toolkit.NewThemeManager(family) // ThemeSystem, system light
m.Follow(colorScheme)                // ColorScheme: Dark() and Watch(f)
unsub := m.OnChange(func() { repaint(m.Theme()) })
m.SetMode(toolkit.ThemeDark)
```

`Theme()` builds the variant in effect; `Dark()` says which it is. `SetMode`
and `SetFamily` tell the subscribers when something changed. `Follow` reads a
`ColorScheme` (in a browser, `prefers-color-scheme`; in a test, a fake) and
tracks it live, so `ThemeSystem` repaints without a reload; `Close` stops
following. A manager is safe for use from several goroutines, and subscribers
run outside its lock, so they may call back into it.
