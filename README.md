<!-- markdownlint-disable-file MD033 MD041 -->
<h1 align="center">cmpt-pagefind-component | FixIt</h1>

<div align="center">
  <p>基于 Pagefind 的 FixIt 搜索替换组件，保持默认搜索 UI，切换搜索引擎为 Pagefind。</p>
  简体中文 · <a href="./README.en.md">English</a>
</div>

## 简介

`cmpt-pagefind-component` 是一个面向 FixIt 的主题组件：

- 使用 `params.search.type = "pagefind"` 接管 FixIt 默认搜索能力
- 保持 FixIt 原有搜索入口和下拉 UI 外观
- 将搜索底层从 `fuse` / 其他引擎切换为 Pagefind（浏览器 API）
- 通过 `params.search.pagefind.*` 提供 Pagefind 配置和高级能力开关

## 特性

- 默认模式 `basic`：尽量贴近 FixIt 原搜索行为
- 高级模式 `advanced`：支持 Pagefind 的防抖搜索与高级搜索选项
- 沿用 FixIt 配置入口：`params.search.*`
- 通过 `data-pagefind-filter` 自动补充 `hidden` / `encrypted` 过滤字段
- 支持多语言站点（按当前页面 `html[lang]` 自动使用对应索引）

## 要求

- FixIt `v0.4.0` 或更高版本
- Hugo Extended `v0.156.0` 或更高版本

> Pagefind 索引生成在站点构建后单独执行，不和 Hugo 构建流程强耦合。

## 安装组件

### 作为 Hugo 模块安装

先确保你的站点已经初始化为 Hugo 模块，然后在站点 `hugo.toml` 中添加：

```toml
[module]

[[module.imports]]
path = "github.com/hugo-fixit/FixIt"

[[module.imports]]
path = "github.com/hugo-fixit/cmpt-pagefind-component"
```

更新模块：

```bash
hugo mod get -u
hugo mod tidy
```

### 作为 Git 子模块安装

```bash
git submodule add https://github.com/hugo-fixit/FixIt.git themes/FixIt
git submodule add https://github.com/0x5c0f/cmpt-pagefind-component.git themes/cmpt-pagefind-component
```

然后在站点 `hugo.toml` 启用：

```toml
theme = ["FixIt", "cmpt-pagefind-component"]
```

## 配置

### 1) 开启搜索并切换为 Pagefind

```toml
[params.search]
enable = true
type = "pagefind"
placeholder = "搜索文章标题或内容"
maxResultLength = 10
```

### 2) 注入组件 Partial（必须）

组件不会自动改写站点的 `params.customPartials.head`。  
请在站点配置中显式加入：

```toml
[params.customPartials]
head = ["inject/cmpt-pagefind-component.fixit.html"]
```

如果你已经有多个 `head` 注入项，请追加，不要覆盖：

```toml
[params.customPartials]
head = [
  "inject/xxx.fixit.html",
  "inject/cmpt-pagefind-component.fixit.html",
  "inject/yyy.fixit.html"
]
```

### 3) 配置 Pagefind 行为（可选）

```toml
[params.search.pagefind]
mode = "basic"              # basic | advanced
bundlePath = "/pagefind/"   # pagefind 索引目录
maxResultLength = 6         # 可选：覆盖 params.search.maxResultLength
debounceTimeoutMs = 300     # advanced 模式下的防抖时间
useBuiltInFilters = true    # 自动附加 hidden=false / encrypted=false 过滤
excerptLength = 80          # 搜索摘要长度（示例：自定义为 80）
```

可选高级配置：

```toml
[params.search.pagefind.filters]
# category = "技术"

[params.search.pagefind.sort]
# date = "desc"

[params.search.pagefind.ranking]
# termSaturation = 1.2
```

### 配置参数总览（建议保留此结构）

#### `params.search.*`（沿用 FixIt）

| 键 | 类型 | 默认值 | 说明 |
|---|---|---:|---|
| `params.search.enable` | `bool` | `false` | 是否启用搜索入口 |
| `params.search.type` | `string` | - | 设为 `"pagefind"` 时由本组件接管 |
| `params.search.placeholder` | `string` | 主题默认文案 | 输入框占位文本 |
| `params.search.maxResultLength` | `int` | `10` | 下拉结果最大条数；设为 `0` 时不展示结果列表 |
| `params.search.snippetLength` | `int` | `30` | `excerptLength` 的默认来源。未设置 `params.search.pagefind.excerptLength` 时会复用该值 |

