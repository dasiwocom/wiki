---
title: "在 Markdown 知识库中渲染 Excalidraw 图纸"
date: 2026-09-08
authors: [workbuddy]
tags: []
draft: false
---

> 原文：[[Rendering-Excalidraw-Drawings-in-a-Markdown-Knowledge-Base]]

# 在 Markdown 知识库中渲染 Obsidian Excalidraw 图纸

## 目标

一个自托管的 Markdown 知识库已经能渲染 `.md`、`.pdf` 和图谱视图。现在再加一种：**Obsidian Excalidraw 插件保存的绘图**（`.excalidraw.md`）—— 在目录树里点一张图，就能看到和在 Obsidian 里一模一样的可交互 SVG。

## 文件格式（第一个坑）

一个 Obsidian Excalidraw 文件长这样：

```markdown
---
excalidraw-plugin: parsed
tags: [excalidraw]
---
==⚠ Switch to EXCALIDRAW VIEW...
# Text Elements
G ^xYm6DtRp
C ^g9LmDvct
...
## Drawing
```compressed-json
N4KAkARALgngDgUwgLgAQQQDwMYEMA2AlgCYBOuA7hADTgQBuCpAzoQPYB2Kq...
```
```

场景数据躺在一个 `compressed-json` 代码块里。**它不是 zlib、gzip、brotli 或裸 deflate。** 开头几个字节（`37 82 80 90...`）能击败所有标准解压器。它是 **lz-string**（`LZString.decompressFromBase64`）。在 Node 里把所有候选试一遍就能认出来：

```js
// inflate / inflateRaw / gunzip / brotli 全失败 —— lz-string 成功
const scene = JSON.parse(LZString.decompressFromBase64(b64.replace(/\s+/g, '')));
// => { type: "excalidraw", version: 2, elements: [...] }
```

把 `lz-string.min.js`（约 5 KB）下载到站点的 assets 目录。

## 架构

1. **PHP**：在 URL 里识别 `.excalidraw.md`（要把它从普通 `.md` 分支里排除出去 —— 两者都以 `.md` 结尾），把原始文件内容内联成一个 JS 变量。
2. **前端**：`LZString.decompressFromBase64` → 解析场景 → 拼出 SVG 字符串 → 插入容器。
3. 视图切换照抄 PDF 阅读器：隐藏 markdown 视图，显示绘图容器，用文件名设置标题。

渲染成 SVG 的元素类型：`text`、`arrow`、`line`、`rectangle`、`ellipse`、`diamond`。箭头需要 `<marker>`；多点箭头必须用 `<polyline>`（只在首尾两点之间画 `<line>` 会把曲线压平）。

## 踩过的坑（每一个都花掉了真实的调试时间）

### 1. 绑定在直线上的文字，渲染在**直线的中点**上，而不是它保存的坐标

这是最大的一个。某些字母（F、I、H、J、A）偏离它们的直线最多 253px。这些文字保存的 `text.x/y` **没有吸附到直线上**，但 Obsidian 却把它们显示在直线上。为什么？

因为 Excalidraw 的**绑定文字**机制：文字元素有个 `containerId` 指向它所在的那条直线；直线的 `boundElements` 里列着这个文字。引擎渲染绑定文字时把它**居中在直线的中点上**，忽略未吸附的保存坐标。检测方法：

- **不要**去检查 `text.boundElementIds` —— 它是空的。要检查 `text.containerId`（那条直线的 id）和直线的 `boundElements`。
- 建一张 `lineId -> midpoint` 映射，然后对所有带 `containerId` 的文字，把它的盒子中心换成直线中点：

```js
const lineMid = {};
els.forEach(a => {
  if (a.type !== 'arrow' && a.type !== 'line') return;
  const pts = a.points || [[0,0],[100,0]];
  lineMid[a.id] = { x: (a.x + pts[0][0] + a.x + pts.at(-1)[0]) / 2,
                    y: (a.y + pts[0][1] + a.y + pts.at(-1)[1]) / 2 };
});
// 渲染文字时：if (e.containerId && lineMid[e.containerId])
//   box center = lineMid[e.containerId]  (left = mid.x - w/2, top = mid.y - h/2)
```

**用几何验证，别用眼睛**：在数据里算出每个文字的盒子中心和最近的直线中点的距离。距离是 0 的那些字母属于「没吸附但没事」；距离不为 0 的，正好就是用户报告有问题的那几个。

### 2. `verticalAlign: "middle"` 是个陷阱 —— y 仍然是盒子顶部

数据里大多数文字都带 `verticalAlign: middle`，看起来像是「y 就是中心」。但测量盒子中心到直线中点的距离，可以证明 y 是顶部（按顶部渲染时距离为 0.0）。别急着靠位移去「修」—— 先对着几何验证。

### 3. 精确定位离不开字体

Excalidraw 是用它自己的手写字体 **Virgil** 来测量文字宽高的。没有这个字体，回退字体会让居中文字偏移。下载 `Virgil.woff2`（从 jsDelivr 上的 `@excalidraw/excalidraw` npm 包里取）并用 `@font-face` 注册。`text-anchor:middle` 能固定 x 中心，但高度/度量仍然会影响 y。

### 4. `dominant-baseline` 不可靠 —— 自己补偿基线

`dominant-baseline: text-before-edge` 在部分 WebView（老版微信/Chromium）里会被忽略，静默回退到字母基线，把字形上移约 20px。不要依赖它，改用正常基线渲染 + 手动补偿：

```js
// 字体加载完成后用 canvas 测量真实的 ascent（兜底 0.9）
const y = textTop + ascentRatio * fontSize;   // 字形顶部 ≈ textTop
```

先 `await document.fonts.load('20px Virgil')`，再用 `canvas.getContext('2d').measureText(...).actualBoundingBoxAscent` 测量。

### 5. 移动端 WebView 的缓存会藏住每一次修复

用户连续几轮都反馈「没变化」。代码每轮都在改，但微信 WebView 的缓存每次都把旧页面端出来。打破这个循环的办法：让用户在手机的真浏览器（Safari/Chrome）里打开 URL，或者加上 `?v=timestamp`。先用 curl 在服务端验证修复是否生效，再区分缓存问题和代码问题。

### 6. 别忘了把容器显示出来

容器初始是 `display:none`；`innerHTML = svg` 之后要设 `style.display = 'block'`，否则页面一片空白且不报错。

## 要点总结

1. 写代码之前先识别压缩格式 —— 在 Node 里把每种候选（zlib/gzip/brotli/lz-string）都试一遍。
2. 绑定相关的字段要全读：`boundElementIds` vs `containerId` vs `boundElements` —— 它们是同一套机制的不同侧面。
3. 渲染位置要对着场景几何**用数学验证**（字母中心 vs 直线中点），而不是靠猜 + 反复重新部署。
4. 用户反复说「没变化」时，先怀疑客户端缓存，再动代码。

## 相关

- [[排查 SVG 与 HTML 文字字体不一致]]
- [[为 Markdown 知识库构建图谱视图]]
- [[Markdown 站点的服务端渲染]]
