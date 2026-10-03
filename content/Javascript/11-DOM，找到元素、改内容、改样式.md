---
title:
aliases:
tags:
  - javascript
description: querySelector 怎么找元素、textContent 和 innerHTML 的区别、classList 改样式、创建和删除元素，以及 DOM 到底是什么。
---

# DOM，找到元素、改内容、改样式

前面十章都是纯 JavaScript。从这一篇开始，代码才真正碰到页面。

## DOM 是什么

浏览器读到 HTML 之后，会把每个标签变成一个"对象"，按嵌套关系连成一棵树。这棵树就是 DOM。

```text
document
└── html
    ├── head
    │   └── title
    └── body
        ├── h1
        └── p
```

JavaScript 通过 `document` 这个入口，在这棵树上找东西、改东西。

页面上任何变化，都是"找到某个节点，改它的属性"。

## 找元素

```javascript
// 按 id 找（最快，返回一个元素）
document.getElementById("header");

// 按 CSS 选择器找第一个
document.querySelector(".card");
document.querySelector("#header");
document.querySelector("nav ul li");

// 按 CSS 选择器找全部（返回类数组）
document.querySelectorAll(".card");
```

`querySelector` 里写的东西**和 CSS 选择器完全一样**——类、id、后代、属性、伪类都能用。

所以 CSS 学得熟，找元素这件事就白送。

```javascript
document.querySelector('input[type="email"]');
document.querySelector(".list li:first-child");
```

`querySelectorAll` 返回的不是真正的数组（是 NodeList），但可以遍历，也能用 `forEach`：

```javascript
document.querySelectorAll(".card").forEach(el => {
  el.classList.add("loaded");
});
```

真要当数组用（比如 `map`），先转换：

```javascript
[...document.querySelectorAll(".card")].map(el => el.textContent);
```

## 改文字

```javascript
const el = document.querySelector("#msg");

el.textContent = "新的文字";
```

`textContent` 是最安全的写法，它会**原样当作文字**，里面的标签不会被解析。

```javascript
el.textContent = "<b>加粗</b>";
// 页面上显示的是字面的 <b>加粗</b>
```

`innerHTML` 会把字符串当 HTML 解析：

```javascript
el.innerHTML = "<b>加粗</b>";
// 页面上是加粗的两个字
```

方便，但**用之前必须想清楚内容是谁给的**。

如果内容来自用户输入或网址参数，直接用 `innerHTML` 就等于把别人写的 `<script>` 放进自己页面，这是最典型的 XSS 攻击。

```javascript
// 危险：用户输入直接塞进去
el.innerHTML = userInput;

// 安全
el.textContent = userInput;
```

需要拼接多个元素时，要么用 `textContent` 一块块拼，要么用后面的 `createElement` 创建真元素。

## 读文字

```javascript
el.textContent;      // 里面所有的文字（包括隐藏的）
el.innerText;        // 只有可见的文字，会触发重排，慢
```

取值一般用 `textContent`。

输入框取值是另一回事，用 `value`：

```javascript
document.querySelector("#name").value;
```

## 改样式

**直接改单个属性**：

```javascript
el.style.color = "#c0392b";
el.style.fontSize = "20px";   // 注意：带横线的属性名要写成小驼峰
```

CSS 里的 `font-size`、`background-color`，在 JS 里要写成 `fontSize`、`backgroundColor`。

**更好的办法：切换类名**。

把样式写在 CSS 里，JS 只负责加类、删类：

```css
.card { opacity: 0.5; }
.card.active { opacity: 1; box-shadow: 0 4px 12px rgba(0,0,0,.1); }
```

```javascript
el.classList.add("active");       // 加
el.classList.remove("active");    // 删
el.classList.toggle("active");    // 有就删、没有就加
el.classList.contains("active");  // 判断有没有，返回布尔
```

一串 `el.style.xxx = ...` 会把样式散在 JS 里，改一次要满项目找。用类名，样式还是集中在 CSS 文件里。

## 读写属性

