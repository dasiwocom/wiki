---
title: "DOM 遍历起点错误：循环把 ul 当成 li"
date: 2026-09-08
authors: [workbuddy]
tags: []
draft: false
---

> 原文：[[DOM-Traversal-Starting-Point-When-Your-Loop-Treats-a-ul-as-an-li]]

# DOM 遍历的起点：当你的循环把 `<ul>` 当成 `<li>`

## 现象

一个可折叠侧边栏菜单由 Markdown 嵌套列表渲染而成。为了支持「强制展开这些目录」，有个函数会遍历 DOM，给每个目录 `<li>` 打上相对路径（`li.dataset.path`），这样折叠逻辑就能拿目录去和白名单比对。

白名单加载是对的（`FRONT_EXPANDED_DIRS = ["draft"]`），折叠逻辑也确实在拿 `li.dataset.path` 比对它 —— 但**所有目录还是全折叠了**，调试日志里每个目录的 `path=` 都是空的。白名单永远匹配不上。

## 原理

遍历函数被调用时传入的是**容器 `<div>`**，而不是里面的 `<ul>`：

```js
function buildDirPaths(root) {
    function walkUl(ul, prefix) {
        var items = ul.children;          // 以为拿到的是 <li>
        for (var i = 0; i < items.length; i++) {
            var li = items[i];            // 但如果 ul 其实是那个 <div>，items 就是一堆 <ul>！
            li.dataset.path = ...;        // 打到了错误的元素上
        }
    }
    walkUl(root, '');                     // root = 抽屉的 <div>，不是 <ul>
}
```

抽屉容器的结构是 `div > ul > li > ul > li ...`。从 **div** 开始遍历，`ul.children` 取到的是**内层的 `<ul>`** —— 循环把 `<ul>` 当成了 `<li>`，把 `dataset.path` 打在了 `<ul>` 上（以及嵌套的 `<ul>` 上），而**真正的 `<li>` 一个都没拿到路径**。下游代码去读真实 `<li>` 上的 `li.dataset.path` → 永远是空 → 白名单永远匹配不上 → 全部折叠。

全程不报错。错误的元素静默拿到了属性，正确的元素一直空着。这正是这个 bug 能活过 code review 的原因：遍历函数「看起来是工作的」（它确实给*某些东西*写了路径）。

## 如何复现

```html
<div id="drawer">          <!-- root 传的是这里 -->
  <ul>
    <li>draft<ul><li><a>#</a></li></ul></li>
  </ul>
</div>
```

```js
function walk(ul, prefix) {
  for (const el of ul.children) {          // div.children = [ul]  → el 是 <ul>，不是 <li>
    el.dataset.path = 'WRONG';             // 打在了 <ul> 上
    const sub = [...el.children].find(c => c.tagName === 'UL');
    if (sub) walk(sub, prefix);            // 递归也在错的一层上走
  }
}
walk(document.getElementById('drawer'), '');   // <li> 永远拿不到 data-path
```

## 解决方案

从容器里**第一个真正的 `<ul>`** 开始遍历：

```js
function buildDirPaths(root) {
    function walkUl(ul, prefix) { /* ...遍历 <li> 子元素，递归进 <ul> 子元素... */ }
    var rootUl = root.querySelector('ul');   // 先往下走一层
    if (rootUl) walkUl(rootUl, '');
}
```

通用规则：**容器元素和列表元素是不同的节点** —— `div.children` 是 `<ul>`，`ul.children` 才是 `<li>`。开始迭代之前，先钉死你的循环期望的到底是哪一种元素类型。

## 调试手法

循环里加一行日志，整个故事立刻就清楚了：

```
[drawer] path= dirs=["draft"] forceExpand=false
```

`dirs` 是对的，比较逻辑也是对的，那么剩下的唯一变量就是 `li.dataset.path` —— 而它对所有节点都是空的。空路径 + 正确的白名单 = 属性从来没被写到被读取的那些元素上。然后去 DevTools（Elements 面板）看 `data-path` 到底落在**哪些**元素身上：看到它挂在 `<ul>` 上（而不是 `<li>` 上），就确认了遍历错了一层。

## 小结

- DOM 遍历要先确认**起始节点的类型**：div → ul → li；起点高了一层就会静默地把属性打在错误元素上。
- 白名单匹配总是失败时，把参与比较的值都打出来（`path`、白名单）—— 空的那一侧就指出了属性被（没被）写在哪里。
- 在 DevTools 里先确认属性到底落在哪儿，再怀疑循环体写错了。
