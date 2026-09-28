---
title: "CSS 过渡与主题切换"
date: 2026-09-08
authors: [workbuddy]
tags: []
draft: false
---

> 原文：[[CSS-Transition-and-Theme-Switching]]

# CSS 过渡与主题切换

## 问题

一条全局规则给所有元素加上颜色过渡，让主题切换更顺滑：

```css
* { transition: background-color .25s, color .25s, border-color .25s; }
```

然后某个区域（一个面板、一个 canvas 查看器）为了修视图切换的闪烁，被加上了 `transition: none !important` —— 于是这个区域**瞬间跳变**，而其余部分都在渐变。这种不一致读起来就是又一次闪烁。

## 规则

永远不要用 `transition: none !important` 去对抗全局过渡，尤其是不要加在整块区域上。要么：

- 让这个区域跟页面一起过渡，要么
- 把逻辑挪到 JavaScript 里（在一帧之内完成像素/类名的替换，浏览器会原子性地渲染出来）。

## canvas 的情况

位图内容（PDF 页面、SVG 图谱）没有可供过渡的 CSS 颜色。那里的主题切换必须在代码里处理——在同一个同步代码块里，先换掉已渲染的像素，再翻主题 class。没有中间帧，也就没有闪烁。

## 相关

- [[明暗模式切换时页面闪烁的成因与统一过渡方案]]
- [[为什么 SVG 图标点击时闪烁]]
- [[防止 Firefox 刷新时夜间模式闪烁]]
