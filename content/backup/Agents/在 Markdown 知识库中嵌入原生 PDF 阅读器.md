---
title: "在 Markdown 知识库中嵌入原生 PDF 阅读器"
date: 2026-09-08
authors: [workbuddy]
tags: []
draft: false
---

> 原文：[[Embedding-a-Native-PDF-Reader-into-a-Markdown-Knowledge-Base]]

# 在 Markdown 知识库中嵌入原生 PDF 阅读器

## 现象

你跑着一个自托管的 Markdown 知识库（单 PHP 入口，`.md` 走 SSR，客户端是 SPA）。用户希望 PDF 书籍也能像文章一样**站内打开**：在目录树里点一个 PDF → 渲染出一个阅读器页面 —— 不下载，不跳转到浏览器的原生查看器。而且还要跟随站点的明暗主题。

这篇笔记记录了完整的集成过程，以及一路上踩到的每一个坑（路由、权限、canvas 渲染、移动端清晰度、全屏、主题切换闪烁）。

## 架构

```
/xxx.pdf (页面 URL，不带 /vault/ 前缀)
   → nginx: location ~* \.pdf$ { rewrite ^(.*)$ /index.php last; }   (SSR 出阅读器页面)
/vault/xxx.pdf (文件 URL)
   → nginx: location ^~ /vault/ { }                                    (静态原件，pdf.js 去 fetch 这个)
```

前端**按需懒加载** `pdf.js`（只有打开 PDF 时才加载），通过 `/vault/...` 取原件，然后**一次一页**渲染到 `<canvas>` 上。每一页都是从 PDF 的矢量指令画出来的位图 —— 不是 HTML 文字。翻页 = 重画 canvas；缩放 = 用新的 scale 重画（像素级精确，不是拉伸）。

## 坑 1：文件权限静默干掉整个站点

**现象**：用 patch 工具编辑完 `functions.php` 之后，每个页面都返回一张 200 的错误页，内容是 `Fatal error: require(.../functions.php): Failed to open stream: Permission denied`。

**原因**：patch 工具以 `root` 身份重写了文件，权限变成 `600`。PHP-FPM 以 `www` 身份运行，读不到。因为这个 fatal 发生在发送响应头之前，CDN 可能把错误页缓存下来 —— 导致修好之后站点看起来还是「没变化」，而且持续很久。

**修法**：任何服务端文件编辑之后，跑 `chown www:www file` 和 `chmod 644 file`，然后用 `curl` 验证（用新 URL 或者穿掉 CDN 缓存，比如把测试文件改个名）。

## 坑 2：大范围替换会把夹在中间的函数删掉

**现象**：Markdown 文章不渲染了。服务端返回 200 和合法的 HTML，但控制台报 `Cannot find variable: showArticle`。点目录树里任何一项都没反应。

**原因**：有个脚本替换了一大段代码（从 `// PDF reader` 到 `async function selectFile`），而被替换的范围**中间**正好夹着 `showArticle` 的函数定义。语法没问题，只是这个函数在运行时已经不存在了。

**修法**：任何大范围替换之后，grep 一下边界函数（`grep -c "function showArticle"`）。优先用锚点唯一的小补丁。把最终的 JS 抽出来跑 `node --check`（先剥掉 `<?php ... ?>` —— 替换成 `null;` —— 否则 PHP 的 echo 会让检查失败）。

## 坑 3：适应宽度不能过早读容器宽度

**现象**：初始缩放显示成一个 30% 的小缩略图，而不是撑满内容区；还出现了横向滚动。

**原因**：刚切换完 `display` 就去读 canvas 容器的 `clientWidth`，可能还是 0（布局还没算出来），于是兜底逻辑把 scale 压到了最小值。

**修法**：改去量一个稳定的元素 —— `document.querySelector('.doc-wrap').clientWidth`（和 Markdown 文章同宽），配一个窗口宽度的兜底。另外：**fit 不能有区间钳制**（0.3~1.5）。特别宽或特别小的 PDF 页面要撑满宽度，本来就需要落在这个区间之外的值；钳制只对手动缩放按钮有意义。

## 坑 4：一个自动纠正的循环把缩放按钮架空了

**现象**：`+` / `−` 按钮像是死的。

**原因**：有个「自动纠正」检查（如果 canvas 比容器宽 → 强制重新 fit），每次放大之后都会被再次触发，把 scale 又弹回 fit。用户根本看不到缩放效果。

**修法**：把自动纠正整个删掉。缩放按钮就是用新的 scale 重画页面，别的逻辑不许跟它抢。

## 坑 5：函数作用域 —— 渲染器引用了局部变量

**现象**：阅读正常，但缩放/翻页时抛 `ReferenceError: cw is not defined`。

**原因**：`renderPdfPage()`（模块作用域）引用了 `cw`，而 `cw` 是在 `openPdf()` 内部创建的局部变量。同样地，`fitScale` 定义在 `openPdf` 内部，却被 `renderPdfPage` 调用。

**修法**：把所有 PDF 状态和辅助函数都声明在模块级（`pdfDoc`、`pdfScale`、`fitScale`、`renderPdfPage`、`openPdf`），`openPdf` 里只保留 UI 绑定的闭包。

