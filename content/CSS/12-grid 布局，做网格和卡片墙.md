---
title:
aliases:
tags:
  - css
description: grid 的行列定义、fr 单位、跨行跨列、命名区域，以及用 minmax 和 auto-fit 做出不用媒体查询的卡片墙。
---

# grid 布局，做网格和卡片墙

flex 管一根轴，grid 管两根轴。要同时控制行和列，用 grid 比 flex 干净。

## 两行开一个网格

```css
.grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;   /* 三列，等宽 */
  gap: 12px;
}
```

↓ 效果：

<style>
.wbd-grid { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 12px; margin: 12px 0; }
.wbd-grid > div { padding: 16px; background: rgba(40,75,99,.1); border: 1px solid #b8b8b8; border-radius: 6px; text-align: center; }
.wbd-grid-span > div:nth-child(1) { grid-column: span 2; }
.wbd-grid-span > div:nth-child(4) { grid-row: span 2; }
</style>

<div class="wbd-grid">
  <div>1</div><div>2</div><div>3</div>
  <div>4</div><div>5</div><div>6</div>
</div>

写了几列，子元素就自动按行填进去，不需要指定每个元素放哪。

## fr 单位

`fr` 是"剩下多少份"的意思，只在 grid 和 flex 里能用的单位。

```css
grid-template-columns: 1fr 2fr;   /* 右边占两倍宽 */
grid-template-columns: 200px 1fr; /* 左边固定，右边吃掉剩下全部 */
```

比写百分比省事，也不会有"加起来超过 100% 撑破容器"的问题（`gap` 会被自动算进去）。

## repeat 简写

```css
grid-template-columns: repeat(3, 1fr);   /* 等于 1fr 1fr 1fr */
grid-template-columns: repeat(4, 80px);
```

列多的时候省事，改成 `repeat(5, 1fr)` 就变五列。

## 跨行跨列

```css
.item { grid-column: span 2; }   /* 横向占两列 */
.item { grid-row: span 2; }      /* 纵向占两行 */
```

也可以用"从第几条线到第几条线"的写法：

```css
.item { grid-column: 1 / 3; }    /* 从第 1 条网格线到第 3 条，也就是占两列 */
```

↓ 第一块横跨两列、第四块纵跨两行：

<div class="wbd-grid wbd-grid-span">
  <div>1（横跨两列）</div><div>2</div>
  <div>3</div>
  <div>4（纵跨两行）</div><div>5</div><div>6</div>
  <div>7</div><div>8</div>
</div>

注意：自动放置时，被跨过的位置如果放不下后面的元素，会留出空档。需要精确控制时给它指定 `grid-column` 和 `grid-row`。

## 行高

```css
.grid {
  grid-auto-rows: 100px;              /* 每一行固定 100px */
  grid-template-rows: 60px 1fr 60px;  /* 或者显式指定头、身、尾 */
}
```

`grid-auto-rows` 管"没显式指定的行"，做等高卡片列表时很有用：

```css
grid-auto-rows: minmax(120px, auto);
```

意思是"至少 120px，内容多了就自动长高"。

## 命名区域：把布局画出来

这是 grid 最好用的功能之一。想清楚页面分几块，直接把它"画"成一张图：

```css
.layout {
  display: grid;
  grid-template-columns: 200px 1fr;
  grid-template-areas:
    "sidebar header"
    "sidebar main"
    "footer  footer";
  gap: 12px;
}
.sidebar { grid-area: sidebar; }
.header  { grid-area: header; }
.main    { grid-area: main; }
.footer  { grid-area: footer; }
```

`grid-template-areas` 里的每一行字符串对应一行，每个词对应一块。同名的格子会自动合并成一块区域。

好处是改布局只要改那三行字符串——比如手机上想要上下堆叠，写一版新的区域图就完事，HTML 一个字都不用动。

## 不用媒体查询的响应式卡片墙

这是 grid 最漂亮的一个用法：

```css
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 16px;
}
```

拆开看：

| 部分 | 意思 |
| --- | --- |
| `auto-fit` | 一行能塞几张卡片，就自动放几张 |
| `minmax(240px, 1fr)` | 每张卡片最小 240px，最大可以平分剩余空间 |

结果是：屏幕宽的时候并排四张，窄了自动变三张、两张、一张，**全程不需要写任何媒体查询**。

↓ 拉一下浏览器窗口宽度，看它们怎么自动重排：

<style>
.wbd-cards { display: grid; grid-template-columns: repeat(auto-fit, minmax(160px, 1fr)); gap: 12px; margin: 12px 0; }
.wbd-cards > div { padding: 20px 12px; background: rgba(132,165,157,.2); border-radius: 8px; text-align: center; }
</style>

<div class="wbd-cards">
  <div>卡片 1</div><div>卡片 2</div><div>卡片 3</div>
  <div>卡片 4</div><div>卡片 5</div><div>卡片 6</div>
</div>

`auto-fit` 和 `auto-fill` 的区别：一行放不满时，`auto-fit` 会把空轨道塌陷、让卡片撑宽；`auto-fill` 会保留空位、卡片保持原宽。多数情况下想要的是 `auto-fit`。

## flex 还是 grid

| 场景 | 选 |
| --- | --- |
| 一排按钮、一个导航栏 | flex |
| 一行文字加个图标 | flex |
| 卡片墙、相册、表单网格 | grid |
| 整页分区（头、侧栏、主体、页脚） | grid |
| 不知道有几项、只要求横着排 | flex |

一句话：**只管一个方向用 flex，同时管行和列用 grid**。

两者不冲突，可以嵌套——grid 的某个格子里放一个 flex 容器，是很常见的结构。

延伸阅读：[CSS 网格视图 - 菜鸟教程](https://www.runoob.com/css/css-rwd-grid.html)，[CSS 网格布局 - 菜鸟教程](https://www.runoob.com/css3/css3-grid.html)，[MDN 的 grid 布局](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_grid_layout)，[Grid Garden](https://cssgridgarden.com/#zh-cn)（练习用的小游戏）。

---

> 系列第 12 / 14 篇 · ← 上一步：[[11-flex 布局，把东西横着排|flex 布局]] · 下一步：[[13-响应式，让页面适应各种屏幕|响应式]] → · 路线图：[[00-CSS 总览]]
