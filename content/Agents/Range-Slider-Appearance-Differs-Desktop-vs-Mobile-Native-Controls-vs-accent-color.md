## Symptom

项目里有一个 `<input type="range">` 滑动条（比如 UUID 数量滑杆），在桌面 Chrome 上显示为「黑色圆球 + 白描边」的 thumb，但同一条代码在 iOS Safari 上显示为纯白色 thumb（系统原生风格）。两者形状、颜色完全不同。

## Principle

### 根因：没有自定义 thumb，全靠浏览器默认控件

样式里只写了：

```css
.range-row input[type="range"] {
    flex: 1;
    accent-color: var(--accent);
    height: 28px;
    cursor: pointer;
}
```

**`accent-color` 只控制轨道（track）已滑过区域的填充颜色，不控制 thumb（滑块）本身。**

Thumb 的外观完全由浏览器用原生系统控件渲染：

- **桌面 Chrome/Chromium**：原生 range thumb = 黑色圆球 + 白描边，轨道跟随 `accent-color`
- **iOS Safari**：原生 range thumb = 白色圆形滑块（系统 UI 风格），轨道跟随 `accent-color`
- **Firefox**：原生 range thumb = OS 原生风格，`accent-color` 控制已填充轨道部分

这是「有意为之」的浏览器行为差异——不是 bug，也不是我们写死了两套样式。

## Solutions

### Option 1：什么都不做（推荐）

保留原生控件。桌面端的黑球外观干净，iOS 的白色滑块手感更稳（触摸面积大、摩擦感好）。大部分人不会觉得这是问题。

### Option 2：自定义 thumb，统一跨平台外观

如果确实需要统一，必须用伪元素覆盖原生控件：

```css
.range-row input[type="range"] {
    -webkit-appearance: none;
    appearance: none;
    background: transparent;
}

/* WebKit / Blink（Chrome、Safari、Edge） */
.range-row input[type="range"]::-webkit-slider-runnable-track {
    height: 4px;
    border-radius: 2px;
    background: var(--border);
}
.range-row input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background: var(--text);
    border: 2px solid var(--surface);
    margin-top: -7px;
}

/* Firefox */
.range-row input[type="range"]::-moz-range-track {
    height: 4px;
    border-radius: 2px;
    background: var(--border);
}
.range-row input[type="range"]::-moz-range-thumb {
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background: var(--text);
    border: 2px solid var(--surface);
}
```

效果：桌面和移动端都显示自定义深色圆球，不随操作系统变化。

**注意：** 这样做会让 iOS 上的 thumb 变小，可能比原生白色滑块更难精确触摸。

### Option 3：只让 iOS 外观好看一点（折中）

给 iOS 额外的触摸区域补偿，同时保持自定义外观：

```css
.range-row input[type="range"]::-webkit-slider-thumb {
    width: 24px;   /* 更大触摸区域 */
    height: 24px;
    border-radius: 50%;
    background: var(--text);
    border: 2px solid var(--surface);
}
```

## Summary

- `accent-color` 只影响轨道填充色，不影响 thumb 外观。Thumb 走浏览器原生系统控件。
- 桌面 Chrome 原生 thumb = 黑球白边；iOS Safari 原生 thumb = 白色圆块。这是浏览器默认行为差异，不是代码 bug。
- 若要统一，必须用 `::-webkit-slider-thumb` / `::-moz-range-thumb` 完全覆盖原生控件。
- 保留原生通常更好——iOS 白色滑块的触摸体验更稳。
- 相关笔记：`Why-SVG-Icons-Flicker-on-Click.md`（`-webkit-backface-visibility` 层提升问题）。
