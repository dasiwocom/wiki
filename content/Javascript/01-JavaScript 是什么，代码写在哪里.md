---
title:
aliases:
tags:
  - javascript
description: JavaScript 能干什么、代码写在哪里、怎么在没有页面的情况下直接试代码，以及注释和分号的规矩。
---

# JavaScript 是什么，代码写在哪里

HTML 和 CSS 都是"描述"：描述有什么、描述长什么样。JavaScript 不一样，它会**算、会判断、会改**。

## 三者的分工

| 语言 | 管什么 |
| --- | --- |
| HTML | 页面上有什么（标题、段落、按钮） |
| CSS | 这些东西长什么样（颜色、位置） |
| JavaScript | 页面能做什么（点了会怎样、数据怎么变） |

一个按钮放在页面上，是 HTML 的事；按钮什么颜色，是 CSS 的事；点下去弹出提示、把内容改掉、算出总价，是 JavaScript 的事。

## 它还能干别的

在没有浏览器的地方，JavaScript 也能跑：

- 服务器程序（Node.js）
- 桌面应用（VS Code、Discord 都是）
- 手机 App（React Native）

但它最经典的战场还是网页，这一组笔记也只讲网页里的用法。

## 代码写在哪里

**① 写在 HTML 里**

```html
<script>
  alert("你好");
</script>
```

**② 单独的文件**

```html
<script src="app.js" defer></script>
```

JavaScript 单独放一个 `.js` 文件，是正常项目的做法。`defer` 让它等页面解析完再执行。

**③ 直接写在事件属性上**（不推荐）

```html
<button onclick="alert('你好')">点我</button>
```

能跑，但逻辑混在 HTML 里，改起来很难受。只在最简单地试一下的时候用。

## 更快的办法：直接开控制台

想试一小段代码，不用建文件。

浏览器里按 `F12`，切到 **Console（控制台）** 标签，光标那里直接敲：

```javascript
1 + 1
```

回车，下面立刻显示 `2`。

再试：

```javascript
"你好" + "世界"
Math.max(3, 7, 2)
[1, 2, 3].map(n => n * 2)
```

控制台里敲的每一行都会立刻执行、显示结果。学 JavaScript 的时候，这一半的时间都应该在这里过。

## 三种输出方式

```javascript
// 输出到控制台，调试用
console.log("你好");

// 弹出提示框
alert("你好");

// 写进页面（只在页面还没加载完时好用，现在基本不用）
document.write("<h1>标题</h1>");
```

**`console.log` 是你最常用的一个。** 想知道某个变量到底是多少，打印出来看一眼，比盯着代码猜快十倍。

`alert` 会打断用户操作，正式代码里不要用它做输出。调试的时候偶尔用一下还行。

在控制台里，不写 `console.log` 也可以——直接敲变量名回车就会显示它的值。但在代码里必须写。

## 注释

```javascript
// 单行注释

/*
  多行注释
  可以写好几行
*/
```

和 CSS 不一样，JavaScript 用的是 `//` 和 `/* */`。

## 分号

```javascript
let a = 1;
let b = 2;
```

每条语句末尾写分号。

严格来说，JavaScript 会自动在换行处补分号（叫 ASI），有时候不写也能跑。但这条规则有几个著名陷阱：

```javascript
// 你以为是这样
let x = 1
[1, 2].forEach(...)

// 实际被理解成这样，报错
let x = 1[1, 2].forEach(...)
```

所以**每句老老实实写分号**，能省掉一整类莫名其妙的 bug。

## 大小写敏感

```javascript
let name = "a";
console.log(Name);   // 报错：Name is not defined
```

`name` 和 `Name` 是两个不同的东西。

函数和方法也一样，`getElementById` 写成 `getElementByID` 就找不到。

敲代码的时候注意大写锁定键。

## 一个能跑的小例子

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>试试 JS</title>
</head>
<body>
  <p id="out">还没点</p>
  <button id="btn">点我</button>

  <script>
    const btn = document.getElementById("btn");
    const out = document.getElementById("out");

    btn.addEventListener("click", () => {
      out.textContent = "现在是 " + new Date().toLocaleTimeString();
    });
  </script>
</body>
</html>
```

页面上多了一个按钮，点一下，上面那行文字就变成一个时间。

`getElementById` 是"按 id 找元素"，`addEventListener` 是"监听点击"，`textContent` 是"改文字"。这三件事会贯穿整个 JavaScript 入门。

## 名字里的两个误会

**JavaScript 和 Java 没有关系。**

名字里带 Java，纯粹是当年为了蹭热度。两者语法像是因为都借鉴了 C 风格，设计上完全是两回事。

**ECMAScript 和 JavaScript 是什么关系？**

JavaScript 由 Netscape 发明，后来交给 ECMA 标准化，标准的名字叫 ECMAScript。所以"ES6""ES2015"指的就是这个标准的新版本。

日常说话两个词混用，没人会纠正你。

延伸阅读：[JavaScript 简介 - 菜鸟教程](https://www.runoob.com/js/js-intro.html)，[JavaScript 用法](https://www.runoob.com/js/js-howto.html)，[MDN 的 JavaScript 指南](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide)。

---

> 系列第 1 / 15 篇 · 下一步：[[02-变量和数据类型|变量和数据类型]] → · 路线图：[[00-JavaScript 总览]]
