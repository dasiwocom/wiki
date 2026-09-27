## Symptom

刷新站点时（尤其偶尔/频繁刷新），页面先闪一下英文（比如导航栏 "ToolBox"、"Home"），随后才变成中文（"工具箱"、"首页"）。同一个站点里**主题**从不闪（黑/白天立即正确），唯独**语言**会闪。

## Principle

### 为什么主题不闪、语言闪

首帧防闪由 `<head>` 里的内联脚本负责：

```html
<script>
(function(){try{
  var t=localStorage.getItem("toolbox-theme");
  var l=localStorage.getItem("toolbox-lang");
  var d=document.documentElement;
  if(t==="dark"){d.setAttribute("data-theme","dark")}
  if(l==="en"||l==="zh"){d.setAttribute("lang",l)}
}catch(e){}})();
</script>
```

- **主题**：它改的是 `<html data-theme>` 这一个属性，CSS 读取该属性后立即生效。属性在 head 里就设好了，所以只有"属性闪烁"的可能性，而这里没有。
- **语言**：它只设了 `<html lang>` 属性，但**文本内容**是 HTML 里写死的英文（`<span data-i18n="siteName">ToolBox</span>`）。把文本翻译成中文的代码在 `core.js` 里，而原先是把 `<script src="core.js">` 放在 `</body>` 最后——解析完整页、首帧贴出英文之后才执行翻译 → **英文一闪 → 变中文**。只有存了 `toolbox-lang=zh` 的用户能看到这个闪（默认 en 场景静态文本就是英文，翻译逻辑不改它，所以不闪）。

这其实是"语言版 FOUC"：主题闪是 `data-theme` 属性晚设置，语言闪是文本晚替换。

### 关键约束

任何"翻译脚本"都依赖 `<body>` 里的元素已经存在才能改 `textContent`。head 的内联脚本执行时 body 尚未解析，`querySelectorAll("[data-i18n]")` 拿不到任何东西。所以语言修复必须「**脚本先行 + DOM 就绪后执行**」。

## Solutions

### Option 1（本项目采用）：core.js 提前到 `<head>` 同步加载，init 挂 DOMContentLoaded

1. 把 `<script src="core.js?v=..."></script>` 从 `</body>` 移到 `</head>`（同步、不加 defer/async）。
2. 把 core.js 里原本「直接执行的 DOM 初始化」整体包进 `boot()`，末尾按就绪状态判断：

```js
if (document.readyState === "loading") {
    document.addEventListener("DOMContentLoaded", boot);
} else {
    boot();   // 脚本仍在 body 底部加载时的兼容路径
}
```

`boot()` 里再做 `applyTheme(); applyLang();` 以及所有 `getElementById(...).addEventListener`、抽屉绑定、build-badge 注入。

效果：`core.js` 在 body 解析前就已下载执行，首帧渲染（FCP）前 DOMContentLoaded 触发 `applyLang()`，第一帧就是正确语言。

**注意/坑（本项目实测验证过）：**
- 各页面 `app.js` 保持在 `</body>` 底部（同步）执行。它执行时 core.js 已加载完，`t()`/`showToast()`/`copyText()` 全局可用；`app.js` 里 `window.onLangChange = ()=>{}` 在 DCL 之前赋值，DCL 时 `boot()` 能读到——顺序不会冲突。
- `lang` / `theme` 是 core.js 顶层 `let` 变量，head 同步执行后即就绪，闭包引用正常。
- **FCP 与 DCL 理论上存在竞态**：多数浏览器首帧在 DCL 之后，大多数情况已足够；极端慢网络 + 极小内容下可能有一次性闪烁，可进一步在 body 首行放同页内联翻译（字典需内联，代价大），通常不需要。
- localStorage 读取要 `try/catch`（隐私模式下会抛异常）。

### DCL 方案不彻底的现实

headless/本地测出 `DOMContentLoaded(≤13ms) < FCP(≥24ms)`（翻译先于首帧），但**真机浏览器（尤其 Firefox）允许在 DCL 前就渲染 body 前部的静态英文文本**。headless 测不出这个时序，用户点进工具页仍偶发闪屏（线上排查：HTML/core 已是最新版、md5 全等，证明问题不在部署）。

### Option 1b（加固：首帧反 FOUC 保护层，本项目最终采用）

不依赖时序，用「语言确定前不显示可翻译文本」：

```css
html.i18n-pending [data-i18n],
html.i18n-pending .tool-head h1,
html.i18n-pending .tool-head p { visibility: hidden; }
```

head 内联脚本（继续放在 Option 1 之前执行的 head 里）：

```js
if (l !== "en") { d.classList.add("i18n-pending"); }
setTimeout(function(){try{document.documentElement.classList.remove("i18n-pending")}catch(e){}}, 1500);
```

`boot()` 在 `applyLang()` 后移除 class：

```js
document.documentElement.classList.remove("i18n-pending");
```

要点：
- 当 `lang===zh`（或未存值=默认 zh）：静态英文已存在于 HTML，但 `visibility:hidden` 让它们在翻译完成前不可见 → 即便浏览器抢先画帧，露出的也只是**空白占位**，绝不露出英文。翻译完成后一次性以正确语言呈现。
- 兜底超时 1.5s：防止 JS 加载失败导致文字永久隐藏（此时退化为"文字晚出现"而非全站空白）。`visibility:hidden` 保留布局，无跳动。
- 保护面要覆盖 `[data-i18n]`（导航/页脚/按钮/hero）+ `.tool-head h1/p`（title/desc 由 JS 填充、无 data-i18n 属性）两处。

### Option 2（兜底或不想动结构时）：body 任意处放 blocked inline

页面较短时，在 `<body>` 之后、`<header>` 之前放同步内联脚本，也可在首帧前完成翻译（header 尚未解析、必然未渲染）。缺点：需要把翻译字典内联进每页，维护成本高。项目最终没采用。

## Summary

- 主题闪 = `data-theme` 属性没提前设；语言闪 = 文本翻译代码执行太晚（body 底部）。同一个 `<head>` 内联脚本只解决了前者。
- 翻译必须等 DOM，所以标准姿势是「core.js 放 head 同步 + DOMContentLoaded 后 boot」，而不是指望头部脚本提前翻译。
- **不要相信 headless 的 FCP/DCL 时序**：真机（Firefox 尤其）可先画帧。要彻底消除语言 FOUC，用 `i18n-pending` 保护层（文本就绪前 `visibility:hidden`）兜住任意时序。
- readyState 分支让同一份代码既能放 head 也能放 body，方便回退。
- 验证方法：headless Chromium + CDP 设 `toolbox-lang`，检查 `Runtime.exceptionThrown` 为空、`.tool-head h1` 首屏即为目标语言、且 `i18n-pending` 在 boot 后被移除。
- 相关笔记：`Causes-of-Page-Flicker-in-LightDark-Mode-Toggle-and-UnifiedTransition-Solution.md`、`Preventing-Night-Mode-Flash-on-Refresh-in-Firefox.md`（同属 FOUC 家族，只是对象不同）。