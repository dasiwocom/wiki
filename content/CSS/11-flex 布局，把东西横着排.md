---
title:
aliases:
tags:
  - css
description: flex 的主轴和交叉轴、justify-content 和 align-items 的区别、flex:1 怎么用、gap 和换行，以及为什么它是现在最常用的布局方式。
---

# flex 布局，把东西横着排

用 float 排横向布局要处理一堆副作用。flex 是专门解决这件事的，两行代码就能做出导航栏、卡片列表、居中。

## 开一个 flex 容器

```css
.row {
  display: flex;
}
```

这一行写在**父元素**上，它所有的**直接子元素**自动变成横向排列、等高。

看一个例子：

<style>
.wbd-flex { display: flex; gap: 8px; margin: 12px 0; }
.wbd-flex > div { padding: 10px 16px; background: rgba(40,75,99,.12); border-radius: 6px; border: 1px solid #b8b8b8; }
.wbd-flex.jc-center { justify-content: center; }
.wbd-flex.jc-between { justify-content: space-between; }
.wbd-flex.ai-center { align-items: center; height: 110px; }
.wbd-flex.ai-center > div:nth-child(2) { padding: 32px 16px; }
</style>

<div class="wbd-flex"><div>1</div><div>2</div><div>3</div></div>

加上 `justify-content: center` ：

<div class="wbd-flex jc-center"><div>1</div><div>2</div><div>3</div></div>

加上 `justify-content: space-between`（两端顶格，中间均分）：

<div class="wbd-flex jc-between"><div>1</div><div>2</div><div>3</div></div>

加上 `align-items: center`（第二个元素更高，看它们怎么对齐）：

<div class="wbd-flex ai-center"><div>1</div><div>2</div><div>3</div></div>

## 两根轴

flex 里所有的对齐属性，都是在描述"沿哪根轴对齐"。所以先把这两根轴记住：

```text
        ┌─────────────────────────── 主轴（main axis）──→
        │
   交叉轴 │   [ 1 ]   [ 2 ]   [ 3 ]
 （cross）│
        ↓
```

- **主轴**：默认是横向，`justify-content` 管它
- **交叉轴**：默认是纵向，`align-items` 管它

改 `flex-direction` 会把这根轴转过来：

```css
.column { flex-direction: column; }   /* 主轴变成纵向，从上往下排 */
```

## 容器上的属性

| 属性 | 作用 | 常用值 |
| --- | --- | --- |
| `display` | 开启 flex | `flex` |
| `flex-direction` | 主轴方向 | `row`（默认）`column` |
| `justify-content` | 主轴对齐 | `flex-start` `center` `space-between` `space-around` `space-evenly` |
| `align-items` | 交叉轴对齐 | `stretch`（默认）`flex-start` `center` `baseline` |
| `flex-wrap` | 放不下换不换行 | `nowrap`（默认）`wrap` |
| `gap` | 子元素之间的间距 | `16px` / `1rem 2rem` |
| `align-content` | 换行后每行怎么分布 | 只在多行时生效 |

**`gap` 是最值得用的一个。** 它只在子元素之间加间距，不用给第一个或者最后一个特别处理，比 `margin-right: 16px` 干净得多。

### justify-content 几个值的区别

```text
flex-start      [1][2][3]............
center          ......[1][2][3]......
space-between   [1]....[2]....[3]
space-around    ..[1]...[2]...[3]..
space-evenly    ...[1]..[2]..[3]...
```

- `space-between`：两端顶格，中间均分（导航栏、工具栏最常用）
- `space-around`：每项左右各留一份，所以边上的空隙是一半
- `space-evenly`：所有空隙完全相等

## 子元素上的属性

| 属性 | 作用 |
| --- | --- |
| `flex` | 简写，控制怎么分剩余空间 |
| `flex-grow` | 有剩余空间时，我分几份 |
| `flex-shrink` | 空间不够时，我缩多少 |
| `flex-basis` | 分空间之前，我先占多宽 |
| `align-self` | 单独设置我自己的交叉轴对齐 |
| `order` | 改排列顺序，数字小的在前 |

## flex: 1 是什么意思

```css
.item { flex: 1; }
```

等于 `flex-grow: 1; flex-shrink: 1; flex-basis: 0%`。

效果是"**把剩余空间全部分掉，所有写了 flex:1 的元素等宽**"。

经典用法，一个输入框配一个按钮，让输入框占满剩下的空间：

```css
.search { display: flex; gap: 8px; }
.search input { flex: 1; }         /* 输入框吃掉剩余宽度 */
.search button { flex: 0 0 auto; } /* 按钮按内容宽度，不参与分配 */
```

`flex: 0 0 auto` 的意思是"别长大、别缩小、宽度按内容算"，按钮该多宽就多宽。

其他组合：

| 写法 | 效果 |
| --- | --- |
| `flex: 1` | 平分剩余空间 |
| `flex: 2` | 分到的空间是 `flex:1` 的两倍 |
| `flex: 0 0 200px` | 固定 200px，不参与伸缩 |
| `flex: 1 1 auto` | 按内容宽度起步，再分配剩余 |

## 用 flex 做居中

横向纵向同时居中，三行搞定，这是 flex 最省事的地方：

```css
.center {
  display: flex;
  justify-content: center;   /* 主轴居中 */
  align-items: center;       /* 交叉轴居中 */
}
```

在 table-cell、绝对定位加 translate 之后，这是目前最清晰的写法。

## 换行

默认 `flex-wrap: nowrap`，子元素放不下会被**硬挤**，压缩到最小也不换行。

```css
.wrap { flex-wrap: wrap; }
```

加上它，放不下的元素会折到下一行。做标签云、卡片墙时必须加。

换行之后，`align-content` 才开始起作用，管的是"行与行之间怎么分布"。

## 一个完整的例子

```css
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  padding: 12px 24px;
  border-bottom: 1px solid #e5e5e5;
}
.nav-links {
  display: flex;
  gap: 20px;
  list-style: none;
  margin: 0;
  padding: 0;
}
```

八行就做出了一个标准的顶部导航。换成 float 得写二十行，还要处理清除浮动。

延伸阅读：[CSS flex 布局 - 菜鸟教程](https://www.runoob.com/css/css-rwd-flexbox.html)（含在线实例），[MDN 的 flex 布局基本概念](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_flexible_box_layout/Basic_concepts_of_flexbox)，[Flexbox Froggy](https://flexboxfroggy.com/#zh-cn)（用游戏的方式熟悉 flex 属性，二十关十分钟）。

---

> 系列第 11 / 14 篇 · ← 上一步：[[10-浮动 float，以及它现在的用途|浮动]] · 下一步：[[12-grid 布局，做网格和卡片墙|grid 布局]] → · 路线图：[[00-CSS 总览]]
