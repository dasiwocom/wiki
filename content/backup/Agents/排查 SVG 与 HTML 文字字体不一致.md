---
title: "排查 SVG 与 HTML 文字字体不一致"
date: 2026-09-08
authors: [workbuddy]
tags: []
draft: false
---

> 原文：[[Debugging-Font-Mismatch-Between-SVG-and-HTML-Text]]

# 排查 SVG 与 HTML 文字的字体不一致

## 现象

知识图谱的节点标签用 SVG `<text>` 渲染，侧边目录树用 HTML `<a>`。用户反馈标签看起来不一样——字号不对、字重不对、行高不对——不管 CSS「修」了多少次都没用。

## 为什么一直修不好

陷阱在于你是在用**眼睛**比，而不是用**数值**比：

1. **SVG 文字和 HTML 文字的字体度量不同。** 同样的 `font-size: 15px; font-weight: 600`，在某些浏览器里 SVG 里就是明显更细，于是你把字重加到 700，再猜到 800，每次都是猜。
2. **`line-height` 是 HTML 的概念。** SVG 文字没有同样意义上的行盒；浏览器照样会给出一个计算出来的 `line-height`，但那是字体的默认值，不是 HTML 布局用的那个。这个差异肉眼看不出来，只有 DevTools 能看到。
3. 每一次「修复」都是一次猜测，而每次猜测都要走一个来回：改 CSS → 部署 → 用户刷新 → 还是不对 → 接着猜。

## 修法：别猜，读数值

在 DevTools 里分别打开两个元素，读它们的**计算样式**：

| 属性 | 目录树（HTML `<a>`） | 图谱标签（SVG `<text>`） |
| --- | --- | --- |
| font-size | 17px | 15px |
| font-weight | 700 (Bold) | 600 (SemiBold) |
| line-height | 28.9px | 25.5px |

然后按这些数值原样设置 SVG 标签：

```css
.graph-label {
    font-family: 'Nunito', sans-serif;
    font-size: 17px;
    font-weight: 700;
    line-height: 28.9px;   /* SVG 虽然没有行盒，但计算样式里能读到这个值 */
    text-rendering: optimizeLegibility;
}
```

一次改动，对着真实数值验证，搞定。不用猜。

## 教训

**不同渲染上下文之间的视觉样式不一致（SVG vs HTML、canvas vs DOM），必须用测量值调试，不能靠肉眼。** 让用户把 DevTools 里的数值发给你（或者你自己去读）——拿到确切数值，多轮猜谜就变成了一次修好。

## 附赠一个坑：规则被塞进了另一条规则的花括号里

就是这次调试多烧了一个小时：修补的时候，那条修复用的 CSS 规则被插到了**上一条规则的花括号内部**：

```css
.drawer-md a {
    font-size: 15px;
    /* BUG: 这一行在 .drawer-md a 的块内部 —— 是无效声明，会被静默忽略 */
    #front-drawer-md > ul > li:first-child > a { font-size: 17px; font-weight: 700; }
    overflow: hidden;
}
```

浏览器把嵌套的选择器当成垃圾，**静默忽略**。症状和缓存问题完全无法区分：刷新、无痕窗口、加缓存穿透的 query string —— 都没变化，因为这条 CSS 从来就没生效过。

**怎么抓到**：`php -l` 只校验 PHP，不校验 CSS —— 它会通过。去读周围的花括号，或者 grep 出这条规则，确认它是独立的一个块（没有被缩进塞在另一个块里面）。

**经验规则**：任何 CSS 补丁之后，先确认新规则的嵌套层级是对的，再去怪缓存。

## 相关

- [[图谱视图力导向布局笔记]]
- [[为什么 SVG 图标点击时闪烁]]
- [[CSS 过渡与主题切换]]
