---
title: Katex公式
date: 2026-09-27
tags:
  - markdown
draft: false
---



## 公式

行内写法：质能方程 $E = mc^2$，混在句子里没问题。

独占一行：

$$
S = \sum_{i=1}^{n} \frac{(x_i - \bar{x})^2}{n}
$$

| 源码 | 效果 |
| --- | --- |
| `$x^2$` `$x_i$` | $x^2$ $x_i$ |
| `$\frac{a}{b}$` | $\frac{a}{b}$ |
| `$\sqrt{x}$` | $\sqrt{x}$ |
| `$\sum_{i=1}^{n}$` `$\int_0^1$` | $\sum_{i=1}^{n}$ $\int_0^1$ |
| `$\alpha \beta \pi \Delta$` | $\alpha \beta \pi \Delta$ |
| `$\pm \times \div \neq \leq \geq \approx$` | $\pm \times \div \neq \leq \geq \approx$ |
| `$\vec{a}$` `$\hat{y}$` | $\vec{a}$ $\hat{y}$ |
| `$\begin{matrix} a & b \\ c & d \end{matrix}$` | $\begin{matrix} a & b \\ c & d \end{matrix}$ |