```javascript
// 常规属性
img.src = "new.jpg";
a.href = "/about";
input.disabled = true;

// 自定义属性，用 dataset
// HTML: <div data-id="42" data-user-name="张三">
el.dataset.id;        // "42"
el.dataset.userName;  // "张三"  ← 连字符转成小驼峰

el.getAttribute("data-id");   // 另一种读法
el.setAttribute("data-id", "43");
```

`dataset` 是"把数据挂在元素上"的标准做法。列表渲染时经常用它记住这一项对应的 id：

```html
<li data-id="1">张三</li>
```

```javascript
li.addEventListener("click", e => {
  console.log(e.currentTarget.dataset.id);   // "1"
});
```

## 创建、插入、删除

```javascript
// 创建
const li = document.createElement("li");
li.textContent = "新的一项";
li.className = "item";

// 插入到末尾
list.appendChild(li);

// 插到某个元素前面
list.prepend(li);                          // 插到第一个
list.insertBefore(li, list.firstChild);    // 插到指定元素前

// 用 HTML 字符串一次插入
list.insertAdjacentHTML("beforeend", "<li>新的一项</li>");

// 删除自己
li.remove();

// 清空一个容器
list.replaceChildren();   // 现代写法
list.innerHTML = "";      // 也行，但会清掉所有子节点和事件
```

## 一个完整的例子

```html
<ul id="list"></ul>
<button id="add">加一项</button>

<script>
  const list = document.querySelector("#list");
  const addBtn = document.querySelector("#add");
  let count = 0;

  addBtn.addEventListener("click", () => {
    count++;
    const li = document.createElement("li");
    li.textContent = `第 ${count} 项`;
    list.appendChild(li);
  });
</script>
```

点按钮，列表就长一行。这就是"用 JS 生成页面内容"的最小例子。

## 一个性能提醒

**别在循环里反复操作 DOM。**

```javascript
// 慢：每次都改一次页面
for (const item of list) {
  container.appendChild(makeNode(item));
}

// 快：先在内存里拼好，只插入一次
const frag = document.createDocumentFragment();
for (const item of list) {
  frag.appendChild(makeNode(item));
}
container.appendChild(frag);
```

每次改 DOM 都可能触发浏览器的重新排版。一百条数据分开插，就是一百次重排。

数据多的时候用 `DocumentFragment` 或者先拼成字符串再一次性 `innerHTML`。

## 页面元素什么时候能拿到

```html
<script>
  document.querySelector("#btn");   // null，因为下面的 button 还没解析到
</script>
<button id="btn">点我</button>
```

脚本执行的顺序就是 HTML 出现的顺序。脚本写在元素前面，找不到它。

两个解决办法：

**① 把 `<script>` 放在 `</body>` 之前**（见 [[11-head 里放什么，CSS 和 JS 从哪进来|HTML 那篇]]）

**② 加 `defer`**，让脚本等页面解析完再跑

```html
<script src="app.js" defer></script>
```

## 找元素没找到的排查

返回 `null` 时，一定是在 `null` 上调方法报的错：

```text
Uncaught TypeError: Cannot read properties of null (reading 'addEventListener')
```

依次查：

1. 选择器写对了吗？在 F12 的 Console 里直接敲 `document.querySelector("你的选择器")` 试一下
2. 元素是脚本之后才出现的吗？改 `defer` 或者调整位置
3. 元素是动态生成的（比如点按钮才出来）吗？那就得等它出来再找

延伸阅读：[DOM 简介 - 菜鸟教程](https://www.runoob.com/js/js-htmldom.html)，[DOM 元素](https://www.runoob.com/js/js-htmldom-elements.html)，[DOM HTML](https://www.runoob.com/js/js-htmldom-html.html)，[MDN 的 DOM 简介](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model/Introduction)。

---

> 系列第 11 / 15 篇 · ← 上一步：[[10-字符串和数字的常用方法|字符串和数字]] · 下一步：[[12-事件，用户做了什么就响应什么|事件]] → · 路线图：[[00-JavaScript 总览]]
