---
title:
aliases:
tags:
  - css
description: static、relative、absolute、fixed、sticky 五个定位值分别相对谁、脱离不脱离文档流，以及 z-index 为什么有时不生效。
---

# 定位，position 的五个值

前面讲的都是"元素按顺序往下排"。想让某个东西精确地待在某个位置，就要用到 `position`。

## 五个值

```css
.a { position: static; }    /* 默认，老老实实按文档流排 */
.b { position: relative; }  /* 相对自己原来的位置挪 */
.c { position: absolute; }  /* 脱离文档流，相对最近的定位祖先放 */
.d { position: fixed; }     /* 相对浏览器窗口固定，滚动不动 */
.e { position: sticky; }    /* 平时正常，滚到某个位置就粘住 */
```

配合使用方向属性：`top` `right` `bottom` `left`。

## relative：挪一下，位置还留着

```css
.box {
  position: relative;
  top: 10px;
  left: 20px;
}
```

元素会往下 10px、往右 20px，但**它原来占的位置还空着**，后面的元素不会补上来。

`relative` 单独用的时候不多，它最主要的作用是"给里面的绝对定位元素当参照物"。

## absolute：脱离文档流

```css
.badge {
  position: absolute;
  top: 0;
  right: 0;
}
```

元素完全脱离原来的排版，后面的内容会当它不存在。

它相对谁定位，看这条规则：

> 往上找，找到**第一个 `position` 不是 `static` 的祖先**，就相对那个祖先的边框内沿定位。找不到就相对 `<html>`。

所以做角标要这样写：

```css
.card { position: relative; }     /* 给它一个定位身份 */
.badge {
  position: absolute;
  top: -8px;
  right: -8px;
}
```

只写 `.badge` 的绝对定位，不写父元素的 `relative`，角标会跑到整个页面右上角去。

看效果：

<style>
.wbd-pos-card {
  position: relative;
  display: inline-block;
  padding: 16px 24px;
  border: 1px solid #dcdcdc;
  border-radius: 8px;
  background: #fff;
  margin: 16px 0;
}
.wbd-pos-card .badge {
  position: absolute;
  top: -10px;
  right: -10px;
  background: #c0392b;
  color: #fff;
  font-size: 12px;
  padding: 2px 8px;
  border-radius: 999px;
}
</style>

<div class="wbd-pos-card">
  一张卡片
  <span class="badge">NEW</span>
</div>

## 绝对定位居中

想让一个东西在父元素里水平垂直都居中，可以这样：

```css
.parent { position: relative; }
.center {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

原理：先把元素的左上角挪到父元素正中，再用 `translate(-50%, -50%)` 把自己往左、往上各挪回自身尺寸的一半。

现在更简单的做法是用 flex，见 [[11-flex 布局，把东西横着排|下一篇]]。

## fixed：钉在窗口上

```css
.back-to-top {
  position: fixed;
  right: 24px;
  bottom: 24px;
}
```

元素固定在浏览器窗口的某个位置，页面怎么滚它都不动。

"返回顶部"按钮、侧边的悬浮客服、固定在顶部的导航栏，都用它。

注意：`fixed` 相对的是**视口**，不是页面。父元素上如果写了 `transform`、`filter`、`perspective`，`fixed` 会改成相对那个父元素定位——这是排查"我的固定元素为什么不固定了"的第一个线索。

## sticky：滚到位置就粘住

```css
thead th {
  position: sticky;
  top: 0;
}
```

这是最实用的一个用途：让长表格的表头在滚动时一直可见。

`sticky` 的规则有点绕：它平时还是正常排版，当滚动到"距离某个方向超过阈值"时，就变成固定。

生效条件：

- `top`、`bottom`、`left`、`right` 至少要写一个
- 它的父元素不能是 `overflow: hidden`
- 父元素必须有足够的高度让它"走"进去

表格里还有一个前提：`<thead>` 的 `th` 要能生效，`border-collapse: collapse` 会让边框跟着滚走（这是浏览器的老问题），可以先试一下再看。

## z-index：谁在上面

元素重叠时，用 `z-index` 决定谁在上：

```css
.modal { position: fixed; z-index: 1000; }
.header { position: sticky; z-index: 100; }
```

数值大的在上面。可以是负数。

它不生效的两种原因：

**① 元素没有定位。** `z-index` 只对 `position` 不是 `static` 的元素生效，或者对 flex/grid 的子项生效。

**② 父元素创建了层叠上下文。** 父元素一旦有 `transform`、`opacity` 小于 1、`filter` 等属性，就形成了一个独立的"层叠上下文"，子元素的 `z-index` 再大也爬不出父元素这一层。

这条是 CSS 里最玄的问题之一。遇到"我 z-index 明明写得很大还是被盖住"，就去 F12 里顺着祖先一层层看是不是有人在"筑墙"。

## CSS 里最省事的调试方向

记住一句话就够了：**`absolute` 相对最近的定位祖先。**

90% 的定位问题，都是忘了给要当参照物的那个父元素写 `position: relative`。

延伸阅读：[CSS Position 定位 - 菜鸟教程](https://www.runoob.com/css/css-positioning.html)，[CSS 对齐](https://www.runoob.com/css/css-align.html)，[MDN 的 position](https://developer.mozilla.org/zh-CN/docs/Web/CSS/position)。

---

> 系列第 9 / 14 篇 · ← 上一步：[[08-display，块级、行内和隐藏|display]] · 下一步：[[10-浮动 float，以及它现在的用途|浮动]] → · 路线图：[[00-CSS 总览]]
