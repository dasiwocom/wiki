---
title: "为 Markdown 知识库构建图谱视图"
date: 2026-09-08
authors: [workbuddy]
tags: []
draft: false
---

> 原文：[[Building-a-Graph-View-for-a-Markdown-Knowledge-Base]]

# 为 Markdown 知识库构建图谱视图

## 目标

给自建的 Markdown 知识库加一个 Obsidian 风格的知识图谱：每篇笔记是一个节点，每个 `[[双链]]` 是一条边，节点靠力导向模拟慢慢飘到各自的位置，可以拖拽 / 缩放 / 平移，点击节点就打开那篇笔记。

## 架构（三块）

```
GET /api/graph          → 扫描仓库、解析 [[双链]]，返回 {nodes, links}
/graph（虚拟 URL）       → nginx 重写到 index.php；SSR 识别出来后通知前端
前端 SVG 渲染器          → 力导向模拟 + 交互（零依赖）
```

### 1. 数据层 — `GET /api/graph`

扫描仓库里每个 `.md`（复用已有的目录树扫描器），排除隐藏路径，然后解析链接：

```php
preg_match_all('/\[\[([^\]\|#]+)(?:\|[^\]]*)?\]\]/u', $content, $m)
```

用**文件名**去匹配目标（`Note-Name` 能匹配到 `dir/Note-Name.md`），这样笔记小范围移动后链接依然有效。无向边要去重（用 `source<target` 当 key）。可选的 `?dir=` 参数用来限定范围。

### 2. 路由层 — `/graph` 是虚拟页面，不是真实文件

单一 PHP 入口根据 URL 决定渲染什么：

- `/xxx.md` → 文章 SSR
- `/xxx.pdf` → PDF 阅读器 SSR
- `/graph` → 图谱 SSR（内联 `SSR_GRAPH = true`，前端据此打开图谱视图）

> [!danger] 坑
> nginx 需要显式加一条规则：`location = /graph { rewrite ^(.*)$ /index.php last; }`
> 不加的话，这个 URL 会落到默认 location，去找文件，然后在 PHP 执行之前就 404 了。

### 3. 前端 — 力导向 SVG，零依赖

每帧（或打开时一次性）迭代物理力：

- **斥力**：每对节点之间（O(n²) —— 几十个节点没问题）
- **弹簧引力**：沿着连线
- **向心重力**：防止整簇飘走
- **阻尼**：能量损耗 —— 没有它就会永远振荡
- 可选的**同目录弱吸引**（按文件夹聚拢笔记，但**不画线**）

交互：拖拽节点（pointer 事件）、滚轮以光标为中心缩放、空白处拖拽平移、移动端双指捏合（两指距离比）、悬停显示标签并高亮邻居、点击打开笔记（真实拖拽之后要抑制点击 —— 见下文）。

## 踩过的坑（按时间顺序）

### 1. 相对资源路径在嵌套 URL 下失效

写成 `assets/foo.css` 的资源是相对当前 URL 解析的 —— 在 `/guide/note.md` 下会变成 `/guide/assets/foo.css` → 404 → JS 包加载失败 → **所有东西都挂了**（白屏，没有图谱）。
**修法**：所有地方一律用绝对路径 `/assets/...`。

### 2. 图谱容器被塞进了一个隐藏面板 —— 白屏排查了好几个小时

某次重构时，图谱的 `<div>` 被插进了目录（TOC）面板里面，而它的父元素是 `display:none`。**只要祖先是 `display:none`，整棵子树都隐藏** —— 不管子元素自己设了什么 `display` / `z-index` / `!important`，都不会渲染。

> [!tip] 调试教训
> 当一个元素设了 `position:fixed` 和 `background:red` 却依然不显示时，去检查**它到底挂在 DOM 的哪个位置**，而不是盯着它自己的样式看。

### 3. 高度塌缩

