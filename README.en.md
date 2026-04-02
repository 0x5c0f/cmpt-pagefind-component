<!-- markdownlint-disable-file MD033 MD041 -->
<h1 align="center">cmpt-pagefind-component | FixIt</h1>

<div align="center">
  <p>A FixIt search replacement component powered by Pagefind, keeping the original search UI while switching the search engine.</p>
  English · <a href="./README.md">简体中文</a>
</div>

## Overview

`cmpt-pagefind-component` is a FixIt theme component that:

- takes over FixIt search when `params.search.type = "pagefind"`
- keeps the original FixIt search entry and dropdown UI appearance
- replaces the underlying search engine (`fuse` / others) with Pagefind (browser API)
- exposes Pagefind options via `params.search.pagefind.*`

## Features

- `basic` mode by default, close to FixIt's original search behavior
- `advanced` mode with Pagefind debounced search and advanced options
- keeps the existing FixIt config namespace: `params.search.*`
- auto-applies `hidden` / `encrypted` filters via `data-pagefind-filter`
- supports multilingual sites (uses index based on current page `html[lang]`)

## Requirements

- FixIt `v0.4.0` or higher
- Hugo Extended `v0.156.0` or higher

> Pagefind indexing runs after Hugo build as an extra step. It is intentionally not tightly coupled to Hugo's build pipeline.

## Install

### Install as a Hugo Module

Make sure your site is already initialized as a Hugo module, then add this in your site `hugo.toml`:

```toml
[module]

[[module.imports]]
path = "github.com/hugo-fixit/FixIt"

[[module.imports]]
path = "github.com/hugo-fixit/cmpt-pagefind-component"
```

Update modules:

```bash
hugo mod get -u
hugo mod tidy
```

### Install as Git submodule

```bash
git submodule add https://github.com/hugo-fixit/FixIt.git themes/FixIt
git submodule add https://github.com/hugo-fixit/cmpt-pagefind-component.git themes/cmpt-pagefind-component
```

Enable themes in your site `hugo.toml`:

```toml
theme = ["FixIt", "cmpt-pagefind-component"]
```

## Configuration

### 1) Enable search and switch to Pagefind

```toml
[params.search]
enable = true
type = "pagefind"
placeholder = "Search by title or content"
maxResultLength = 10
```

### 2) Inject component partial (required)

The component does not automatically rewrite your site `params.customPartials.head`.
Add this explicitly in site config:

```toml
[params.customPartials]
head = ["inject/cmpt-pagefind-component.fixit.html"]
```

If you already have multiple head partials, append it instead of replacing:

```toml
[params.customPartials]
head = [
  "inject/xxx.fixit.html",
  "inject/cmpt-pagefind-component.fixit.html",
  "inject/yyy.fixit.html"
]
```

### 3) Configure Pagefind behavior (optional)

```toml
[params.search.pagefind]
mode = "basic"              # basic | advanced
bundlePath = "/pagefind/"   # pagefind index directory
maxResultLength = 6         # optional: override params.search.maxResultLength
debounceTimeoutMs = 300     # debounce in advanced mode
useBuiltInFilters = true    # auto-add hidden=false / encrypted=false
excerptLength = 80          # example custom excerpt length
```

Optional advanced blocks:

```toml
[params.search.pagefind.filters]
# category = "Tech"

[params.search.pagefind.sort]
# date = "desc"

[params.search.pagefind.ranking]
# termSaturation = 1.2
```

### Parameter reference

#### `params.search.*` (FixIt namespace)

| Key | Type | Default | Description |
|---|---|---:|---|
| `params.search.enable` | `bool` | `false` | Enables search entry |
| `params.search.type` | `string` | - | Set to `"pagefind"` to let this component take over |
| `params.search.placeholder` | `string` | Theme default | Input placeholder text |
| `params.search.maxResultLength` | `int` | `10` | Max dropdown items; `0` means no result list |
| `params.search.snippetLength` | `int` | `30` | Default source for `excerptLength`. Used when `params.search.pagefind.excerptLength` is not set |

#### `params.search.pagefind.*` (component config)

| Key | Type | Default | Description |
|---|---|---:|---|
| `mode` | `string` | `"basic"` | Search mode: `basic` or `advanced` |
| `bundlePath` | `string` | `"/pagefind/"` | Pagefind index directory; component loads `${bundlePath}pagefind.js` |
| `maxResultLength` | `int` | Reuse `params.search.maxResultLength` (default `10`) | Result limit for Pagefind. Can override global `params.search.maxResultLength` |
| `debounceTimeoutMs` | `int` | `300` | Debounce interval for `advanced` mode; `0` disables debounce |
| `useBuiltInFilters` | `bool` | `true` | Auto-tries `hidden=false` and `encrypted=false` when these filter fields exist in index |
| `excerptLength` | `int` | Reuses `params.search.snippetLength` when unset (default `30`) | Dropdown excerpt length. If explicitly set, this value takes precedence. `0` usually means title-only results |
| `baseUrl` | `string` | `""` | Advanced option for subpath deployment. Misconfiguration can break result URLs |
| `highlightParam` | `string` | `""` | Passed to `pagefind.options({ highlightParam })` |
| `filters` | `map[string]any` | `{}` | Passed to `pagefind.search(..., { filters })` |
| `sort` | `map[string]any` | `{}` | Passed as `sort` |
| `ranking` | `map[string]any` | `{}` | Passed as Pagefind `ranking` |
| `indexWeight` | `map[string]any` | `{}` | Passed as Pagefind `indexWeight` |
| `mergeFilter` | `map[string]any` | `{}` | Passed as Pagefind `mergeFilter` |

