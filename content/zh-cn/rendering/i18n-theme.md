---
title: "应用的语言与浅色/深色模式"
linkTitle: "语言与主题"
weight: 25
description: "toolkit v0.403.0：用于应用自身文案的 Catalog 与 NegotiateLocale，保证其完整的 i18ntest，以及在深色、浅色、跟随系统之间选择的 ThemeManager。"
tags: [rendering, toolkit, i18n, theme]
---

自 toolkit v0.403.0 起，该包为支持多种语言、并提供浅色与深色外观的应用增加了三个小部件。

## Catalog

toolkit 自身的文字仍由 `SetLocalizer` 和 `Tr` 处理。`Catalog` 保存的是*应用*自己写的文字：每个
语言环境一张 `键 -> 文本` 表，外加一个替其他环境作答的后备语言环境。

```go
cat := toolkit.NewCatalog("en").
	Add("en", map[string]string{"save": "Save", "count": "%d items"}).
	Add("fr", map[string]string{"save": "Enregistrer", "count": "%d éléments"})
loc := cat.Negotiate(browserLanguages) // 对 []string{"fr-CA", "en"} 得到 "fr"
toolkit.SetLocalizer(cat.Localizer(loc))
```

- `Add(locale, msgs)` 合并到该语言环境的表并返回目录，因此可以链式调用。标签以小写和 `-` 比较
  （`fr_CA` 与 `FR-ca` 都是 `fr-ca`）。
- `Lookup(locale, key)` 从该语言环境自己的表作答；对没有表的地区标签（`fr-ca`），则从其语言的表
  （`fr`）作答。它不使用后备，键缺失时 `ok` 为 false。
- `Text(locale, key)` 先 `Lookup`，再取后备语言环境的文本，否则为 `""`。因此只翻译了一半的表显示
  英文，而不会显示键。
- `Localizer(locale)` 是 `SetLocalizer` 接受的函数。
- `Fallback()`、`Locales()` 与 `Keys()` 按顺序列出目录内容。

## NegotiateLocale

`NegotiateLocale(requested, available, fallback)` 从 `available` 中挑出最能满足 `requested`
（最想要的在前，如浏览器的 `navigator.languages`）的条目。对每个请求的标签，精确匹配（忽略大小写，
`_` 视为 `-`）优先，否则取同一语言的可用语言环境（`fr-CA` 由 `fr` 应答，`fr` 由 `fr-FR` 应答）；
第一个得到应答的请求标签获胜。都不匹配则返回 `fallback`。`;q=` 权重只读取标签、忽略其值。
`Catalog.Negotiate(requested)` 对目录自身的语言环境做同样的事。

## i18ntest

只翻译了一半的目录是只在某一种语言里才看得到的缺陷。`i18ntest.Check` 把它变成会失败的测试：

```go
func TestCatalogIsComplete(t *testing.T) { i18ntest.Check(t, catalog) }
```

每个缺少键的语言环境失败一次（并列出这些键），每条 `fmt` 动词与后备文本不一致的译文失败一次
（丢失或调换的 `%d` 会输出乱码），目录没有任何语言环境时也会失败。目录的 `Missing()` 与
`VerbMismatches()` 把同样的列表交给你自己的代码。

## ThemeManager

`ThemeFamily` 是带两种变体（`Name`、`Light`、`Dark`）的主题。`default`、`adwaita`、`fluent` 与
`whitesur` 已注册；`RegisterThemeFamily` 添加你自己的，`ThemeFamilyNamed` 与 `ThemeFamilies` 读取
注册表。`ThemeMode` 是用户的选择：`ThemeSystem`（零值，跟随系统）、`ThemeLight` 或 `ThemeDark`；
`String` 给出用于存储的形式 `"system"`、`"light"` 或 `"dark"`，`ParseThemeMode` 将其读回。

```go
m := toolkit.NewThemeManager(family) // ThemeSystem，系统为浅色
m.Follow(colorScheme)                // ColorScheme：Dark() 与 Watch(f)
unsub := m.OnChange(func() { repaint(m.Theme()) })
m.SetMode(toolkit.ThemeDark)
```

`Theme()` 构建当前生效的变体，`Dark()` 说明是哪一种。`SetMode` 与 `SetFamily` 在发生变化时通知订阅者。
`Follow` 读取一个 `ColorScheme`（浏览器中是 `prefers-color-scheme`，测试中是假实现）并实时跟随，
所以 `ThemeSystem` 无需重新加载即可重绘；`Close` 停止跟随。管理器可在多个 goroutine 中安全使用，订阅者
在其锁之外运行，因此可以回调管理器。
