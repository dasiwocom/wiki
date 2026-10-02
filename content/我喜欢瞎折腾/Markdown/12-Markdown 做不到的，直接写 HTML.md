---
title:
aliases:
tags:
description: Markdown 表达不了的时候可以直接写 HTML：折叠块、上下标、键盘按键、嵌入视频、内联 SVG 示意图，以及这个站点对这些的支持情况。
---

# Markdown 做不到的，直接写 HTML

> 系列第 12 / 13 篇 · ← 上一步：[[11-Mermaid 图表和数学公式|Mermaid 图表]] · 下一步：[[13-网上抄来的 Markdown 写法不生效，先查这几种|网上抄来的写法不生效]] → · 路线图：[[Markdown 语法总览]]

Markdown 本质上就是一套"简写"，它最终会变成 HTML。所以遇到它表达不了的东西，**直接写 HTML 标签就行**——这个站点实测会原样保留，不会被过滤掉。

这不是什么 hack，是 Markdown 规范本身就允许的。

## 最实用的几个标签

| 想要什么 | 写法 | 效果 |
| --- | --- | --- |
| 强制换行 | `<br>` | 见下面 |
| 上标 | `X<sup>2</sup>` | X<sup>2</sup> |
| 下标 | `H<sub>2</sub>O` | H<sub>2</sub>O |
| 下划线 | `<u>下划线</u>` | <u>下划线</u> |
| 键盘按键 | `<kbd>Ctrl</kbd>` | <kbd>Ctrl</kbd> |
| 标记文字 | `<mark>标记</mark>` | <mark>标记</mark> |
| 折叠块 | `<details><summary>` | 见下面 |
| 音频 | `<audio controls src="...">` | 播放器 |
| 嵌入网页 | `<iframe src="...">` | 见下面 |
| 自定义图形 | `<svg>...</svg>` | 见下面 |

## 折叠块

`<details>` 是最好用的一个——它能把内容藏起来，想看再点开：

```html
<details>
<summary>点开看答案</summary>

这里是藏起来的内容，可以写多行，
也能用 **Markdown**。
</details>
```

↓ 渲染出来是这样（点一下试试）：

<details>
<summary>点开看答案</summary>

这里是藏起来的内容，可以写多行，
也能用 **Markdown**。
</details>

`<summary>` 里是标题（永远显示），后面是内容（默认隐藏）。

和 Callout 的折叠有什么区别？

| | Callout 折叠 | `<details>` |
| --- | --- | --- |
| 写法 | `> [!note]- 标题` | `<details><summary>标题</summary>` |
| 样式 | 带图标和颜色的提示块 | 朴素的折叠条 |
| 适合 | 提示、补充说明 | 塞大段内容、代码、答案 |

## 嵌入视频

视频没法存在笔记里（文件太大），但可以嵌站外播放器。

B 站的视频链接是 `https://www.bilibili.com/video/BV1xx411c7mD`，要换成播放器地址：

```html
<iframe
  src="//player.bilibili.com/player.html?bvid=BV1xx411c7mD"
  style="width:100%;aspect-ratio:16/9;border:0;border-radius:8px"
  allowfullscreen>
</iframe>
```

两个必须注意的点：

- **`style` 里的宽度和 `aspect-ratio` 一定要写**，不然 iframe 会变成默认的小尺寸，或者变形
- **别用 `![[xxx.mp4]]`**，这个站点不支持嵌入本地视频文件

讲操作步骤、实物演示这类"文字说不清、看一眼就懂"的内容，嵌个视频比写五百字有用。

## 内联 SVG：自己画示意图

这是 HTML 最强大的用法——**直接在笔记里画出你想表达的图**。

```html
<svg viewBox="0 0 400 120" xmlns="http://www.w3.org/2000/svg" style="max-width:100%;height:auto">
  <rect x="10" y="20" width="90" height="60" rx="8" fill="var(--lightgray)" stroke="var(--gray)" />
  <text x="55" y="55" font-size="14" text-anchor="middle">K1</text>
  <line x1="100" y1="50" x2="160" y2="50" stroke="var(--secondary)" stroke-width="2" />
</svg>
```

↓ 渲染出来是这样：

<svg viewBox="0 0 400 120" xmlns="http://www.w3.org/2000/svg" style="max-width:100%;height:auto">
  <rect x="10" y="20" width="90" height="60" rx="8" fill="var(--lightgray)" stroke="var(--gray)" />
  <text x="55" y="55" font-size="14" text-anchor="middle">K1</text>
  <line x1="100" y1="50" x2="160" y2="50" stroke="var(--secondary)" stroke-width="2" />
</svg>

几个必须遵守的规则：

- **必须带 `viewBox`**，并加 `style="max-width:100%;height:auto"`，否则窄屏会溢出
- 中文用 `<text>` 时加 `text-anchor="middle"` 才能居中
- **不要写 `<script>`**，也不要加 XML 声明

**颜色要用 CSS 变量**，这样图在亮色和暗色主题下都能看清：

| 用途 | 写法 | 亮色 / 暗色实际值 |
| --- | --- | --- |
| 页面背景 | `var(--light)` | `#faf8f8` / `#161618` |
| 卡片底色 | `var(--lightgray)` | `#e5e5e5` / `#393639` |
| 连线、边框 | `var(--gray)` | `#b8b8b8` / `#646464` |
| 强调色（蓝） | `var(--secondary)` | `#284b63` / `#7b97aa` |
| 强调色（青绿） | `var(--tertiary)` | `#84a59d` / `#84a59d` |
| 标题文字 | `var(--dark)` | `#2b2b2b` / `#ebebec` |

直接写死颜色（比如 `fill="#000"`）的话，总有一个主题下看不清。

> [!tip] SVG 里的文字不用写 fill
> 站点样式表里有 `text { fill: var(--darkgray) }`，所以 `<text>` 会自动继承正文字色。手动画 `fill="#000"` 反而会在暗色主题下变成黑底黑字。
>
> 唯一例外：深底上的文字要显式写 `fill="var(--light)"`。

## 什么时候该用 HTML，什么时候不该

**该用的时候：**

- Markdown 真的没有对应语法（上下标、下划线、折叠、视频）
- 需要精确控制排版（居中、分栏、自定义图形）
- 表格要合并单元格

**不该用的时候：**

- 明明有 Markdown 语法却偏要写 HTML。能用 `**粗体**` 就别写 `<b>粗体</b>`
- 只是为了改个颜色、调个字号——这类样式应该交给主题统一处理

**判断标准：这段内容是不是"内容"，还是"样式"？**内容是 HTML 该干的，样式不是。

## 常见坑

**在 Obsidian 编辑视图里看到的是源码**

内联 HTML 和 SVG 在 Obsidian 的**编辑视图**里显示为一行行代码，只有切到**阅读视图**才是真正的效果。别以为写错了。

**RSS 里看不到**

和 Mermaid 一样，SVG 和 iframe 在 RSS 阅读器、分享预览卡片里**都不会显示**。重要的东西别只放在里面。

**嵌入 HTML 文件是关掉的**

`![[xxx.html]]` 这种"嵌入一个 HTML 文件"的方式这个站点是**关闭的**（配置里 `enableInHtmlEmbed: false`）。要画图就把 `<svg>` 直接写在正文里，别想着放到单独文件再嵌入。

**标签没闭合**

HTML 比 Markdown 严格：写了 `<details>` 就要写 `</details>`。漏一个闭合标签，后面整页的排版都会乱。

**用了 `<script>`**

这个站点虽然不过滤 HTML，但**别写脚本**。一是没必要，二是给网站引入安全风险。
