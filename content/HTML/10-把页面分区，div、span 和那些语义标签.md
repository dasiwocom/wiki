---
title:
aliases:
tags:
  - html
description: 块级和行内的区别，div 和 span 怎么用，以及 header、nav、main、section、article、aside、footer 这些语义标签各自管什么。
---

# 把页面分区，div、span 和那些语义标签

前面讲的标签都是在描述"这段文字是什么"。剩下的一件事是：怎么把页面切成几块。

## 块级和行内

浏览器对元素有两大分类，这个区别必须先搞清：

| | 块级元素 | 行内元素 |
| --- | --- | --- |
| 例子 | `div` `p` `h1` `ul` `table` | `span` `a` `strong` `img` |
| 排列 | 自己独占一行 | 和文字排在同一行 |
| 宽高 | 可以设 | 设了也没用（改 `display` 除外） |
| 能装什么 | 里面可以放块级和行内 | 原则上只放行内 |

看效果最直观：

<div style="border:1px dashed var(--gray);padding:4px 8px;margin-bottom:6px">我是一个 div，独占一行</div>
<div style="border:1px dashed var(--gray);padding:4px 8px">我也是 div，另起一行</div>
<p>这一行里有个 <span style="border:1px dashed var(--gray);padding:0 4px">span</span>，它没有换行，就挤在字中间。</p>

## div 和 span：无语义的容器

`<div>` 是块级容器，`<span>` 是行内容器。

它们本身不带任何含义，纯粹用来"圈出一块地方"，好在 CSS 里给它加样式、在 JS 里操作它。

```html
<div class="card">
  <h3>标题</h3>
  <p>正文……</p>
</div>

<p>价格 <span class="price">¥199</span></p>
```

没有别的标签能用，才用它们。有合适的语义标签就别用 div。

## 语义标签：让结构能读出来

下面这些标签在**浏览器里显示的样子和 `<div>` 完全一样**，区别在含义：

```html
<body>
  <header>页头：logo、站点名</header>
  <nav>导航：一排链接</nav>
  <main>
    <article>
      <h1>文章标题</h1>
      <section>
        <h2>第一节</h2>
        <p>正文……</p>
      </section>
    </article>
    <aside>侧栏：相关链接、广告</aside>
  </main>
  <footer>页脚：版权、备案号</footer>
</body>
```

| 标签 | 放什么 |
| --- | --- |
| `<header>` | 页头或某个区块的开头，通常有标题、logo |
| `<nav>` | 一组导航链接 |
| `<main>` | 页面主体内容，一个页面只能有一个 |
| `<article>` | 能独立拿出去存在的内容：一篇帖子、一条新闻、一条评论 |
| `<section>` | 有标题的一个章节 |
| `<aside>` | 和正文关系不大的旁支内容：侧栏、相关推荐 |
| `<footer>` | 页脚，版权、联系方式 |
| `<figure>` / `<figcaption>` | 图 + 图注 |
| `<details>` / `<summary>` | 可折叠的一块 |
| `<mark>` `<time>` | 高亮、时间 |

## 为什么不干脆全用 div

用 div 也能做出一样的页面，但：

- 屏幕阅读器可以跳到 `<nav>`、直达 `<main>`，用户不用听完整页的头部
- 搜索引擎知道哪块是正文、哪块是广告，正文权重更准
- 你自己三个月后回来看，一眼就知道每块是干什么的

代价只是换个标签，所以没有理由不写。

## 结构长什么样

一个页面的典型骨架：

```text
<body>
├── header      页头
├── nav         导航
├── main        主体（一个页面只能一个）
│   ├── article
│   │   ├── section
│   │   └── section
│   └── aside   侧栏
└── footer      页脚
```

## section 和 div 怎么选

判断标准是"这块内容有没有标题"：

- 有 `<h2>`/`<h3>` 当标题，是一个独立章节 → 用 `<section>`
- 只是为样式圈一块地方，没有标题 → 用 `<div>`

## article 和 section 怎么选

问自己一句：**这块内容单独拿出来，还能讲通吗？**

能——一篇完整的帖子、一条独立的产品介绍 → `<article>`。

不能，离开上下文就看不懂 → `<section>`。

一篇文章里可以嵌多条 `<article>`（比如论坛帖子下面的每条回复都是 article）。

## 布局靠 CSS

这七个标签说完，你会发现它们只解决了"这块内容是什么"，没解决"这块该放左边还是右边"。

真正的排版是 CSS 的 flex 和 grid 干的活，标签只负责表明身份。

下一步可以参考：[CSS 网页布局 - 菜鸟教程](https://www.runoob.com/css/css-website-layout.html)（用 float 的经典两栏、三栏布局），[HTML5 语义元素 - 菜鸟教程](https://www.runoob.com/html/html5-semantic-elements.html)。

---

> 系列第 10 / 13 篇 · ← 上一步：[[09-表单，输入框、单选、下拉和按钮|表单]] · 下一步：[[11-head 里放什么，CSS 和 JS 从哪进来|head 里放什么]] → · 路线图：[[HTML 总览]]