#### `params.search.pagefind.*`（本组件配置）

| 键 | 类型 | 默认值 | 说明 |
|---|---|---:|---|
| `mode` | `string` | `"basic"` | 搜索模式：`basic` 或 `advanced` |
| `bundlePath` | `string` | `"/pagefind/"` | Pagefind 索引目录，组件会动态加载 `${bundlePath}pagefind.js` |
| `maxResultLength` | `int` | 复用 `params.search.maxResultLength`（默认 `10`） | Pagefind 结果条数上限。设置后可覆盖全局 `params.search.maxResultLength` |
| `debounceTimeoutMs` | `int` | `300` | `advanced` 模式下 `debouncedSearch` 防抖间隔；设为 `0` 等价于不防抖 |
| `useBuiltInFilters` | `bool` | `true` | 自动尝试附加 `hidden=false`、`encrypted=false`。仅在索引中存在这两个 filter 字段时生效 |
| `excerptLength` | `int` | 未设置时复用 `params.search.snippetLength`（默认 `30`） | 下拉摘要长度。若显式设置则优先使用该值；设为 `0` 时通常会导致“仅标题无摘要” |
| `baseUrl` | `string` | `""` | 高级参数，默认不需要设置。仅在“站点部署于子路径”这类场景使用；误设可能导致结果 URL 异常 |
| `highlightParam` | `string` | `""` | 透传到 `pagefind.options({ highlightParam })` |
| `filters` | `map[string]any` | `{}` | 查询过滤条件，透传到 `pagefind.search(..., { filters })` |
| `sort` | `map[string]any` | `{}` | 查询排序参数，透传 `sort` |
| `ranking` | `map[string]any` | `{}` | Pagefind 排序权重参数，透传 `ranking` |
| `indexWeight` | `map[string]any` | `{}` | Pagefind 索引权重参数，透传 `indexWeight` |
| `mergeFilter` | `map[string]any` | `{}` | Pagefind 合并过滤参数，透传 `mergeFilter` |

组件的 `hugo.toml` 已包含主要默认键，并对高级项提供注释示例；用户仅在需要定制时于站点配置中覆盖对应字段即可。

> 说明：`ranking` 支持常见键名的大小写兼容（例如 `termsaturation` 会自动归一化为 `termSaturation`）。  
> `indexWeight` 同时支持对象与数字写法。
> `baseUrl` 与 `bundlePath` 不应设置为同一个值（如都为 `/pagefind/`），否则可能出现 `/pagefind/posts/...` 这类错误结果链接。

### 参数取值优先级（重点）

为避免“看文档和看结果不一致”，下面是组件当前实现的实际取值链路：

| 参数 | 取值优先级（从高到低） | 当前开箱默认 |
|---|---|---|
| `excerptLength` | `params.search.pagefind.excerptLength` → `params.search.snippetLength` → `30` | 默认复用 `snippetLength`（FixIt 默认 `30`） |
| `maxResultLength` | `params.search.pagefind.maxResultLength` → `params.search.maxResultLength` → `10` | 默认复用 `params.search.maxResultLength`（FixIt 默认 `10`） |
| `debounceTimeoutMs` | `params.search.pagefind.debounceTimeoutMs` → `300` | `300` |
| `bundlePath` | `params.search.pagefind.bundlePath` → `"/pagefind/"` | `"/pagefind/"` |
| `useBuiltInFilters` | `params.search.pagefind.useBuiltInFilters` → `true` | `true` |

> 对 `excerptLength` 的具体说明：  
> 组件默认不再单独设置 `excerptLength`，会优先复用 `params.search.snippetLength`。  
> 若站点也未设置 `snippetLength`，则最终回退到 `30`（与 FixIt 默认行为保持一致）。

### 透传参数示例（可按需启用）

