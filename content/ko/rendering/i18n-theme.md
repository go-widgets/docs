---
title: "앱의 언어와 라이트/다크 모드"
linkTitle: "언어와 테마"
weight: 25
description: "toolkit v0.403.0: 애플리케이션 고유 문구를 위한 Catalog와 NegotiateLocale, 이를 완전하게 지키는 i18ntest, 다크·라이트·시스템을 고르는 ThemeManager."
tags: [rendering, toolkit, i18n, theme]
---

toolkit v0.403.0부터 여러 언어를 쓰고 라이트·다크 두 가지 모양을 제공하는 애플리케이션을 위해
작은 부품 세 개가 추가되었습니다.

## Catalog

toolkit 자체의 문구는 계속 `SetLocalizer`와 `Tr`이 맡습니다. `Catalog`는 *애플리케이션*이 쓰는
문구를 담습니다. 로케일마다 `키 -> 텍스트` 표 하나, 그리고 다른 로케일을 대신해 답하는
대체(fallback) 로케일입니다.

```go
cat := toolkit.NewCatalog("en").
	Add("en", map[string]string{"save": "Save", "count": "%d items"}).
	Add("fr", map[string]string{"save": "Enregistrer", "count": "%d éléments"})
loc := cat.Negotiate(browserLanguages) // []string{"fr-CA", "en"}이면 "fr"
toolkit.SetLocalizer(cat.Localizer(loc))
```

- `Add(locale, msgs)`는 해당 로케일의 표에 병합하고 카탈로그를 돌려주므로 호출을 이어 쓸 수 있습니다.
  태그는 소문자와 `-`로 비교됩니다(`fr_CA`와 `FR-ca`는 모두 `fr-ca`).
- `Lookup(locale, key)`는 그 로케일의 표에서, 표가 없는 지역 태그(`fr-ca`)라면 해당 언어의 표(`fr`)에서
  답합니다. 대체 로케일은 쓰지 않으며, 키가 없으면 `ok`는 false입니다.
- `Text(locale, key)`는 `Lookup`, 다음으로 대체 로케일의 텍스트, 없으면 `""`입니다. 번역이 덜 된
  표도 키가 아니라 영어를 보여 줍니다.
- `Localizer(locale)`은 `SetLocalizer`가 받는 함수입니다.
- `Fallback()`, `Locales()`, `Keys()`는 내용을 정렬해서 돌려줍니다.

## NegotiateLocale

`NegotiateLocale(requested, available, fallback)`은 `requested`(가장 원하는 것이 먼저, 브라우저의
`navigator.languages`)에 가장 잘 맞는 `available`의 항목을 고릅니다. 각 태그마다 정확히 일치하는 것
(대소문자 무시, `_`는 `-`로 읽음)이 우선이고, 없으면 같은 언어의 사용 가능한 로케일(`fr-CA`에는 `fr`,
`fr`에는 `fr-FR`)이 선택됩니다. 답을 얻은 첫 번째 태그가 이기며, 하나도 없으면 `fallback`입니다.
`;q=` 가중치는 값을 무시하고 읽습니다. `Catalog.Negotiate(requested)`는 카탈로그 자신의 로케일에
대해 같은 일을 합니다.

## i18ntest

번역이 덜 된 카탈로그는 한 언어에서만 보이는 결함입니다. `i18ntest.Check`는 이를 실패하는 테스트로
만듭니다.

```go
func TestCatalogIsComplete(t *testing.T) { i18ntest.Check(t, catalog) }
```

키가 빠진 로케일마다 한 번(키 이름을 밝혀서), `fmt` 동사가 대체 로케일과 다른 번역마다 한 번
(`%d`가 빠지거나 순서가 바뀌면 엉뚱한 출력이 나옵니다), 그리고 카탈로그에 로케일이 하나도 없을 때
실패합니다. 카탈로그의 `Missing()`과 `VerbMismatches()`는 같은 목록을 여러분의 코드에 돌려줍니다.

## ThemeManager

`ThemeFamily`는 두 변형(`Name`, `Light`, `Dark`)을 가진 테마입니다. `default`, `adwaita`, `fluent`,
`whitesur`가 등록되어 있고, `RegisterThemeFamily`로 직접 추가하며, `ThemeFamilyNamed`와
`ThemeFamilies`가 레지스트리를 읽습니다. `ThemeMode`는 사용자의 선택입니다. `ThemeSystem`(영값,
시스템을 따름), `ThemeLight`, `ThemeDark`. `String`은 저장용 형태인 `"system"`, `"light"`, `"dark"`를
돌려주고 `ParseThemeMode`가 다시 읽습니다.

```go
m := toolkit.NewThemeManager(family) // ThemeSystem, 시스템은 라이트
m.Follow(colorScheme)                // ColorScheme: Dark()와 Watch(f)
unsub := m.OnChange(func() { repaint(m.Theme()) })
m.SetMode(toolkit.ThemeDark)
```

`Theme()`은 현재 유효한 변형을 만들고, `Dark()`는 어느 쪽인지 알려 줍니다. `SetMode`와 `SetFamily`는
변경이 있을 때 구독자에게 알립니다. `Follow`는 `ColorScheme`(브라우저에서는 `prefers-color-scheme`,
테스트에서는 가짜)을 읽고 실시간으로 따라가므로 `ThemeSystem`은 새로 고침 없이 다시 그려집니다.
`Close`는 따라가기를 멈춥니다. 매니저는 여러 goroutine에서 안전하게 쓸 수 있고, 구독자는 잠금 밖에서
호출되므로 매니저를 다시 호출해도 됩니다.
