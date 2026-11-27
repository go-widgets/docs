---
title: "アプリの言語とライト/ダークモード"
linkTitle: "言語とテーマ"
weight: 25
description: "toolkit v0.403.0: アプリ独自の文言のための Catalog と NegotiateLocale、それを完全に保つ i18ntest、ダーク・ライト・システムを選ぶ ThemeManager。"
tags: [rendering, toolkit, i18n, theme]
---

toolkit v0.403.0 から、複数の言語を話し、ライトとダークの外観を備えるアプリのために、
3 つの小さな部品が加わりました。

## Catalog

toolkit 自身の文言はこれまでどおり `SetLocalizer` と `Tr` が扱います。`Catalog` が持つのは
*アプリケーション*が書く文言です。ロケールごとの `キー -> テキスト` の表と、他のロケールの
代わりに答えるフォールバックのロケールから成ります。

```go
cat := toolkit.NewCatalog("en").
	Add("en", map[string]string{"save": "Save", "count": "%d items"}).
	Add("fr", map[string]string{"save": "Enregistrer", "count": "%d éléments"})
loc := cat.Negotiate(browserLanguages) // []string{"fr-CA", "en"} なら "fr"
toolkit.SetLocalizer(cat.Localizer(loc))
```

- `Add(locale, msgs)` はそのロケールの表にマージしてカタログを返すので、呼び出しを連鎖できます。
  タグは小文字・`-` 区切りで比較されます(`fr_CA` も `FR-ca` も `fr-ca`)。
- `Lookup(locale, key)` はそのロケール自身の表から、表のない地域タグ(`fr-ca`)なら言語の表(`fr`)から
  答えます。フォールバックは使わず、キーがなければ `ok` は false です。
- `Text(locale, key)` は `Lookup`、次にフォールバックのロケールのテキスト、なければ `""` です。
  翻訳が途中の表でもキーではなく英語が表示されます。
- `Localizer(locale)` は `SetLocalizer` が受け取る関数です。
- `Fallback()`、`Locales()`、`Keys()` は内容をソートして返します。

## NegotiateLocale

`NegotiateLocale(requested, available, fallback)` は、`requested`(最も希望するものが先頭。
ブラウザの `navigator.languages`)に最もよく合う `available` の要素を選びます。各タグについて、
完全一致(大文字小文字無視、`_` は `-` と読む)が優先され、なければ同じ言語の利用可能なロケール
(`fr-CA` には `fr`、`fr` には `fr-FR`)が選ばれます。答えが見つかった最初のタグが勝ちで、
どれも合わなければ `fallback` です。`;q=` の重みは値を無視して読みます。
`Catalog.Negotiate(requested)` はカタログ自身のロケールに対して同じことをします。

## i18ntest

翻訳が途中のカタログは、ある言語でしか見えない不具合です。`i18ntest.Check` はそれを失敗するテストにします。

```go
func TestCatalogIsComplete(t *testing.T) { i18ntest.Check(t, catalog) }
```

キーが足りないロケールごとに 1 回(キー名を示して)、`fmt` の書式動詞がフォールバックと異なる翻訳ごとに
1 回(`%d` の欠落や順序違いは文字化けや panic の元です)、そしてカタログにロケールが 1 つもないときに
失敗します。カタログの `Missing()` と `VerbMismatches()` は同じ一覧を自分のコードに返します。

## ThemeManager

`ThemeFamily` は 2 つの変種(`Name`、`Light`、`Dark`)を持つテーマです。`default`、`adwaita`、
`fluent`、`whitesur` が登録済みで、`RegisterThemeFamily` で独自のものを追加し、
`ThemeFamilyNamed` と `ThemeFamilies` でレジストリを読みます。`ThemeMode` は利用者の選択です。
`ThemeSystem`(ゼロ値、システムに従う)、`ThemeLight`、`ThemeDark`。`String` は保存用の形
`"system"`、`"light"`、`"dark"` を返し、`ParseThemeMode` が読み戻します。

```go
m := toolkit.NewThemeManager(family) // ThemeSystem、システムはライト
m.Follow(colorScheme)                // ColorScheme: Dark() と Watch(f)
unsub := m.OnChange(func() { repaint(m.Theme()) })
m.SetMode(toolkit.ThemeDark)
```

`Theme()` は有効な変種を作り、`Dark()` はどちらかを返します。`SetMode` と `SetFamily` は変化があったときに
購読者へ知らせます。`Follow` は `ColorScheme`(ブラウザでは `prefers-color-scheme`、テストでは偽物)を
読み、ライブで追従するので、`ThemeSystem` は再読み込みなしで再描画されます。`Close` で追従を止めます。
マネージャは複数の goroutine から安全に使え、購読者はロックの外で呼ばれるため、マネージャを呼び返せます。
