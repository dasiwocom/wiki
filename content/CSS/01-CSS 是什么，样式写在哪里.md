---
title:
aliases:
tags:
  - css
description: CSS 的规则长什么样、注释怎么写、三种引入方式分别放在哪，以及为什么推荐用外部样式文件。
---

# CSS 是什么，样式写在哪里

HTML 决定了页面上有什么，CSS 决定这些东西长什么样。

同一段 HTML，加几行 CSS 就能从"白底黑字的文档"变成一张像样的页面。

## 一条规则长什么样

```css
p {
  color: red;
  text-align: center;
}
```

拆开看：

```text
选择器        声明块
  ↓            ↓
  p     {  color: red;  text-align: center;  }
              ↑        ↑
            属性     值（一对声明）
```

- **选择器**：指哪一批元素要被改
- **声明**：一条"改什么、改成什么"
- 每条声明是 `属性: 值` 的形式，**末尾的分号不能漏**
- 多条声明用分号隔开，整块用一对大括号包起来

最后一条声明的分号可以省略，但建议每次都写——将来往下加一行就不用回头补。

## 分号和冒号写错是新手最常见的错

```css
/* 错：句尾用了中文分号 */
p { color: red； }

/* 错：属性和值之间用了等号 */
p { color = red; }

/* 对 */
p { color: red; }
```

CSS 用的是英文半角符号。中文输入法没切回来，就会出现这种"看着一样但完全不生效"的情况。

## 让它好读

属性少的时候一行写完没问题，多的时候一行一个属性：

```css
.card {
  width: 300px;
  padding: 16px;
  border: 1px solid #ddd;
  border-radius: 8px;
  background: #fff;
}
```

## 注释

```css
/* 这是注释，浏览器会忽略 */
p {
  color: red; /* 注释也可以写在声明后面 */
}
```

注意 CSS 里没有 `//` 注释，只有 `/* */`。写成 `//` 会让后面一整行失效。

## 样式写在哪里：三种方式

**① 外部样式文件**（推荐）

单独建一个 `style.css`，在 HTML 的 `<head>` 里引进来：

```html
<link rel="stylesheet" href="style.css">
```

**② 写在 head 里**

```html
<style>
  p { color: red; }
</style>
```

**③ 写在标签的 style 属性上**

```html
<p style="color: red;">红色文字</p>
```

↓ 三种方式最终的效果都一样：

<p style="color: #c0392b;">这段文字是红色的</p>

## 三种方式的区别

| | 写在哪 | 能复用吗 | 改起来 |
| --- | --- | --- | --- |
| 外部文件 | 单独的 .css | 多个页面共用 | 改一处，全站生效 |
| head 里的 style | 当前页面 | 只有当前页 | 每个页面都要改 |
| 标签的 style 属性 | 单个标签 | 不能复用 | 要满页面搜着改 |

一个网站有十个页面，改一次主色调：用外部文件改一行，用行内样式要改几百处。

所以只有临时调试、或者邮件里那种必须自带样式的场景才用行内样式。

## 优先级的顺序

同一条属性被多处写了，谁赢：

```text
标签的 style 属性   >   head 里的 style   >   外部文件
```

越"贴身"的优先级越高。所以行内样式最难覆盖，写多了以后别人接手会很难受。

## 一个完整的例子

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>样式示例</title>
  <style>
    body {
      font-family: system-ui, sans-serif;
      line-height: 1.7;
      max-width: 40em;
      margin: 2rem auto;
      padding: 0 1rem;
    }
    h1 { color: #284b63; }
    .note {
      padding: 12px 16px;
      border-left: 4px solid #84a59d;
      background: #f4f7f6;
    }
  </style>
</head>
<body>
  <h1>标题</h1>
  <p>正文段落。</p>
  <p class="note">一个带左侧色条的说明块。</p>
</body>
</html>
```

十几行 CSS，页面就从"默认样子"变成了能看的样子。

## CSS 和 CSS3 是什么关系

CSS3 不是另一个东西，它是 CSS 的第三个大版本，把语言拆成了很多"模块"各自升级。

现在说"CSS3 的圆角""CSS3 的动画"，指的是这十几年陆续加进来的新特性。写的时候还是写 `border-radius`，不用管它是哪个版本的。

延伸阅读：[CSS 教程 - 菜鸟教程](https://www.runoob.com/css/css-tutorial.html)，[CSS 语法](https://www.runoob.com/css/css-syntax.html)，[MDN 的 CSS 入门](https://developer.mozilla.org/zh-CN/docs/Web/CSS)。

---

> 系列第 1 / 14 篇 · 下一步：[[02-选择器，怎么精确指到要改的那个元素|选择器]] → · 路线图：[[00-CSS 总览]]
