---
title:
aliases:
tags:
  - javascript
description: 全局、函数、块级作用域的区别，var 和 let 到底差在哪，this 在什么情况下指向谁。
---

# 作用域和 this，变量能被谁看见

变量不是写完就到处能用的。它在哪儿能被读到，就是作用域问题。

## 三种作用域

```javascript
let global = "全局";

function outer() {
  let funcScoped = "函数内";

  if (true) {
    let blockScoped = "块内";
    console.log(blockScoped);   // 能读到
    console.log(funcScoped);    // 能读到（外层）
    console.log(global);        // 能读到（最外层）
  }

  console.log(blockScoped);     // 报错：blockScoped is not defined
}
```

规则是**从内往外找**：

```text
全局作用域
  └── 函数作用域
        └── 块作用域（if、for 的花括号里）
```

里面的能看见外面的，外面的看不见里面的。

## var 和 let 的真正区别

```javascript
// var：不受花括号限制
if (true) {
  var a = 1;
}
console.log(a);   // 1  ← 跑到外面来了

// let：只在当前块里
if (true) {
  let b = 1;
}
console.log(b);   // 报错
```

`var` 的作用范围是**整个函数**，`let` 和 `const` 的作用范围是**最近的一对花括号**。

这个差别在循环里最要命：

```javascript
// 用 var，全打印 3
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}

// 用 let，打印 0 1 2
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

`var` 声明的 `i` 只有一个，循环结束时它已经是 3 了，三个回调看到的都是同一个 3。

`let` 每一轮都会生成一个新的 `i`，所以各是各的。

这就是"用 `let` 和 `const`，别用 `var`"的一个具体理由。

## 提升

```javascript
console.log(a);   // undefined，不报错
var a = 1;

console.log(b);   // 报错：Cannot access 'b' before initialization
let b = 1;
```

`var` 的声明会被"提升"到函数顶部，但赋值不会，所以读到的是 `undefined`。

`let` 和 `const` 提升之后处于"暂时性死区"，读到就报错。

**结论还是一样：别用 `var`。**

## 闭包

函数可以记住它出生时所在的那个作用域：

```javascript
function makeCounter() {
  let count = 0;

  return function () {
    count++;
    return count;
  };
}

const next = makeCounter();
next();   // 1
next();   // 2
next();   // 3
```

`makeCounter()` 早就执行完了，但里面的 `count` 没有消失——因为返回的那个函数还在用它。

这就是闭包：**函数和它记住的那些变量一起打包存活。**

最常见的用途是"造一个私有的计数器"，外面的代码没法直接改 `count`，只能通过 `next()` 去动它。

## this 指向谁

`this` 的值由**调用方式**决定，不由定义位置决定。这是最容易懵的一条。

简单记四条：

**① 直接调用函数，`this` 是 `undefined`**（严格模式）或全局对象（非严格）：

```javascript
function f() { console.log(this); }
f();   // undefined
```

**② 当作对象的方法调用，`this` 是那个对象**：

```javascript
const user = {
  name: "张三",
  hello() { console.log(this.name); }
};

user.hello();   // "张三"
```

**③ 用箭头函数，`this` 跟外面的作用域走**：

```javascript
const user = {
  name: "张三",
  hello: () => console.log(this.name)   // this 不是 user
};
```

这也是为什么**对象的方法不要用箭头函数**。

**④ 事件监听里，`this` 是触发事件的元素**：

```javascript
button.addEventListener("click", function () {
  console.log(this);   // button 这个元素
});

button.addEventListener("click", () => {
  console.log(this);   // 不是 button
});
```

## 一个常见的翻车现场

```javascript
const timer = {
  seconds: 0,

  start() {
    setInterval(function () {
      this.seconds++;      // this 不是 timer
      console.log(this.seconds);
    }, 1000);
  }
};
```

回调是"直接调用"的，所以里面的 `this` 不是 `timer`。

两种改法：

```javascript
// ① 用箭头函数，this 跟外面
setInterval(() => {
  this.seconds++;
}, 1000);

// ② 先把 this 存起来
const self = this;
setInterval(function () {
  self.seconds++;
}, 1000);
```

第一种更清爽，所以现在都用箭头函数。

## 实际写代码时的原则

- 变量**尽量声明在离使用最近的地方**，别一上来就在最顶上堆一堆 `let`
- **不要创建全局变量**。全局变量在哪个函数里都能被改，出 bug 时根本查不出是谁改的
- 一个文件里的东西如果只在文件内用，就别暴露出去
- 少用 `this`。用箭头函数和普通参数能绕开大部分 `this` 的困惑

延伸阅读：[JavaScript 作用域 - 菜鸟教程](https://www.runoob.com/js/js-scope.html)，[JavaScript this](https://www.runoob.com/js/js-this.html)，[JavaScript let 和 const](https://www.runoob.com/js/js-let-const.html)，[MDN 的闭包](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Closures)。

---

> 系列第 7 / 15 篇 · ← 上一步：[[06-函数，把一段代码打包起来|函数]] · 下一步：[[08-数组，以及几个好用的数组方法|数组]] → · 路线图：[[00-JavaScript 总览]]
