---
title: Katex公式
date: 2026-09-27
tags:
  - markdown
draft: false
---

Obsidian 本地用 MathJax 渲染，网站（Quartz）用 KaTeX，写法完全一样。

> [!tip] 这篇笔记怎么看
> 效果在上，下面紧跟源码块（用 `markdown` 语言标记，代码块里不会被渲染）。

## 两种写法

**行内**：单个 `$` 包起来，混在句子里 —— 质能方程 $E = mc^2$ 就是这样。

**块级**：两个 `$$` 独占一段，居中显示：

$$
S = \sum_{i=1}^{n} \frac{(x_i - \bar{x})^2}{n}
$$

```markdown
行内：$E = mc^2$

$$
S = \sum_{i=1}^{n} \frac{(x_i - \bar{x})^2}{n}
$$
```

## 符号速查

| 源码 | 效果 | 说明 |
| --- | --- | --- |
| `$x^2$` | $x^2$ | 上标 |
| `$x_i$` | $x_i$ | 下标 |
| `$x^{2n}$` | $x^{2n}$ | 多字符用花括号 |
| `$\frac{a}{b}$` | $\frac{a}{b}$ | 分数 |
| `$\sqrt{x}$` `$\sqrt[3]{x}$` | $\sqrt{x}$ $\sqrt[3]{x}$ | 根号 |
| `$\sum_{i=1}^{n}$` | $\sum_{i=1}^{n}$ | 求和 |
| `$\prod_{i=1}^{n}$` | $\prod_{i=1}^{n}$ | 连乘 |
| `$\int_0^1$` `$\iint$` | $\int_0^1$ $\iint$ | 积分 |
| `$\lim_{x \to 0}$` | $\lim_{x \to 0}$ | 极限 |
| `$\pm \times \div \cdot$` | $\pm \times \div \cdot$ | 四则 |
| `$\neq \leq \geq \approx \equiv$` | $\neq \leq \geq \approx \equiv$ | 关系符 |
| `$\vec{a}$` `$\hat{y}$` `$\bar{x}$` | $\vec{a}$ $\hat{y}$ $\bar{x}$ | 向量、估计值、均值 |
| `$\infty \partial \nabla$` | $\infty \partial \nabla$ | 无穷、偏导、梯度 |
| `$\to \Rightarrow \leftrightarrow$` | $\to \Rightarrow \leftrightarrow$ | 箭头 |
| `$\in \subset \cup \cap \emptyset$` | $\in \subset \cup \cap \emptyset$ | 集合 |
| `$\land \lor \neg \forall \exists$` | $\land \lor \neg \forall \exists$ | 逻辑 |
| `$\sin \cos \tan \log \ln$` | $\sin \cos \tan \log \ln$ | 函数（要加反斜杠） |
| `$\overline{AB}$` `$\underline{x}$` | $\overline{AB}$ $\underline{x}$ | 上划线、下划线 |

## 希腊字母 

| 源码           | 效果         | 源码              | 效果            |
| ------------ | ---------- | --------------- | ------------- |
| `$\alpha$`   | $\alpha$   | `$\beta$`       | $\beta$       |
| `$\gamma$`   | $\gamma$   | `$\Gamma$`      | $\Gamma$      |
| `$\delta$`   | $\delta$   | `$\Delta$`      | $\Delta$      |
| `$\epsilon$` | $\epsilon$ | `$\varepsilon$` | $\varepsilon$ |
| `$\theta$`   | $\theta$   | `$\Theta$`      | $\Theta$      |
| `$\lambda$`  | $\lambda$  | `$\Lambda$`     | $\Lambda$     |
| `$\mu$`      | $\mu$      | `$\pi$`         | $\pi$         |
| `$\rho$`     | $\rho$     | `$\sigma$`      | $\sigma$      |
| `$\tau$`     | $\tau$     | `$\phi$`        | $\phi$        |
| `$\omega$`   | $\omega$   | `$\Omega$`      | $\Omega$      |

> [!warning] 大写希腊字母只有 9 个有专门命令
> 只有这几个有：`\Gamma` `\Delta` `\Theta` `\Lambda` `\Xi` `\Pi` `\Sigma` `\Upsilon` `\Phi` `\Psi` `\Omega`。
>
> 剩下的 Α Β Ε Ζ Η Ι Κ Μ Ν Ο Ρ Τ Χ 跟拉丁字母 A B E Z H I K M N O P T X 长得一样，**直接写拉丁字母就行**。写 `\Alpha` `\Beta` 会报错飘红。

## 常用结构

**矩阵**（`&` 分列，`\\` 换行）

$$
\begin{bmatrix}
a & b \\
c & d
\end{bmatrix}
\quad
\begin{pmatrix}
1 & 2 \\
3 & 4
\end{pmatrix}
$$

**多行对齐**（用 `aligned`，等号对整齐）

$$
\begin{aligned}
(a+b)^2 &= a^2 + 2ab + b^2 \\
&= a^2 + b^2 + 2ab
\end{aligned}
$$

**方程组**（`cases`）

$$
f(x) =
\begin{cases}
x, & x > 0 \\
0, & x = 0 \\
-x, & x < 0
\end{cases}
$$

**自适应大小的括号**（`\left` `\right`）

$$
\left( \frac{a}{b} \right)^{2}
\quad
\left\{ x \mid x > 0 \right\}
$$

```markdown
$$
\begin{bmatrix}
a & b \\
c & d
\end{bmatrix}
$$

$$
\begin{aligned}
(a+b)^2 &= a^2 + 2ab + b^2 \\
&= a^2 + b^2 + 2ab
\end{aligned}
$$

$$
f(x) =
\begin{cases}
x, & x > 0 \\
-x, & x < 0
\end{cases}
$$

$$
\left( \frac{a}{b} \right)^{2}
$$
```

## 三个坑

> [!danger] 正文里的美元符号会被吃掉
> 写价格 `$5` 会被当成公式开头。必须转义成 `\$5`。

> [!warning] 本地能看，网站不一定能看
> Obsidian 的 MathJax 支持的命令比 KaTeX 多。发布前跑一次 `npx quartz build --serve` 预览确认。
> 不确定某个命令行不行，查 [KaTeX 支持列表](https://katex.org/docs/supported.html)。

> [!tip] 公式里怎么写中文
> 用 `\text{}` 包起来：$\text{平均值} = \frac{\text{总和}}{\text{个数}}$

## 参考

- 支持哪些命令：<https://katex.org/docs/supported.html>
- 在线试写：<https://katex.org/#demo>
