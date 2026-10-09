# docs

[![docs](https://github.com/go-widgets/docs/actions/workflows/docs.yml/badge.svg)](https://github.com/go-widgets/docs/actions/workflows/docs.yml)
[![License](https://img.shields.io/badge/license-BSD--3--Clause-0D9488?style=flat-square)](LICENSE)

**The go-widgets documentation site**, published at
<https://go-widgets.github.io/docs/>.

A [Hugo](https://gohugo.io/) site with the [tannevaled/hextra](https://github.com/tannevaled/hextra)
fork of the [Hextra](https://github.com/imfing/hextra) theme, laid out like the
[go-fileshare documentation](https://github.com/go-fileshare/docs), with
go-widgets' branding. The content is derived from the modules' code, not only
from their READMEs, and where the two disagree the pages follow the code: a
claim here should be traceable to something a module does.

## Versions

go-widgets is nineteen modules, each released on its own, so no module's
version can label the site. A version of the documentation is **its own minor
version**:

| Source | Published under |
| --- | --- |
| branch `main`, with `params.docs.version: v0.2.0` in `hugo.yaml` | `https://go-widgets.github.io/docs/0.2/` |
| the newest version | also `https://go-widgets.github.io/docs/latest/` |

`https://go-widgets.github.io/docs/` redirects to `latest/`. Each push to `main`
replaces the directory of the version it describes, and nothing else.

A patch (`v0.2.1`) updates `0.2` in place. Raise the minor (`v0.3.0`) when the
pages of the current version should stay readable as they were: `0.3` is then
published beside `0.2`, and `latest` moves to it.

The version selector in the navbar lists every version from
`docs/versions.json` and opens the same page in the version chosen; if it is
not there, the same page in English, else that version's home.

Version 0.1 was built with MkDocs and mike before this site moved to Hugo.
Its directory on `gh-pages` is kept as it was published, and it reads the same
`versions.json` (mike's format).

## Languages

English is the default and sits at the root of each version
(`/docs/<version>/surfaces/terminal/`). French, Spanish and German are under
their code (`/docs/<version>/fr/surfaces/terminal/`, `/es/`, `/de/`). Version
0.1 is English only, and the version selector falls back to its English page.
The language switch at the foot of the sidebar opens the same page in another
language, in the same version.

To add a language `<lang>`:

1. `content/<lang>/`: a translation of every page, with the same file names
   (that is how a page and its translation are paired). A translated heading
   keeps the English one's anchor as an explicit id
   (`## Comment Open choisit {#how-open-chooses}`), so the relrefs resolve in
   every language and links into a page keep their fragment.
2. `i18n/<lang>.yaml`: the site's own strings (copyright, version).
3. A `languages.<lang>` entry in `hugo.yaml`: label, `contentDir`, weight,
   `params.flag`, the translated `params.description`, and a `dateFormat`.
4. Its flag in `static/images/flags/`, from
   [lipis/flag-icons](https://github.com/lipis/flag-icons) (4x3, MIT).

Declare a language only with its content: a declared language without pages
is an empty entry in the switch. A change to an English page is a change to
its three translations too.

## Layout

| Path | What |
| --- | --- |
| `content/en/` | the pages; `_index.md` is a section's own page, `weight` orders the sidebar |
| `i18n/<lang>.yaml` | the site's strings, beside the theme's |
| `static/images/flags/` | the language switch's flags (lipis/flag-icons, MIT) |
| `hugo.yaml` | `baseURL`, the documentation's version, the languages, the navbar (projects › version › theme › search › GitHub), the brand mounts |
| `layouts/_partials/custom/version-select.html` | the version selector (reads `versions.json`) |
| `layouts/_partials/favicons.html` | the brand's favicons |
| `assets/css/custom.css` | the brand teal as Hextra's primary colour, and the table headers |
| `themes/hextra` | submodule: [tannevaled/hextra](https://github.com/tannevaled/hextra), pinned to a release tag |
| `branding` | submodule: [go-widgets/brand](https://github.com/go-widgets/brand), pinned to a release tag (the mark, its PNGs and ICO, and their LICENSE) |
| `scripts/publish-version.sh` | puts one build into a `gh-pages` checkout and rewrites `versions.json`, `latest` and the root redirect |

## Writing a page

Each page starts with a title, a one-sentence description, and tags:

```yaml
---
title: "A native window"
linkTitle: "Native window"
weight: 10
description: "…"
tags: [surfaces, window]
---
```

Link to other pages with `{{< relref "/surfaces/native-window.md#how-open-chooses" >}}`:
a broken reference fails the build, and a missing anchor fails the link check.
A heading's anchor is a URL other sites link to: when a heading's words
change, keep the old anchor with an explicit `{#id}`.

A number or a version written in a page is measured at a tag, and the page
says which: a README read at a later tag than it was written at is how the
landing page came to say toolkit v0.109.0 while the tag was v0.328.0.

The page history at the foot of each page (created, modified, by whom) comes
from git: `themes/hextra/scripts/page-history.sh` writes
`data/pagehistory.json` before the build.

## Working locally

```sh
git clone --recurse-submodules https://github.com/go-widgets/docs.git
cd docs
mkdir -p data && sh themes/hextra/scripts/page-history.sh > data/pagehistory.json
hugo server --baseURL http://localhost:1313/docs/0.2/
```

then open <http://localhost:1313/docs/0.2/>. Without a `versions.json` the
selector shows only the current version. Hugo 0.146 or later (CI uses the
version pinned in the workflow); no Python, no Node.

## Publication

`.github/workflows/docs.yml`:

| Job | When | What |
| --- | --- | --- |
| `build` | pull requests, `main` | builds with `--baseURL …/docs/<version>/`, checks internal links and anchors (lychee, offline), uploads the site |
| `links` | pull requests, `main` | fetches every external URL the pages name |
| `deploy` | `main` | `scripts/publish-version.sh` into `gh-pages`, then a commit and a push; one deploy at a time |

GitHub Pages serves the `gh-pages` branch.

## Licence

BSD-3-Clause.
