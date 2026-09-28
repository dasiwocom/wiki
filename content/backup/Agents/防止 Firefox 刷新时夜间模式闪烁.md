---
title: "防止 Firefox 刷新时夜间模式闪烁"
date: 2026-09-08
authors: [workbuddy]
tags: []
draft: false
---

> 原文：[[Preventing-Night-Mode-Flash-on-Refresh-in-Firefox]]

# 防止 Firefox 刷新时的夜间模式闪烁

## 现象

一个带手动明暗主题切换（用户选择存在 `localStorage` 里）的 Markdown 知识库站点，在深色模式下用 Firefox 刷新时，会**闪一帧浅色主题**：页面先短暂渲染成白色，然后才切到深色。文章标题下面那条分隔线也会闪一下浅色，然后才稳定。

Chrome 没有这个现象，只有 Firefox 有。

## 为什么 Firefox 会闪

两个浏览器都是通过 `<head>` 里的内联脚本给 `<html>` 加 `dark` class 来应用主题的：

```js
if (localStorage.getItem('vp-theme') === 'dark') {
    document.documentElement.classList.add('dark');
}
```

- **Chrome** 会阻塞首帧绘制，直到 head 脚本跑完，所以第一帧就已经带上 class 了 —— 不闪。
- **Firefox** 会在执行脚本**之前**先用*默认*样式画一个推测性的首帧，然后 class 才落上去，页面重新渲染成深色。第一帧是浅色 —— 就是那个闪。

换句话说：**Firefox 先画后想。** `<head>` 里的内联脚本还不够早，因为首帧根本不等它。

## 修法：用 cookie 在服务端宣告主题

稳妥的修法把 JS 从关键路径上彻底拿掉 —— 由服务端在 HTML 的前几个字节里宣告主题：

1. **主题每次变更时，JS 除写 `localStorage` 外再写一个 cookie**：
   ```js
   localStorage.setItem('vp-theme', dark ? 'dark' : 'light');
   document.cookie = 'vp-theme=' + (dark ? 'dark' : 'light') + '; path=/';
   ```

2. **PHP 读这个 cookie，直接把 class 输出到 `<html>` 标签上**：
   ```php
   <html lang="zh-CN" class="<?php echo (($_COOKIE['vp-theme'] ?? '') === 'dark') ? 'dark' : ''; ?>">
   ```

3. HTML 的第一帧就已经是深色的 —— Firefox 没什么可闪的。head 里的内联脚本保留下来，作为首次访问（还没有 cookie 时）的兜底。

## 修法内部的坑：`default_light` 与手动选择冲突

第一版试图用那个「强制默认浅色」的配置 flag 去把关服务端输出的 class：

```php
// 错：强制浅色的 flag 会静默取消掉手动选择的深色
class="<?php echo (($_COOKIE['vp-theme'] ?? '') === 'dark' && !$defaultLight) ? 'dark' : ''; ?>"
```

结果是：一个在站点配置写着「默认浅色」时手动切到深色的用户，仍然会闪 —— 服务端拒绝输出 `dark`，第一帧是浅色，只有跑得晚的 JS 才把深色恢复回来。

**教训**：cookie 代表的是用户*当前的实际选择* —— 绝不能被默认偏好 flag 过滤掉。默认值只在**完全没有选择**（既无 cookie 也无 `localStorage`）时才适用。

## 打个比方

之前的页面像一盏接在「初始为开」的开关上的灯：浏览器先画出一间亮着的屋子（默认浅色），然后脚本走过去把灯关掉（加 `dark`）—— 于是有一帧是亮的。cookie 方案等于递给浏览器一盏**一开始就是关着**的灯：第一帧就是暗的，没什么需要纠正的。

## 验证

```bash
# 带上深色 cookie → HTML 第一帧就带着 class
curl -s -b "vp-theme=dark" https://your.site/?v=test | grep -o '<html[^>]*>'
# → <html lang="zh-CN" class="dark">

# 不带 cookie → 默认浅色，class 为空
curl -s https://your.site/?v=test2 | grep -o '<html[^>]*>'
```

手动测试：切一次深色（写入 cookie），然后在 Firefox 里按 F5 —— 页面从第一帧起就是深色的。没有闪烁，包括标题分隔线（它只是另一条用 `var(--line)` 的边框，以前会用默认值闪一下）。

## 相关

- [[切换语言后刷新闪烁：与主题闪烁同源的 FOUC]]
- [[明暗模式切换时页面闪烁的成因与统一过渡方案]]
