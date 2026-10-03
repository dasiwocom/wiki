---
title:
aliases:
tags:
  - css
description: float 怎么让文字绕图，父元素高度为什么会塌，清浮动有哪几种写法，以及现在还需不需要用它。
---

# 浮动 float，以及它现在的用途

`float` 是上一个时代的布局主力，现在大部分活被 flex 和 grid 接走了。但它还剩一个非常好的用途。

## 最初的设计目的

`float` 本来就是为"文字环绕图片"设计的：

```css
img {
  float: left;
  margin-right: 16px;
}
```

↓ 效果：

<style>
.wbd-float-demo img, .wbd-float-demo .fakeimg {
  float: left;
  width: 96px;
  height: 96px;
  margin: 0 16px 8px 0;
  border-radius: 6px;
  background: linear-gradient(135deg,#84a59d,#284b63);
}
.wbd-float-demo::after { content: ""; display: block; clear: both; }
</style>

<div class="wbd-float-demo">
  <div class="fakeimg"></div>
  这段文字会绕着左边的方块排。浮动元素脱离正常的文档流，后面的行内内容会紧贴着它的另一侧填进来，这就是"文字环绕"。杂志排版里那种图配文，用的就是这个特性。把窗口拉窄，文字会自动在方块下面继续，不需要额外设置。
</div>

`float: right` 就是绕到另一边。

现代网页里，这个用法依然值得留着——新闻正文里配一张小图，用它比 flex 省事得多。

## 它当年是怎么被拿去布局的

```css
.left  { float: left;  width: 200px; }   /* 侧栏 */
.right { float: right; width: calc(100% - 200px); }  /* 正文 */
```

在 flex 出现之前，两栏、三栏、整站布局都是这么拼出来的。

代价是有一堆副作用要处理，其中最烦的是下面这个。

## 父元素高度塌陷

浮动元素脱离了正常文档流，**父元素算高度的时候直接无视它**：

一次浮动之后，父元素的高度会变成 0，下面的内容顶上来，把布局压乱。

```html
<div class="parent">
  <div class="child" style="float:left">我会飘出去</div>
</div>
<p>父元素高度是 0，这段会盖上来</p>
```

## 清浮动

要让父元素重新把浮动子元素"包住"，传统上有三种写法。

**① 给父元素加 clearfix（最经典）**

```css
.clearfix::after {
  content: "";
  display: block;
  clear: both;
}
```

**② 给父元素加 `overflow: hidden`**

```css
.parent { overflow: hidden; }
```

副作用是超出的内容会被裁掉，做下拉菜单时会出问题。

**③ 用 `display: flow-root`（现代写法）**

```css
.parent { display: flow-root; }
```

一条声明解决，没有副作用。新代码优先用它。

## clear：不许挨着浮动元素

```css
.footer { clear: both; }
clear: left;   /* 不许左边有浮动 */
clear: right;  /* 不许右边有浮动 */
```

含义是"这个元素的上边缘不能和浮动的元素并排"。用了它，元素会被挤到浮动元素下面去。

顺带说一句命名：`clear` 是"清除浮动的影响"，不是"删掉浮动元素"，别理解反了。

## 现在怎么选

| 想干的事 | 用什么 |
| --- | --- |
| 文字绕图 | `float`（它本来就是这个用途） |
| 横向排列、两端对齐 | `flex` |
| 整页分区网格 | `grid` |
| 元素居中 | `flex` + `justify-content` / `align-items` |

一个简单的判断法：**要排的元素是不是"文字和图的混排"**？是就用 `float`；不是，就用 flex 或 grid。

用 float 做整站布局，是十几年前的技术债，现在没人这么写了。

## 顺便说一句：inline-block 的缝

不用 float 之后，有人会用 `inline-block` 横向排列。它有个怪毛病：**元素之间会多出几像素的空白**。

原因是标签之间的换行符被当成了一个空格。

解决办法有几个，最简单的是让父元素去管间距：

```css
.list { display: flex; gap: 8px; }
```

这也是"布局交给 flex"的一个理由。

延伸阅读：[CSS Float 浮动 - 菜鸟教程](https://www.runoob.com/css/css-float.html)，[CSS 布局](https://www.runoob.com/css/css-website-layout.html)（用 float 的经典布局写法）。

---

> 系列第 10 / 14 篇 · ← 上一步：[[09-定位，position 的五个值|定位]] · 下一步：[[11-flex 布局，把东西横着排|flex 布局]] → · 路线图：[[CSS 总览]]
