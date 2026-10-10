---
title: 文本格式（HTML）
aliases: []
tags: [HTML]
description: HTML 里怎么加粗、倾斜、上下标、删除线——标签的语义与 Markdown 写法的区别
created: 2026-10-10
draft: false
---

## 粗体

用 `<b>`（bold，纯样式）或 `<strong>`（语义上"重要"）。

```
<b>这是粗体</b>
<strong>这也是粗体，但表示内容重要</strong>
```

展示：<b>这是粗体</b> · <strong>这也是粗体，但表示内容重要</strong>

## 斜体

`<i>`（italic，纯样式）或 `<em>`（emphasis，语义强调）。

```
<i>这是斜体</i>
<em>这是强调</em>
```

展示：<i>这是斜体</i> · <em>这是强调</em>

> [!note] 成对标签只是"容器"
> `<b>` 这类标签本身不带含义，只描述它**包住的那段文字**该怎么显示。
> 所以必须写**结束标签** `</b>`，否则从那一点往后全是粗体。
> 少数标签没有内容可包（如 `<br>`），写成单标签即可。

## 上下标

```
H<sub>2</sub>O
E = mc<sup>2</sup>
```

展示：H<sub>2</sub>O · E = mc<sup>2</sup>

## 删除线与插入

```
<del>被删掉的内容</del>
<ins>后来插入的内容</ins>
```

展示：<del>被删掉的内容</del> · <ins>后来插入的内容</ins>

不要用 `<u>` 做下划线——网页上下划线默认是超链接，容易引起误会。

## 小字与高亮

```
<small>附注、版权声明</small>
<mark>需要读者注意的地方</mark>
```

展示：<small>附注、版权声明</small> · <mark>需要读者注意的地方</mark>

## 和 Markdown 写法的区别

| 效果 | HTML | Markdown |
|---|---|---|
| 粗体 | `<strong>x</strong>` | `**x**` |
| 斜体 | `<em>x</em>` | `*x*` |
| 删除线 | `<del>x</del>` | `~~x~~` |
| 上标 | `<sup>x</sup>` | 无，得写 HTML |

Markdown 写不了的上标、下标、高亮，直接在 Markdown 里内联写 HTML 标签即可——Obsidian 和 Quartz 都支持这种混写。

## 相关

- [[段落和换行]] — `<p>` 与 `<br>`
- [[转义字符（HTML）]] — 想在页面里显示 `<b>` 这几个字符本身怎么写
- [[文本格式（Markdown）]] — 同一件事的 Markdown 写法
