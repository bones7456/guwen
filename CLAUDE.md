# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) and Codex (Codex.ai/code) when working with code in this repository.

## 项目概况

「古文解析」——经典古文的逐字解析静态网站，纯 HTML/CSS/JS，无构建工具、无依赖、无测试框架。部署在 GitHub Pages（https://bones7456.github.io/guwen/）。

本地预览：直接用浏览器打开 HTML 文件，或在仓库根目录运行 `python3 -m http.server`。

## 架构

每篇文章一个独立目录（`TWGX/` 滕王阁序、`QZW/` 千字文），每个目录包含三个文件：

- **`data.js`** — 结构化内容数据，挂在全局变量上（TWGX 为 `window.TENGWANG_DATA`，QZW 为 `window.QIANZIWEN_DATA`）。这是内容的唯一来源；改注释、典故、分段都只改这个文件。
- **`index.html`** — 页面骨架 + 全部渲染与交互逻辑（内联 `<script>`）。
- **`styles.css`** — 样式，含每个段落的「意境」配色主题。

### 数据模型（data.js）

顶层是段落数组，每段：`{ id, label, title, subtitle, mood, items, coda? }`。`mood` 对应 styles.css 里的 `.section.mood-*` 主题类（如 `parchment`、`celadon`、`deep-night` 等），新增段落配色需在 CSS 中补一个主题块。

每个 item（一句原文）：`{ n, raw, defs, meaning?, note?, allusion?, highlight? }`

- `n: "题"` 的 item 走特殊的题名/序言渲染（`renderPreface`）。
- `defs` 是 `{ w: 词, d: 释义 }` 数组。渲染时 `buildRows()` 用 defs 里的词（按长度从长到短）去匹配 `raw` 原文切分成行——所以 **defs 里的 `w` 必须与 `raw` 中的原文字面完全一致**，否则该词不会被匹配、显示为无释义的灰行。
- `allusion` 可为单个对象或数组：`{ title, era, people, story, impact }` 或简式 `{ title, text }`。
- `meaning`（句意）、`allusion`（典故）、`note`（说明）会合并渲染在该句最后一个有释义的词条下方。

### 渲染逻辑中的两个硬编码表（index.html）

- `TRIVIAL` 集合：TWGX 中虚词（之、于、而、其……）即使出现在 defs 中也不渲染释义，单独成灰行——这是 README 中「虚词单独标注，不强行附义」约定的实现。QZW 逐字全释，该集合为空。
- `PINYIN` 映射：生僻字注音表。新增文章内容若含生僻字，需在此补注音。

### 交互

均在 index.html 内联脚本中：点击词行钉住高亮（Esc 或点空白释放）、IntersectionObserver 驱动的目录侧栏高亮与当前阅读句高亮、字号四档调节（`--char-scale` CSS 变量 + localStorage 持久化）。

## 新增篇目时

参照 `TWGX/` 复制一套三文件结构，改数据与全局变量名，并在根 `README.md` 中补条目链接。