## 坑 6：手机上文字发虚 —— canvas 像素密度

**现象**：移动端文字模糊，桌面端还可以。

**原因**：canvas 的像素是逻辑像素；手机屏的设备像素比（DPR）是 2–3，所以每个 canvas 像素被拉伸到 2–3 个物理像素上。

**修法**：按 `scale × devicePixelRatio × 1.5` 渲染（超采样），然后把 canvas 的 CSS 尺寸设回逻辑尺寸（`pixelWidth / dpr`）。文字清晰，布局不变。代价：canvas 更大，翻页略慢。

## 坑 7：扫描版 PDF 不理透明背景

**现象**：文字版 PDF 能拿到跟随主题的背景（canvas 透明 → 页面背景透出来），但扫描版图书在反色之后还是黑的。

**原因**：扫描版 PDF 是整页图片 —— 白背景在**图片像素里面**，不是渲染器的背景。`page.render({ background: 'transparent' })` 帮不上忙。

**修法**：对像素做后处理 —— 渲染完之后遍历 `ImageData`，把接近白的像素（`r,g,b > 235`）的 alpha 设为 0。配合 CSS 的 `invert` 滤镜，深色文字变白，页面背景透出来。文字版 PDF 不受影响（它们的背景本来就是透明的）。

## 坑 8：主题切换闪烁 —— 完整链条

这场闪烁打了四个回合，每一回合的「修复」都导致了下一回合的症状。记录在此，方便你直接跳过：

### 第 1 回合：整个 PDF 框架在闪

加了 `#pdf-view, #pdf-view * { transition: none !important; }`（本意是「别让 canvas 过渡」），结果整个框架（背景、工具栏、边框）**瞬间跳变，而页面在 0.25s 内渐变**。这种不同步就是经典的 `transition: none !important` 陷阱（见主题过渡那篇笔记）。**把它删掉** —— 全局的 `*, *::before, *::after { transition: ... }` 规则必须也覆盖到 PDF 框架。

### 第 2 回合：canvas 区域在深色→浅色时仍然闪

canvas 是位图：它的像素是瞬间变的（没有 CSS 过渡可用），而页面背景在 0.25s 内渐变。具体来说，移除 `html.dark` 会让 `filter: invert` 立刻失效 —— 文字在背景渐变到一半的时候就从白跳到黑。

**修法 A（机制）**：缓存像素缓冲。每次渲染之后保存原始的 `ImageData`；在第一次切到深色时，生成一份处理过的副本（白→透明），只做一次。之后主题切换只需要 `putImageData`（快速拷贝，不用每次遍历像素，也没有中间帧）。

**修法 B（时序）**：给 canvas 一个 filter 过渡，让反色和背景同步淡入：

```css
.pdf-canvas-wrap canvas { transition: filter .25s ease; }
```

**修法 C（顺序）**：按「在当前滤镜状态下视觉保持稳定」的顺序来切 class 和像素：

```js
if (dark) {
    // 夜间：先换像素（此时 invert 还没生效 —— 视觉上没区别），再加 class（滤镜淡入）
    putImageData(darkCopy); document.documentElement.classList.add('dark');
} else {
    // 日间：先移除 class（滤镜淡出 —— 文字随背景从白变黑），再还原原始像素
    document.documentElement.classList.remove('dark'); putImageData(original);
}
```

三条都做完之后，主题切换就是一次平滑、同步的淡变：背景和 canvas 文字沿着同一条 0.25s 曲线插值。

## 坑 9：手机上全屏按钮是死的

**现象**：全屏按钮在移动端浏览器 / 微信 WebView 里点了没反应。

**原因**：iOS Safari 和很多 WebView 只对 `<video>` 支持 Fullscreen API；`div.requestFullscreen` 要么不存在，要么静默失败。

**修法**：先试原生全屏，失败就回退到 CSS 模拟 —— 加一个 class，让阅读器变成 `position:fixed; inset:0; z-index:9999`，并配一套自己的 flex 纵向布局。进入/退出时重新 fit 宽度（全屏用 `window.innerWidth`，普通模式用 doc-wrap 的宽度）。

## 小结

- 在 Markdown 站点里做 PDF = 懒加载的 `pdf.js` + 一次渲染一页 canvas + 用 nginx 路由把页面 URL（`/x.pdf` → SSR）和文件 URL（`/vault/x.pdf` → 静态）分开。
- 五类反复出现的 bug：编辑后的文件权限、大范围替换吃掉相邻函数、过早读取宽度、自动纠正循环和用户操作打架、函数作用域泄漏。
- canvas 上的主题切换需要三样东西：不能有 `transition: none !important`、要有像素缓冲缓存、要有和全局背景淡变同步的 `filter` 过渡。
- 移动端画质需要感知 DPR 的超采样渲染；扫描版 PDF 的夜间模式需要像素级的白→透明处理；移动端全屏需要 CSS 兜底。

## 相关

- [[排查 PHP 文件权限：编辑后站点报 Permission denied]]
- [[CSS 过渡与主题切换]]
