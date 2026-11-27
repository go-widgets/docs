---
title: "Sprachen und Hell/Dunkel-Modus Ihrer Anwendung"
linkTitle: "Sprachen und Theme"
weight: 25
description: "toolkit v0.403.0: Catalog und NegotiateLocale für die Texte der Anwendung, i18ntest hält sie vollständig, ThemeManager für Dunkel, Hell und System."
tags: [rendering, toolkit, i18n, theme]
---

Seit toolkit v0.403.0 bietet das Paket drei kleine Bausteine für eine
Anwendung, die mehrere Sprachen spricht und ein helles und ein dunkles Aussehen
anbietet.

## Catalog

Die eigenen Wörter des Toolkits bleiben bei `SetLocalizer` und `Tr`. Ein
`Catalog` hält die Wörter, die die *Anwendung* schreibt: eine Tabelle
`Schlüssel -> Text` je Locale und eine Rückfall-Locale, die für die anderen
antwortet.

```go
cat := toolkit.NewCatalog("en").
	Add("en", map[string]string{"save": "Save", "count": "%d items"}).
	Add("fr", map[string]string{"save": "Enregistrer", "count": "%d éléments"})
loc := cat.Negotiate(browserLanguages) // "fr" für []string{"fr-CA", "en"}
toolkit.SetLocalizer(cat.Localizer(loc))
```

- `Add(locale, msgs)` fügt in die Tabelle der Locale ein und gibt den Katalog
  zurück, Aufrufe lassen sich also verketten. Tags werden kleingeschrieben mit
  `-` verglichen (`fr_CA` und `FR-ca` sind beide `fr-ca`).
- `Lookup(locale, key)` antwortet aus der Tabelle der Locale oder, bei einem
  regionalen Tag ohne Tabelle (`fr-ca`), aus der ihrer Sprache (`fr`); der
  Rückfall wird nicht benutzt, `ok` ist bei fehlendem Schlüssel false.
- `Text(locale, key)` ist `Lookup`, dann der Text der Rückfall-Locale, sonst
  `""`. Eine halb übersetzte Tabelle zeigt also Englisch, nie einen Schlüssel.
- `Localizer(locale)` ist die Funktion, die `SetLocalizer` erwartet.
- `Fallback()`, `Locales()` und `Keys()` listen den Inhalt sortiert auf.

## NegotiateLocale

`NegotiateLocale(requested, available, fallback)` wählt den Eintrag aus
`available`, der `requested` am besten beantwortet, der gewünschteste zuerst
(das `navigator.languages` eines Browsers). Für jeden angefragten Tag gewinnt
ein exakter Treffer (Groß-/Kleinschreibung egal, `_` als `-` gelesen), sonst
eine verfügbare Locale derselben Sprache (`fr-CA` wird von `fr` beantwortet,
`fr` von `fr-FR`); der erste angefragte Tag mit Antwort gewinnt. Keiner:
`fallback`. Ein `;q=`-Gewicht wird ohne seinen Wert gelesen.
`Catalog.Negotiate(requested)` tut dasselbe über die Locales des Katalogs.

## i18ntest

Ein halb übersetzter Katalog ist ein Fehler, den man nur in einer Sprache
sieht. `i18ntest.Check` macht daraus einen fehlschlagenden Test:

```go
func TestCatalogIsComplete(t *testing.T) { i18ntest.Check(t, catalog) }
```

Er schlägt einmal je Locale mit fehlenden Schlüsseln fehl (und nennt sie),
einmal je Übersetzung, deren `fmt`-Verben von denen des Rückfalls abweichen
(ein verlorenes oder vertauschtes `%d` gäbe Unsinn aus), und wenn der Katalog
gar keine Locale hat. `Missing()` und `VerbMismatches()` liefern dieselben
Listen für Ihren eigenen Code.

## ThemeManager

Eine `ThemeFamily` ist ein Theme in zwei Varianten (`Name`, `Light`, `Dark`).
`default`, `adwaita`, `fluent` und `whitesur` sind registriert;
`RegisterThemeFamily` fügt eigene hinzu, `ThemeFamilyNamed` und
`ThemeFamilies` lesen das Register. Ein `ThemeMode` ist die Wahl der
Nutzerin: `ThemeSystem` (Nullwert, folgt dem System), `ThemeLight` oder
`ThemeDark`; `String` liefert `"system"`, `"light"` oder `"dark"`, die zu
speichernde Form, `ParseThemeMode` liest sie zurück.

```go
m := toolkit.NewThemeManager(family) // ThemeSystem, System hell
m.Follow(colorScheme)                // ColorScheme: Dark() und Watch(f)
unsub := m.OnChange(func() { repaint(m.Theme()) })
m.SetMode(toolkit.ThemeDark)
```

`Theme()` baut die geltende Variante; `Dark()` sagt, welche es ist. `SetMode`
und `SetFamily` benachrichtigen die Abonnenten bei einer Änderung. `Follow`
liest ein `ColorScheme` (im Browser `prefers-color-scheme`, im Test ein Fake)
und folgt ihm live, `ThemeSystem` zeichnet also ohne Neuladen neu; `Close`
beendet das Folgen. Ein Manager ist nebenläufig sicher, und Abonnenten laufen
außerhalb seiner Sperre, dürfen ihn also wieder aufrufen.
