---
title:
aliases:
tags:
  - html
description: 用 iframe 嵌别的网页或视频播放器，用 video 和 audio 放自己的媒体文件，以及这几类标签各自的注意点。
---

# 嵌入外部内容，iframe、视频和音频

HTML 里能开一个"窗口"，把别的地方的内容装进来。

## iframe：网页里的网页

```html
<iframe src="https://example.com" width="600" height="400"></iframe>
```

`<iframe>` 会在你页面中间挖一块区域，里面显示的是另一个完整的网页，它有自己的滚动条、自己的 CSS。

常见用途：

- 嵌地图
- 嵌视频播放器（B 站、YouTube 给出的"分享 → 嵌入"就是这段代码)
- 嵌表单（问卷、报名）

## iframe 的四个注意点

**必须写 title**：

```html
<iframe src="/map.html" title="公司位置地图"></iframe>
```

屏幕阅读器会把 iframe 当独立页面，没标题就不知道里面是什么。

**宽高要控住**，否则会变形或者溢出：

```html
<iframe
  src="//player.bilibili.com/player.html?bvid=BV1xx411c7mD"
  title="视频播放器"
  style="width:100%;aspect-ratio:16/9;border:0;border-radius:8px"
  loading="lazy"
  allowfullscreen>
</iframe>
```

`aspect-ratio:16/9` 让它始终保持 16:9 的比例，屏幕多宽都不会变形。

**默认带一圈边框**，不想要就写 `style="border:0"`。

**跨域限制**：iframe 里的页面和外面的页面互相拿不到对方的内容，这是浏览器的安全规则。想通信只能用 `postMessage`。

**别拿 iframe 做整站布局**，那是上个时代 `<frameset>` 的用法，现在会导致网址、书签、前进后退全乱掉。

## video：放自己的视频

```html
<video controls width="600" poster="cover.jpg">
  <source src="movie.mp4" type="video/mp4">
  <source src="movie.webm" type="video/webm">
  你的浏览器不支持 video 标签。
</video>
```

| 属性 | 作用 |
| --- | --- |
| `controls` | 显示播放、音量、进度条这些控件 |
| `width` / `height` | 播放器尺寸 |
| `poster` | 还没播放时显示的封面图 |
| `autoplay` | 自动播放，多数浏览器要求同时加 `muted` 才生效 |
| `loop` | 循环播放 |
| `muted` | 静音 |
| `preload` | 预加载策略：`none` `metadata` `auto` |

标签中间那一行文字是"兜底内容"，浏览器不支持 `<video>` 时才会显示出来。

`<video>` 和 `<source>` 是两回事：`<video>` 定一个播放器，里面可以放多个 `<source>` 提供不同格式，浏览器自己挑一个能放的。

现在主流只需要 `.mp4`（H.264 编码）就够了，兜底的 `.webm` 是给老浏览器准备的。

## 自动播放的坑

```html
<video src="a.mp4" autoplay muted playsinline></video>
```

浏览器不允许"有声音的自动播放"，会直接拦掉。所以自动播放必须配 `muted`。

手机上还要加 `playsinline`，否则 iOS 会强行全屏播放。

`playsinline` 是 Safari 的历史包袱，没有它，iPhone 上点视频就弹成全屏。

## audio：放音频

```html
<audio controls>
  <source src="song.mp3" type="audio/mpeg">
  <source src="song.ogg" type="audio/ogg">
  你的浏览器不支持 audio 标签。
</audio>
```

属性和 `<video>` 基本一样，没有 `poster` 和 `playsinline`。

音频在页面上只显示一条控制条。如果连控制条都不想要（做背景音乐），去掉 `controls` 加 `autoplay loop`——但用户没法关掉，体验很差，慎用。

## 直接嵌别人的视频

自己托管视频很费流量，通常直接用平台给的嵌入代码：

```html
<iframe
  src="//player.bilibili.com/player.html?bvid=BV1xx411c7mD"
  style="width:100%;aspect-ratio:16/9;border:0"
  title="B 站视频"
  allowfullscreen>
</iframe>
```

B 站的分享面板里有"嵌入代码"，复制过来就能用。

## 过时的东西

`<embed>` 和 `<object>` 是早期用来插 Flash 的，Flash 已经在 2020 年底被所有浏览器停止支持，这两个标签现在基本用不上。

看到就得知道它是什么，但新页面不要写。

延伸阅读：[HTML 多媒体 - 菜鸟教程](https://www.runoob.com/html/html-media.html)，[HTML5 视频](https://www.runoob.com/html/html5-video.html) 和 [HTML5 音频](https://www.runoob.com/html/html5-audio.html)，[MDN 的 video 元素](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/video)。

---

> 系列第 12 / 13 篇 · ← 上一步：[[11-head 里放什么，CSS 和 JS 从哪进来|head 里放什么]] · 下一步：[[13-页面打开不对，先查这几种|页面打开不对先查这几种]] → · 路线图：[[HTML 总览]]
