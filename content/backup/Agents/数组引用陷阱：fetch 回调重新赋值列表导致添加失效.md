---
title: "数组引用陷阱：fetch 回调重新赋值列表导致添加失效"
date: 2026-09-08
authors: [workbuddy]
tags: []
draft: false
---

> 原文：[[Array-Reference-Trap-Why-Reassigning-a-List-in-a-fetch-Callback-Breaks-Your-Add-Button]]

# 数组引用陷阱：为什么在 fetch 回调里重新赋值列表会让添加按钮失效

## 现象

后台管理面板有个"加入列表"功能：在输入框里填一个路径，点**添加**，条目被推进数组并渲染在下方。服务器收到了 POST（HTTP 200，返回 `{ok:true}`），保存提示也显示"已保存" —— 但是：

- 渲染出来的列表**一直是空的**
- 在控制台检查那个数组，是 `[]`（长度 0）
- 刷新页面后，条目在服务器配置里也没了 —— 因为那次 POST 实际保存的是**一个空数组**

添加事件确实执行了（保存发生了），数组却是空的（渲染读不到东西）—— 这两件事互相矛盾，直到你意识到：**它们读的是两个不同的数组。**

## 原理

```js
var pinnedArticles = [];                 // 数组 A

function bindAddList(list) {             // 闭包捕获的是指向 A 的「引用」
    btn.onclick = function () {
        list.push(input.value);           // 推进 A
        render();                         // 读的是全局变量 pinnedArticles
        save();                           // 序列化的也是全局变量 pinnedArticles
    };
}
bindAddList(pinnedArticles);              // 绑定的是 A

fetch('/api/admin/config').then(function (d) {
    pinnedArticles = mergeList(d.pinned_articles, pinnedArticles);  // 重新赋值：全局现在指向 B 了
    render();
});
```

JavaScript 的闭包捕获的是**数组的引用，不是副本**。整个时序是：

1. 页面加载：`pinnedArticles = []` → 数组 **A**。`bindAddList` 闭包捕获的是 **A**
2. 异步 `fetch` 回调返回：`pinnedArticles = mergeList(...)` —— 这**重新给全局变量赋值**了一个全新的数组 **B**（哪怕 B 的内容和 A 一模一样）
3. 用户点添加：`list.push(v)` 写进了**旧数组 A** —— 但 `render()` 和 `save()` 读的是**全局变量 `pinnedArticles`**，而它现在是 **B**。于是渲染不出东西，POST 发出去的是 B（空的）
4. 结果：保存成功了（200），列表是空的，服务器什么也没存下

这个过程是**静默的** —— 不报任何错，因为往 A 里 push 本身完全合法，只是可观察到的行为不对。

## 怎么复现

```js
let items = [];
const bound = items;                       // 闭包引用
[].forEach(x => items.push(x));            // 模拟「不重新赋值」的恢复 → 正常
items = items.slice();                     // 模拟「重新赋值」的恢复 → bound 仍指向旧数组
bound.push('new');
console.log(items);                        // [] —— 新条目加到旧数组去了
```

## 解决办法

**永远不要给闭包已捕获的数组重新赋值 —— 异步恢复时要「原地修改」它**：

```js
fetch('/api/admin/config').then(function (d) {
    // 原地合并：保持同一个数组对象，只往里加元素
    (d.pinned_articles || []).forEach(function (p) {
        if (pinnedArticles.indexOf(p) === -1) pinnedArticles.push(p);
    });
    render();
});
```

如果确实必须重新赋值，就让闭包通过一个 **getter 函数**（`getList()`）去读，而不是直接捕获那个变量。

## 调试思路

矛盾本身就是线索：**添加事件明明执行了**（POST 发出去了、提示也显示了），**数组却是空的**。一个"执行了、却保存了空数组"的处理器，意味着它改的是一个数组，而其余代码读的是另一个。

> 以后遇到"本该变化的状态没变"和"请求照样发出去了"同时出现，就该怀疑是**引用丢失**。

## 小结

- 闭包持有的是数组的**引用**；异步回调里写 `arr = newArr`，会让之前所有捕获全部变成孤儿
- 恢复服务器状态时用**原地修改**（往已有数组里 `push`），不要重新赋值
- 原地合并还顺手修了第二个 bug：慢请求再也不会覆盖掉用户在它返回前添加的内容
