---
title:
aliases:
tags:
  - css
description: block、inline、inline-block、none 四个值的区别，哪些标签默认是什么，以及 display:none、visibility:hidden、opacity:0 三者的差别。
---

# display，块级、行内和隐藏

`display` 决定一个元素怎么参与排版。它是最值得先搞懂的一个属性。

## 四个常用值

```css
.a { display: block; }         /* 块级：独占一行 */
.b { display: inline; }        /* 行内：和文字排在同一行 */
.c { display: inline-block; }  /* 行内块：不换行，但能设宽高 */
.d { display: none; }          /* 直接不显示，不占位置 */
```

看效果：

<style>
.wbd-disp span, .wbd-disp div {
  border: 1px solid #b8b8b8;
  padding: 4px 8px;
  background: rgba(132,165,157,.15);
  margin: 2px;
}
.wbd-disp .blk { display: block; }
.wbd-disp .inl { display: inline; width: 200px; height: 60px; }  /* 宽高写了也不生效 */
.wbd-disp .ib  { display: inline-block; width: 120px; height: 48px; }
</style>

<div class="wbd-disp">
  <div class="blk">display:block，独占一行</div>
  <div class="blk">display:block，也独占一行</div>
  <span class="inl">display:inline，写了宽高也没用</span>
  <span class="ib">display:inline-block</span>
  <span class="ib">display:inline-block</span>
</div>

三个区别一眼就能看出来：

| 值 | 换行 | 能设宽高 | 上下 margin/padding |
| --- | --- | --- | --- |
| `block` | 独占一行 | 能 | 都生效 |
| `inline` | 不换行 | **不能** | 上下不撑开行高 |
| `inline-block` | 不换行 | 能 | 都生效 |

`inline` 元素设 `width` 和 `height` 无效，是新手最常困惑的一点。想让一个 `<span>` 有尺寸，改成 `inline-block`。

## 常见标签的默认 display

| display | 标签 |
| --- | --- |
| `block` | `div` `p` `h1`~`h6` `ul` `ol` `li` `table` `form` `header` `nav` `main` `section` `footer` |
| `inline` | `span` `a` `strong` `em` `img` `input` `label` `code` |
| `inline-block` | `button` `select` `textarea` |
| `none` | `head` `title` `meta` `link` `style` `script` |

注意 `img` 和 `input` 是行内元素，但它们是"替换元素"，所以能设宽高。这一点和普通行内元素不一样。

想知道某个元素默认是什么，F12 里选中它看一眼 Computed 面板就有了。

## 改动默认行为

```css
li { display: inline-block; }     /* 把列表项横过来排，做导航 */
a { display: block; }             /* 让整个链接区域都能点 */
span.tag { display: inline-block; }
```

第二条很常用：`<a>` 默认只有文字本身能点，改成 `block` 之后整块区域包括内边距都能点，手机上体验好很多。

## 三种"隐藏"的区别

这三个都能让元素看不见，但行为完全不同：

```css
.a { display: none; }        /* 不显示，不占位置，读屏软件也读不到 */
.b { visibility: hidden; }   /* 不显示，但位置还留着 */
.c { opacity: 0; }           /* 完全透明，位置留着，还能被点到 */
```

看对比（中间那个是 `visibility: hidden`，它占的位置还在）：

<style>
.wbd-hide div { padding: 8px; border: 1px dashed #b8b8b8; margin: 4px 0; }
.wbd-hide .h1 { display: none; }
.wbd-hide .h2 { visibility: hidden; }
.wbd-hide .h3 { opacity: 0; }
</style>

<div class="wbd-hide">
  <div>第一个：正常显示</div>
  <div class="h1">第二个：display:none，整块消失了</div>
  <div class="h2">第三个：visibility:hidden</div>
  <div class="h3">第四个：opacity:0</div>
  <div>第五个：正常显示</div>
</div>

上面看到的空档，就是 `visibility: hidden` 和 `opacity: 0` 留下的位置。

怎么选：

| 想干的事 | 用哪个 |
| --- | --- |
| 收起一块内容，布局重排 | `display: none` |
| 保留位置，只是暂时看不见 | `visibility: hidden` |
| 做淡出动画 | `opacity: 0` |
| 藏起来但键盘还能聚焦 | `opacity: 0` 或 `clip-path` |

有个无障碍上的小坑：`opacity: 0` 的元素，键盘按 Tab 还能聚焦到它，用户会在一个"看不见的地方"按到按钮。真的想藏起来就用 `display: none`。

## 现代布局的两个值

```css
.row { display: flex; }
.grid { display: grid; }
```

`flex` 和 `grid` 也是 `display` 的值，它们开启了全新的排版模式，后面两篇专门讲。

## 其他值

| 值 | 用途 |
| --- | --- |
| `inline-flex` | 行内排列的 flex 容器 |
| `inline-grid` | 行内排列的 grid 容器 |
| `table-cell` | 让元素表现得像表格单元格，常用于垂直居中 |
| `contents` | 元素本身不生成盒子，让子元素直接参与外层布局 |
| `list-item` | 表现得像列表项 |

`table-cell` 是 flex 出现之前做垂直居中的常见办法，现在直接用 flex 更简单。

延伸阅读：[CSS Display - 菜鸟教程](https://www.runoob.com/css/css-display-visibility.html)，[MDN 的 display](https://developer.mozilla.org/zh-CN/docs/Web/CSS/display)。

---

> 系列第 8 / 14 篇 · ← 上一步：[[07-列表、表格和链接的样式|列表、表格和链接]] · 下一步：[[09-定位，position 的五个值|定位]] → · 路线图：[[CSS 总览]]