在没有显式高度的 flex 容器里，子元素写 `height:100%` 会被解析成 `auto` → 0 → 什么都看不见。
**修法**：`position:fixed; top:56px; left:0; right:0; bottom:0` —— 固定定位能给出确定的尺寸，不受父级布局影响。

### 4. 模拟永不收敛 —— 节点一直弹

参数不对：斥力太强、向心重力太弱、阻尼太小，节点在各种力的拉扯中持续振荡。
**修法**：适度的斥力、更强的向心重力、`DAMP 0.85–0.9`、给每个节点加速度上限，再加**硬帧数上限**（比如 200–600 帧）保证模拟一定会停下来。然后用向心重力调"松紧"（0.03 = 紧凑，0.015 = 松散、不那么果冻感）。

### 5. 拖拽会触发点击 —— 拖着拖着就跳转了

浏览器在同一个元素上完成 `pointerdown` + `pointerup` 后会触发 `click`，于是拖拽结束时发生了页面跳转。
**修法**：记录指针移动距离，超过约 5px 就标记该节点为"已拖拽"，并抑制点击处理器。

### 6. 移动端拖拽卡顿

原始的 `pointermove` 处理器触发频率 120Hz+，每一次都更新 DOM → 主线程被堵死。
**修法**：用 `requestAnimationFrame` 合并移动事件（只存最新坐标，每帧应用一次），并在画布上加 `touch-action:none`，避免浏览器用滚动/手势跟拖拽抢控制权。

### 7. 拖拽很"死" —— 只有被拖的那个节点在动

为了性能在拖拽时停掉了模拟，结果 Obsidian 那种手感没了。
**修法**：拖拽时每帧跑一步力学计算（34 个节点开销很小），并且只更新速度超过阈值的节点（`updateMovingEls`）—— 图谱有响应，又不用重绘全部。

### 8. SVG 标签和 HTML 树的字体不一致

SVG 的 `<text>` 和 HTML 文本的渲染度量不同，靠肉眼估差异会陷入无止境的猜测（参见 [[Debugging-Font-Mismatch-Between-SVG-and-HTML-Text]]）。
**修法**：在 DevTools 里读计算后的样式（17px / 700 / line-height 28.9px），然后把 SVG 标签设成一模一样的值。

### 9. 观感上的卡顿

- 30fps 的模拟（隔帧跑）看起来就是卡 → 改成每帧都跑
- `setAttribute('transform')` 会触发属性和布局更新 → 把节点移动改成 CSS 的 `style.transform`（走合成器 / GPU 路径；因为 SVG 没有 viewBox，CSS 像素等于 SVG 单位，所以安全），再给容器组加 `will-change: transform`

## 最终效果

- 打开 `/graph`：布局在绘制前就基本稳定（220 次预迭代），之后轻轻收敛一下就静止
- 拖动节点：邻居跟着动（实时力学），悬停高亮显示关系网，背景变暗
- 滚轮 / 捏合缩放，标签在缩放低于阈值时自动隐藏（Obsidian 的行为，后台可开关）
- 点击节点（非拖拽）：打开笔记
- 后台：图谱设置视图（显示 / 隐藏文件名）

## 经验总结

1. **白屏 + 代码没问题**，通常是元素**挂错了 DOM 容器**或者**资源 404** —— 先查这两项，别急着怀疑逻辑
2. 力导向模拟必须有**硬停止条件**（帧数上限 + 速度阈值），否则会永远振荡
3. 移动端流畅 = 合并指针事件 + `touch-action` + 只更新在动的元素
4. SVG 和 HTML 的视觉对齐 = 用 DevTools 量出来的值，**绝不靠肉眼**

## 相关笔记

- [[Graph-View-Force-Directed-Layout-Notes]]
- [[Debugging-Font-Mismatch-Between-SVG-and-HTML-Text]]
- [[Markdown-Wikilinks-Connect-Your-Knowledge]]
- [[Server-Side-Rendering-for-Markdown-Sites]]
