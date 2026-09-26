## Symptom

移动端（手机浏览器）打开站点时，页面能**左右拖动**、宽度"不稳定"，有种能水平滑动的感觉。而用户对比的其他站点"卡得很死，只能上下滚动"。全站没有任何可见的横向滑块、没有横向滚动条，却依然能左右拖。

## Principle

### 根因：某个元素把文档撑宽了

横移能力 = `document.documentElement.scrollWidth > clientWidth`。只要任意元素的实际右边距超出视口宽度（`getBoundingClientRect().right > innerWidth`）或左边距越过 0，默认 `overflow-x: visible` 就会让页面产生水平滚动空间，于是能左右拖。

典型"元凶"类别（本项目实测遇到）：
1. **无 `flex-wrap` 的行内 flex**：一行里放 `datetime-local` 大控件 + 按钮，英文文案长时总宽超过容器。
2. **grid 内的 `input` 没限宽**：`input` 有浏览器默认 size（约 170px），`grid-template-columns: repeat(auto-fit, minmax(140px,1fr))` 里没有 `width:100%; min-width:0` 就溢出。
3. **隐藏的 fixed 抽屉/画布**：`drawer` 用 `transform: translateX(100%)` 藏到视口外时，其盒模型右边缘仍在视口右边外（fixed 不产生滚动条，但 getBoundingClientRect 会报溢出，且部分浏览器手势受影响）。

### 移动端更敏感的原因

桌面浏览器 overflow 通常附带回弹滚动条或干脆不出现横移；手机上触摸滚动直接让用户"感觉"宽度不稳定，且 iOS 橡皮筋回弹会放大这种感觉。

## Solutions

### Option 1（兜底，本项目采用）：整体锁死横向滚动

```css
html, body { overflow-x: hidden; }
```

一行即可让任何残余溢出都不再可横向拖动。`overflow-x: hidden` 会创建滚动容器，纯静态站无 sticky 依赖时可放心用；若有 `position: sticky`，改用现代 `overflow-x: clip`（不创建滚动容器，但对旧 Safari（<16）需降级）。

注意：这只是"掩盖"，真正的溢出元素仍存在，最好同时修掉（见 Option 2）。

### Option 2（定位并修复真实溢出，推荐）

用 headless Chromium 检测元凶：

```js
// 1) 视口设为手机宽度，例如 360px, mobile:true（Emulation.setDeviceMetricsOverride）
// 2) 逐页判断
document.documentElement.scrollWidth > document.documentElement.clientWidth
// 3) 列出越界元素
[...document.querySelectorAll('*')].filter(el => {
  const r = el.getBoundingClientRect();
  return r.right > innerWidth + 1 || r.left < -1;
}).map(el => el.tagName + '.' + el.className + '#' + el.id)
```

对定位到的溢出元素对症修复（本项目三个案例）：
- **timestamp 页**：inline `display:flex; ...` 加 `flex-wrap:wrap`（或改用已有 `.btn-row`）。
- **批量重命名页 `.input-grid`**：

```css
.input-grid input, .input-grid select { width: 100%; min-width: 0; box-sizing: border-box; }
```

- **隐藏的抽屉**：加 `overflow-x: hidden` 兜底后不再产生可滚动的横向空间。

验证条件要覆盖**最窄视口 + 最长文本**（本项目用 360px + zh/en 两语言全页回归），因为英文 label 最长、最容易触发。

## Summary

- 能不能横移，判断标准只有一个：`scrollWidth > clientWidth`。量它、再对越界元素动手，别只猜。
- 三类高频元凶：无 wrap 的 flex、grid 里未限宽的 input、藏在视口外的 fixed 面板。
- `html,body{overflow-x:hidden}` 是好用兜底（锁死拖动），但真实溢出还要修，否则弹层/动画里会出现怪位。
- 移动端务必同时测 en/zh，英文文案长，是溢出雷区。
- 相关笔记：`Page-Jumping-on-Mobile` 系列（`Mobile-Input-Focus-Zoom-and-Page-Jumping-Causes-and-CSS-JS-Workarounds.md`），同属移动端布局稳定性排查。