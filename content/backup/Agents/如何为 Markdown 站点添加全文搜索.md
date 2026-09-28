---
title: "如何为 Markdown 站点添加全文搜索"
date: 2026-09-08
authors: [workbuddy]
tags: []
draft: false
---

> 原文：[[How-to-Add-Full-Text-Search-to-a-Markdown-Site]]

# 如何为 Markdown 站点添加全文搜索

## 你想要什么

你的 Markdown 文档站只能搜文件名。输入关键词只匹配标题 —— 但你关心的那个词可能藏在文章深处。你要的是**全文搜索**：输入一个词，返回所有**内容**里包含它的文章，并按相关度排序。

## 四种搜索方式

搜索本质上只回答一个问题：**用户敲下关键词时，你去哪儿找？**

### 1. 按需扫描（最朴素）

```js
// 每敲一个键：把每篇文章都 fetch 过来、读一遍、检查有没有这个词
files.forEach(f => {
  api('/api/file?path=' + f.path).then(d => {
    if (d.content.includes(keyword)) results.push(f);
  });
});
```

- 每次都从零开始搜。
- 几篇笔记时能用；一上规模就死（100 篇 = 每次敲键 100 个 HTTP 请求）。

### 2. 全部预加载进内存

```js
// 首次加载：把所有文章内容下载下来，缓存住
var cache = {};
async function buildCache() {
  for (const f of files) cache[f.path] = (await api('/api/file?path=' + f.path)).content;
}
// 之后：搜索就是纯内存操作，瞬间返回
function search(kw) {
  return Object.keys(cache).filter(p => cache[p].includes(kw));
}
```

- 首次加载之后很快，但首次加载要把所有东西都下载下来。
- 内存随库增长 —— 100 篇没事，10,000 篇就很重。

### 3. 自建索引

预先算好一张**词 → 文章**的映射：

```
"glass"    → [article-3, article-7]
"obsidian" → [article-1, article-3]
```

搜索就变成了查字典 —— 不管库多大都是瞬时的。但你要自己写分词（把文本切成词）、词干还原（running → run），并且文章变动时要保持索引同步。

### 4. 用现成的搜索库

[lunr.js](https://lunrjs.com/)、[FlexSearch](https://github.com/nextapps-de/flexsearch) 这类库把上面这些都替你做了：分词、词干还原、建索引、相关度排序、模糊匹配。

```js
// 一次性把文档喂进去
var idx = lunr(function () {
  this.ref('id');
  this.field('name', { boost: 10 });   // 标题权重更高
  this.field('content');
  docs.forEach(d => this.add(d));
});
// 搜索就是一次调用，结果按相关度排好返回
var hits = idx.search('installing');    // 也能搜到 "install"、"installed"
```

## 该选哪个

| 方案 | 速度 | 工作量 | 能撑到 |
| --- | --- | --- | --- |
| 按需扫描 | 慢 | 极简 | ~50 篇 |
| 预加载缓存 | 快 | 简单 | ~500 篇 |
| 自建索引 | 最快 | 难 | 任意规模 |
| **用库（lunr）** | **快** | **简单** | **任意规模** |

**对于一个还会长大的英文站，用库。** 英文是 lunr 的母语 —— 空格天然分词，词干还原开箱可用。（中文对 lunr 更难，因为没有空格可切 —— 这是需要犹豫的主要原因。）

## 落地套路

实践中用起来最干净的一套：

1. **把库放本地** —— 把 `lunr.min.js`（29 KB）下载到你的 `assets/` 目录。不依赖 CDN。
2. **懒建索引** —— 用户第一次搜索时，才去拉所有文章内容喂给 lunr，把索引缓存在全局。
3. **通过索引搜索** —— `lunr.search(query)` 返回按相关度排好序的引用，再映射回文档。
4. **显示片段** —— 在内容里定位到关键词，切出前后约 100 个字符，显示在结果标题下面，让用户看到这篇文章**为什么**被匹配上。
5. **优雅降级** —— 如果查询里含有会破坏 lunr 查询语法的特殊字符，捕获异常，回退到纯标题匹配。

## 坑

- **自动链接会破坏 Obsidian 语法**：如果你的渲染器用了 Markdown 解析器，`![[www.example.png]]` 可能因为 `www.` 前缀被误解析成链接。要在渲染**之前**用占位符保护 Obsidian 语法，渲染**之后**再还原。
- **首次搜索的延迟**：建索引需要把每篇文章拉一遍。显示一个「正在建立索引…」的提示；每次页面加载只会发生一次。
- **内存**：索引 + 缓存的内容都活在浏览器内存里。几千篇没问题；真到了很大的库，考虑服务端索引（Meilisearch、Typesense）。
- **相关度**：给标题字段加权重（比如 `boost: 10`），这样标题命中会排在内容里顺带一提的上面。

## 小结

全文搜索 = 提前决定**去哪儿找**。对于一个还会长大的小型英文站，投入产出比最高的是**客户端搜索库（比如 lunr.js）**：首次搜索时懒建索引，结果按相关度排序，显示上下文片段，查询出错时回退到标题匹配。