The component `hugo.toml` includes major defaults and commented examples for advanced keys. Override in site config only when needed.

> Notes:
> `ranking` supports common key alias normalization (for example `termsaturation` -> `termSaturation`).
> `indexWeight` supports both object and numeric forms.
> Do not set `baseUrl` equal to `bundlePath` (for example both `/pagefind/`), or result URLs may become incorrect (such as `/pagefind/posts/...`).

### Parameter precedence (important)

Actual value resolution order:

| Parameter | Precedence (high -> low) | Current out-of-box value |
|---|---|---|
| `excerptLength` | `params.search.pagefind.excerptLength` -> `params.search.snippetLength` -> `30` | Reuses `snippetLength` by default (FixIt default `30`) |
| `maxResultLength` | `params.search.pagefind.maxResultLength` -> `params.search.maxResultLength` -> `10` | Reuses `params.search.maxResultLength` by default (FixIt default `10`) |
| `debounceTimeoutMs` | `params.search.pagefind.debounceTimeoutMs` -> `300` | `300` |
| `bundlePath` | `params.search.pagefind.bundlePath` -> `"/pagefind/"` | `"/pagefind/"` |
| `useBuiltInFilters` | `params.search.pagefind.useBuiltInFilters` -> `true` | `true` |

For `excerptLength`: the component no longer sets a standalone default value. It reuses `params.search.snippetLength` first, and only falls back to `30` if `snippetLength` is also unset.

### Pass-through config examples

```toml
[params.search.pagefind]
# sync matched keywords into URL query for result-page highlighting
highlightParam = "highlight"

[params.search.pagefind.filters]
# search only in specific categories/tags
category = "Tech"
tags = ["hugo", "fixit"]
# exclude encrypted content if this field exists in your index
encrypted = "false"

[params.search.pagefind.sort]
# sort by date descending (field must exist in index)
date = "desc"

[params.search.pagefind.ranking]
# ranking fine-tuning
termSaturation = 1.2
termSimilarity = 1.0
pageLength = 0.8

[params.search.pagefind.indexWeight]
# field-level boosting
title = 8
tags = 3
# or a single numeric weight
# value = 1

[params.search.pagefind.mergeFilter]
# merge strategy for multiple filters: and / or (Pagefind semantics)
operator = "and"
```

> Recommendation: start with minimal config, then enable advanced options one by one to make behavior changes easier to verify.

### Minimal recommended config

```toml
[params.search]
enable = true
type = "pagefind"

[params.customPartials]
head = ["inject/cmpt-pagefind-component.fixit.html"]
```

### Common setup examples

#### A) Keep defaults (recommended start)

```toml
[params.search]
enable = true
type = "pagefind"

[params.customPartials]
head = ["inject/cmpt-pagefind-component.fixit.html"]
```

#### B) Advanced mode + custom list length and excerpt

```toml
[params.search]
enable = true
type = "pagefind"
maxResultLength = 12

[params.search.pagefind]
mode = "advanced"
debounceTimeoutMs = 200
excerptLength = 80

[params.customPartials]
head = ["inject/cmpt-pagefind-component.fixit.html"]
```

#### C) Subpath deployment (only when needed)

```toml
[params.search]
enable = true
type = "pagefind"

[params.search.pagefind]
bundlePath = "/blog/pagefind/"
baseUrl = "/blog/"

[params.customPartials]
head = ["inject/cmpt-pagefind-component.fixit.html"]
```

> `baseUrl` is an advanced option. Prefer leaving it empty unless your site is deployed under a subpath.

### About component `hugo.toml`

Keeping `hugo.toml` in this component repo is intentional:

1. It declares minimum runtime requirements (`Hugo Extended >= 0.156.0`).
2. It provides component defaults (`mode`, `bundlePath`, etc.) to reduce manual config; `excerptLength` reuses FixIt `snippetLength` by default.

It does not forcibly override site config. Site-level values have higher priority.

## Build and indexing

Pagefind indexing must run after Hugo build.

### Default recommendation: binary workflow (fits Hugo sites)

1. Build Hugo site:

```bash
hugo --gc --minify
```

2. Generate index with `pagefind_extended` binary:

```bash
./pagefind_extended --site public
```

This creates `public/pagefind/` (including `pagefind.js` and index files).

> `extended` is recommended for better CJK tokenization.

### Local dev workflow (`hugo server`)

For local dogfooding with `hugo server`, make sure output is written to disk before indexing:

```bash
hugo server --renderToDisk
./pagefind_extended --site public
```

### Alternative: npx workflow

```bash
hugo --gc --minify
npx pagefind@1.4.0 --site public
```

### Standard vs extended

- `pagefind_extended`: better CJK tokenization, recommended default
- `pagefind`: smaller standard build for non-CJK-heavy content
- `npx pagefind@1.4.0` downloads `pagefind_extended` by default

## Design notes

- The component does not modify FixIt core files directly; it extends through custom partial injection.
- It hooks `autocomplete` early on `DOMContentLoaded` and swaps search data source to Pagefind.
- It keeps the original FixIt search UI structure and styling.
- It injects filter/date metadata via `meta` tags to improve result controllability.

## References

- [Pagefind Docs](https://pagefind.app/docs/)
- [Pagefind GitHub Repository](https://github.com/CloudCannon/pagefind)
- [FixIt Custom Blocks](https://fixit.lruihao.cn/zh-cn/references/blocks/)
- [FixIt Component Development](https://fixit.lruihao.cn/contributing/components/)
