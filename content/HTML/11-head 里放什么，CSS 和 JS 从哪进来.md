---
title:
aliases:
tags:
  - html
description: head 里的 title、meta、link、script 各写什么，CSS 和 JS 的三种引入方式，以及 script 放头部还是放尾部、defer 和 async 的区别。
---

# head 里放什么，CSS 和 JS 从哪进来

`<head>` 里的东西永远不显示，但少写一条，页面就出问题。

## 一个完整的 head

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="这里是用一两句话描述这个页面">
  <title>页面标题 - 站点名</title>
  <link rel="icon" href="/favicon.ico">
  <link rel="stylesheet" href="style.css">
</head>
```

逐条说。

## charset 必须放在最前面

```html
<meta charset="UTF-8">
```

浏览器是从上往下读文件的，如果它读到一半才知道编码是 UTF-8，前面已经按错的编码解析过了，中文就乱。

规范要求它出现在 `<head>` 的前 1024 个字节以内，实际就是放第一条。

## viewport：手机上会不会变小

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

不写这一条，手机上打开网页会按桌面宽度（约 980px）渲染，然后整体缩到屏幕大小，字小得看不清。

写了之后，页面宽度就跟手机屏幕宽度一致，一切正常。

只要页面要在手机上看，这条就必须写。

## title

```html
<title>页面标题 - 站点名</title>
```

它出现在三个地方：浏览器标签页、收藏夹、搜索结果的标题。

格式一般是"这个页面的名字 - 站点的名字"，前面是这页特有的，后面是固定的站点名。

每个页面的 title 都应该不一样。

## description

```html
<meta name="description" content="一段一两句话的摘要。">
```

搜索引擎有时会拿它当搜索结果的摘要文字。

它不直接影响排名，但影响点击率——写清楚这页讲什么，比不写强。

## 其他常见的 meta

| 写法 | 作用 |
| --- | --- |
| `<meta name="author" content="名字">` | 作者 |
| `<meta name="keywords" content="关键词">` | 关键词，现在搜索引擎基本不看 |
| `<meta http-equiv="refresh" content="5;url=/next.html">` | 5 秒后自动跳到别的页，滥用会被当垃圾站 |
| `<meta property="og:title" content="标题">` | 分享到社交平台时显示的标题 |
| `<meta property="og:image" content="/cover.jpg">` | 分享时显示的封面图 |

`og:` 开头的是 Open Graph 协议，决定了别人把链接发到聊天软件里时，卡片上显示什么。

## 引入 CSS：三种方式

**外部文件**（推荐）：

```html
<link rel="stylesheet" href="style.css">
```

**写在 head 里**：

```html
<style>
  body { background: #fff; }
</style>
```

**写在标签的 style 属性上**（尽量别用）：

```html
<p style="color: red;">红色文字</p>
```

优先级是"越贴身越高"：style 属性 > head 里的 style > 外部文件。

所以行内样式最难覆盖，改起来要满页面搜，一旦写多了就没法维护。

## 引入 JS：两种方式

**外部文件**：

```html
<script src="app.js"></script>
```

**写在页面里**：

```html
<script>
  console.log("你好");
</script>
```

## script 放在哪

`<script>` 会**卡住页面渲染**：浏览器读到它就必须停下来，把里面的代码下载并执行完，才继续往下画。

所以放在 `<head>` 里会让页面白屏更久。

三种做法，优先选第一种：

**① 放在 body 的最后一行**：

```html
<body>
  ...所有内容...
  <script src="app.js"></script>
</body>
```

页面内容先显示出来，再执行脚本。

**② 加 defer**：

```html
<script src="app.js" defer></script>
```

放在 head 里也行，浏览器会并行下载，等文档解析完再按顺序执行。这是现在最推荐的方式。

**③ 加 async**：

```html
<script src="analytics.js" async></script>
```

下载完立刻执行，不管文档解没解析完。适合统计代码这类互不依赖的脚本。

`defer` 和 `async` 都只对外部文件生效，写在标签里的代码加了没用。

| | 下载 | 执行时机 | 多个脚本的顺序 |
| --- | --- | --- | --- |
| 什么都不加 | 阻塞 | 下载完立刻 | 按顺序 |
| `defer` | 并行 | 文档解析完 | 按顺序 |
| `async` | 并行 | 下载完立刻 | 不保证 |

## 还有个 base

```html
<base href="https://example.com/">
```

它会给页面里所有相对路径定一个基准，一般用不到，用错了会让整页的链接和图片全指向别的地方。

---

> 系列第 11 / 13 篇 · ← 上一步：[[10-把页面分区，div、span 和那些语义标签|把页面分区]] · 下一步：[[12-嵌入外部内容，iframe、视频和音频|嵌入外部内容]] → · 路线图：[[HTML 总览]]