```toml
[params.search.pagefind]
# highlightParam 会把命中的词同步到 URL 参数，便于结果页高亮
highlightParam = "highlight"

[params.search.pagefind.filters]
# 仅检索指定分类/标签内容
category = "技术"
tags = ["hugo", "fixit"]
# 若你的索引中有该字段，可强制排除加密内容
encrypted = "false"

[params.search.pagefind.sort]
# 按日期倒序（字段需存在于索引中）
date = "desc"

[params.search.pagefind.ranking]
# 结果排序策略微调
termSaturation = 1.2
termSimilarity = 1.0
pageLength = 0.8

[params.search.pagefind.indexWeight]
# 字段加权：标题命中权重更高
title = 8
tags = 3
# 或者统一权重（数字写法也支持）
# value = 1

[params.search.pagefind.mergeFilter]
# 多个过滤条件如何合并：and / or（按 Pagefind 语义）
operator = "and"
```

> 建议：先用最小配置跑通，再逐项开启这些高级参数；每次只改一个参数，便于定位效果变化。

### 推荐最小配置

```toml
[params.search]
enable = true
type = "pagefind"

[params.customPartials]
head = ["inject/cmpt-pagefind-component.fixit.html"]
```

> 其余参数都有组件默认值；仅在你需要定制行为时再添加 `params.search.pagefind.*`。

### 常见配置案例

#### 案例 A：保持默认行为（推荐起步）

```toml
[params.search]
enable = true
type = "pagefind"

[params.customPartials]
head = ["inject/cmpt-pagefind-component.fixit.html"]
```

#### 案例 B：高级模式 + 自定义结果条数与摘要

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

#### 案例 C：部署在子路径（仅必要时）

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

> 注意：`baseUrl` 是高级参数，只有子路径部署时才建议配置。
> `baseUrl` 不要与 `bundlePath` 配置成同一个值。

### 关于 `hugo.toml`（组件仓库内）

本组件仓库根目录的 `hugo.toml` 是**有必要保留**的，作用有两点：

1. 声明组件最低运行要求（`Hugo Extended >= 0.156.0`），防止低版本环境出现不可预期行为。
2. 给组件提供默认参数（如 `mode`、`bundlePath`），减少用户手动配置量；`excerptLength` 默认复用 FixIt 的 `snippetLength`。

它不会强制覆盖站点自己的同名配置；站点配置优先级更高。  
实践上建议：`hugo.toml` 保持“可运行默认值”，详细使用案例与参数解释放在 `README`，便于维护。

## 构建与索引生成

Pagefind 需要在 Hugo 构建后额外执行一次索引。

### 默认推荐：二进制方式（贴近 Hugo 站点流程）

1. 先构建 Hugo 站点（生成 `public/`）：

```bash
hugo --gc --minify
```

2. 使用 `pagefind_extended` 二进制生成索引：

```bash
./pagefind_extended --site public
```

执行后会生成 `public/pagefind/` 目录（包含 `pagefind.js` 和索引数据）。

> 说明：`extended` 对中文分词更友好，中文站点建议优先使用。

### 本地开发联调（Hugo Server）

若你使用 `hugo server` 进行本地联调，建议先确保输出目录可落盘，然后再单独执行 Pagefind：

```bash
hugo server --renderToDisk
./pagefind_extended --site public
```

这样可以更贴近“先 Hugo、后索引”的生产流程。

### 备选：npx 方式

```bash
hugo --gc --minify
npx pagefind@1.4.0 --site public
```

### 关于标准版 / extended 版

- `pagefind_extended`：中文/日文分词更友好，推荐作为默认选择
- `pagefind`：标准版，体积更小，适合不需要增强分词的场景
- 使用 `npx pagefind@1.4.0` 时，默认下载 `pagefind_extended`
- 若你使用二进制手动部署，也可以自行选择 `pagefind` 或 `pagefind_extended`

## 设计说明

- 本组件不直接改 FixIt 核心文件，而是通过自定义注入点扩展
- 在 `DOMContentLoaded` 早期挂钩 `autocomplete`，将搜索数据源替换为 Pagefind
- 保留 FixIt 现有搜索 UI 结构和样式（桌面/移动端入口、下拉容器）
- 通过 `meta` 注入 Pagefind 过滤与日期元数据，提升结果可控性

## 参考

- [Pagefind 文档](https://pagefind.app/docs/)
- [Pagefind 官方仓库](https://github.com/CloudCannon/pagefind)
- [FixIt 自定义块](https://fixit.lruihao.cn/zh-cn/references/blocks/)
- [开发主题组件 | FixIt](https://fixit.lruihao.cn/contributing/components/)
