---
title: "Les langues et le mode clair/sombre de votre application"
linkTitle: "Langues et thème"
weight: 25
description: "toolkit v0.403.0 : Catalog et NegotiateLocale pour les textes de l'application, i18ntest pour les garder complets, ThemeManager pour Sombre, Clair et Système."
tags: [rendu, toolkit, i18n, thème]
---

Depuis toolkit v0.403.0, le paquet offre trois petites briques à une
application qui parle plusieurs langues et propose un aspect clair et un
aspect sombre.

## Catalog

Les mots du toolkit restent gérés par `SetLocalizer` et `Tr`. Un `Catalog`
contient les mots que l'*application* écrit : une table `clé -> texte` par
locale, et une locale de repli qui répond pour les autres.

```go
cat := toolkit.NewCatalog("en").
	Add("en", map[string]string{"save": "Save", "count": "%d items"}).
	Add("fr", map[string]string{"save": "Enregistrer", "count": "%d éléments"})
loc := cat.Negotiate(browserLanguages) // "fr" pour []string{"fr-CA", "en"}
toolkit.SetLocalizer(cat.Localizer(loc))
```

- `Add(locale, msgs)` fusionne dans la table de la locale et renvoie le
  catalogue, d'où le chaînage. Les étiquettes sont comparées en minuscules avec
  des `-` (`fr_CA` et `FR-ca` valent `fr-ca`).
- `Lookup(locale, key)` répond depuis la table de la locale ou, pour une
  étiquette régionale sans table (`fr-ca`), depuis celle de sa langue (`fr`) ;
  il n'utilise pas le repli, et `ok` est faux pour une clé absente.
- `Text(locale, key)` est `Lookup`, puis le texte de la locale de repli, sinon
  `""`. Une table à moitié traduite affiche donc de l'anglais, jamais une clé.
- `Localizer(locale)` est la fonction que prend `SetLocalizer`.
- `Fallback()`, `Locales()` et `Keys()` listent le contenu, triés.

## NegotiateLocale

`NegotiateLocale(requested, available, fallback)` choisit l'entrée de
`available` qui répond le mieux à `requested`, la plus souhaitée d'abord (le
`navigator.languages` d'un navigateur). Pour chaque étiquette demandée, une
correspondance exacte (casse ignorée, `_` lu comme `-`) l'emporte, sinon une
locale disponible de la même langue (`fr-CA` est servie par `fr`, `fr` par
`fr-FR`) ; la première étiquette demandée qui trouve réponse gagne. Aucune :
`fallback`. Un poids `;q=` est lu sans sa valeur. `Catalog.Negotiate(requested)`
fait de même sur les locales du catalogue.

## i18ntest

Un catalogue à moitié traduit est un défaut qu'on ne voit que dans une langue.
`i18ntest.Check` en fait un test qui échoue :

```go
func TestCatalogIsComplete(t *testing.T) { i18ntest.Check(t, catalog) }
```

Il échoue une fois par locale qui manque de clés (en les nommant), une fois par
traduction dont les verbes `fmt` diffèrent de ceux du repli (un `%d` perdu ou
déplacé afficherait n'importe quoi), et si le catalogue n'a aucune locale.
`Missing()` et `VerbMismatches()` donnent les mêmes listes à votre code.

## ThemeManager

Une `ThemeFamily` est un thème en deux variantes (`Name`, `Light`, `Dark`).
`default`, `adwaita`, `fluent` et `whitesur` sont enregistrées ;
`RegisterThemeFamily` ajoute la vôtre, `ThemeFamilyNamed` et `ThemeFamilies`
lisent le registre. Un `ThemeMode` est le choix de l'utilisateur :
`ThemeSystem` (valeur zéro, suit le système), `ThemeLight` ou `ThemeDark` ;
`String` donne `"system"`, `"light"` ou `"dark"`, la forme à stocker, et
`ParseThemeMode` la relit.

```go
m := toolkit.NewThemeManager(family) // ThemeSystem, système clair
m.Follow(colorScheme)                // ColorScheme : Dark() et Watch(f)
unsub := m.OnChange(func() { repaint(m.Theme()) })
m.SetMode(toolkit.ThemeDark)
```

`Theme()` construit la variante en vigueur ; `Dark()` dit laquelle c'est.
`SetMode` et `SetFamily` préviennent les abonnés quand quelque chose change.
`Follow` lit un `ColorScheme` (dans un navigateur, `prefers-color-scheme` ;
dans un test, un faux) et le suit en direct : `ThemeSystem` se repeint sans
rechargement ; `Close` arrête le suivi. Un gestionnaire est sûr entre
goroutines, et les abonnés s'exécutent hors de son verrou : ils peuvent le
rappeler.
