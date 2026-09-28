---
title: "为什么 SVG 图标点击时闪烁"
date: 2026-09-08
authors: [workbuddy]
tags: []
draft: false
---

> 原文：[[Why-SVG-Icons-Flicker-on-Click]]

# 为什么 SVG 图标点击时会闪烁

## 现象

用内联 SVG 做的图标按钮，在移动端浏览器（iOS Safari / WebView）上点击时会短暂闪一下，而同一个页面在桌面端却很顺。

## 原因

点击会触发样式/布局的重新计算，浏览器会为这一帧重绘 SVG 图标。在 iOS 上，这个重绘表现为一次闪光，因为该图标活在主合成层上。

## 修法

把图标强制推到它自己的 GPU 图层上，这样重绘永远不会到达屏幕：

```css
.vp-icon-btn svg {
    -webkit-backface-visibility: hidden;
    backface-visibility: hidden;
}
.vp-icon-btn { transform: translateZ(0); }
```

当图标本身带 `transform` 动画（形变动画）时，**不要**把 `translateZ(0)` 加在 SVG 自己身上 —— 它会覆盖掉动画。按钮才是承载这个图层提示的安全宿主。

## 相关

- [[明暗模式切换时页面闪烁的成因与统一过渡方案]]
- [[防止 Firefox 刷新时夜间模式闪烁]]
- [[CSS 过渡与主题切换]]
