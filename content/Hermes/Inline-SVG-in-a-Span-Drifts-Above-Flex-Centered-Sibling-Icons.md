## Symptom

导航栏（ToolBox，Apple 风格）里三个图标按钮并排：地球、菜单、主题（日/月）。地球和菜单是 `<svg>` 直接作为 flex 按钮的子元素；主题图标是 `<svg>` 被 **注入到 `<span id="themeIcon">` 里**。用户感知：**日/月图标比地球、菜单偏上约 1px**（"日月位置偏上"），高分屏上是亚像素漂移，放大截图和逐像素对比时才明显。三者的 svg 尺寸、CSS stroke 完全一致，无从 CSS 层面解释。

## Principle

### 根因：`<span>` 里的内联 svg 走的是"行框基线"，不走 flex 布局

- flex 容器的**直接子元素会被块化**（blockified），由按钮的 `align-items: center` 在按钮盒内垂直居中。
- 但 `<span><svg></span>` 里的 svg 是 **inline‑level** 元素：它参与父 span 的**行内排版**（line box），按 `vertical-align`（默认 `baseline`）对齐到该行文本基线附近。
- 于是这个 svg 根本不参与按钮的 flex 垂直居中，而是"坐在文字基线上"——相对被 flex 居中的兄弟图标上/下飘约半个 x‑height。

### 为什么用眼睛很难发现

1px（且是亚像素）的漂移在正常缩放看不出，只有放大截图逐像素对比才露馅。和 [[Debugging-Font-Mismatch-Between-SVG-and-HTML-Text]] 同一家族：**跨渲染上下文的不一致，必须用测量值而不是肉眼判断**。

### 怎样量化判定

```js
// 每个 svg 的中心 y 是否等于按钮的中心 y
function center(el) { const r = el.getBoundingClientRect(); return (r.top + r.bottom) / 2; }
center(btn)            // 按钮中心，例 26
center(btn.querySelector('svg'))  // 直接子元素图标 = 26 ✓
center(themeBtn.querySelector('svg')) // span 内图标 = 25 ✗ 偏上 1px
```

## Solutions

### Option 1（本方案采用）：让包装 span 变成无行高的弹性容器

```css
#themeIcon {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    line-height: 0;
}
```

三个要点：
- `flex` 让 svg 也被 `align-items: center` 居中，行为与其他图标按钮一致。
- `line-height: 0` 去掉基线所在行框的撑高影响，中心点才精确对齐。
- 测量复验：改后所有图标 `svgCy === btnCenter`（本项目实测 26 === 26）。

### Option 2：让 svg 脱离行内排版

```css
#themeIcon svg { display: block; }
/* 或 vertical-align: middle */
```

也能生效，但不如 Option 1 对容器级对齐可控。

### 顺带经验：同源"测量偏差"陷阱

- 图标几何尺寸对比用**无头 Chromium + canvas 光栅化**：`data:image/svg+xml` → `drawImage` → 遍历 alpha 求 ink bbox / 质心，量化的才是真实的。
- 但**静态服务器无缓存头时，浏览器按 URL 启发式缓存**——改了文件但 `?v=` 版本号没升，同一 URL 命中旧缓存，无头测量若复用同一 `--user-data-dir` 还会跨会话缓存。**结论：每次部署先升 URL 版本串（本项目 `20260919x` 日期+字母），测量前换全新 user-data-dir。**

### 顺带：细碎图标的笔画补偿

太阳/月亮这类多短笔画的字形，在同样 `stroke-width` 下**视觉上比地球的整环更细**（光学补偿规律），可单独加粗：

```css
#icon-btn-container svg { stroke-width: 1.8; }
#themeIcon svg          { stroke-width: 2.2; }
```

## Summary

- **span 包裹 = 基线对齐陷阱；flex 直接子元素 = flex 居中。** 被包一层，布局体系就换了，别按"都是同一个按钮里的图标"推断。
- 量 `svgCy` vs `btnCenter`，别靠眼睛——1px 亚像素漂移肉眼不可靠。
- `line-height: 0` 是把内联子元素对齐误差清零的关键一步。
- 改文件后测量结果仍是旧值：先怀疑 `?v=` 缓存与 user-data-dir 复用，再怀疑代码。
- 相关笔记：[[Theme-Toggle-Settled-on-Instant-Swap-Not-Animated-Transition]]、[[Why-SVG-Icons-Flicker-on-Click]]、[[Language-Switch-Flashes-on-Refresh-Same-FOUC-Family-as-Theme-Flash]]（同属导航栏/头部图标与主题的视觉问题）。