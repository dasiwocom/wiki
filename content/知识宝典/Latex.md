# Obsidian 里 LaTeX（MathJax）从零学

> 重点：Obsidian 只支持**数学公式**，不是完整 LaTeX！ 分隔符： 行内公式：`$内容$` 块公式（单独居中）：`$$内容$$`

---

## 第1课：行内公式 $ $

作用：公式混在文字中间 写法：文字 `$公式$` 文字 例子：

```
勾股定理 $a^2+b^2=c^2$，很常用。
```

预览效果：勾股定理 $a^2+b^2=c^2$，很常用。

✅ 规则：

- `$` 紧贴公式，中间不要乱空格
- 只写数学符号，中文放外面

## 第2课：块公式 $$ $$

作用：单独占一行、居中，适合长公式

```
$$
a^2 + b^2 = c^2
$$
```

> 习惯：两个 `$$` 单独占一行，公式写中间，可读性更好

---

## 第3课：上下标（最常用）

- 上标：`^`
- 下标：`_`
    
    > 如果上/下标**多于1个字符**，必须用大括号 `{}` 包起来
    

练习：

```
$x^2$  $x_1$
$x^{n+1}$  $x_{i+2}$
```

预览：$x^2$ $x_1$ $x^{n+1}$ $x_{i+2}$

📌 练习任务：自己写出 $y^{2k}$、$a_{m-1}$

---

## 第4课：分式 \frac{分子}{分母}

语法：`\frac{分子}{分母}`

```
行内：$\frac12$
块公式：
$$
\frac{x+1}{x-2}
$$
```

预览：$\frac12$ $$ \frac{x+1}{x-2} $$

📌 练习：写出 $\frac{a+b}{2}$

---

## 第5课：根号 \sqrt

- 平方根：`\sqrt{...}`
- n次方根：`\sqrt[n]{...}`

```
$\sqrt{2},\quad \sqrt[3]{8}$
```

$\sqrt{2},\quad \sqrt[3]{8}$

> `\quad` 是公式里增加一点间距

---

## 第6课：求和、极限、积分

- 求和：`\sum`

```
$$
\sum_{k=1}^n k = \frac{n(n+1)}{2}
$$
```

$$ \sum_{k=1}^n k = \frac{n(n+1)}{2} $$

- 极限：`\lim`，箭头 `\to`

```
$$
\lim_{x\to 0}\frac{\sin x}{x}=1
$$
```

$$ \lim_{x\to 0}\frac{\sin x}{x}=1 $$

- 积分：`\int`，无穷 `\infty`

```
$$
\int_{0}^{+\infty} e^{-x} dx
$$
```

$$ \int_{0}^{+\infty} e^{-x} dx $$

---

## 第7课：希腊字母

小写：

```
$\alpha,\beta,\gamma,\theta,\pi,\lambda,\sigma,\omega$
```

$\alpha,\beta,\gamma,\theta,\pi,\lambda,\sigma,\omega$

大写（首字母大写）：

```
$\Gamma,\Delta,\Theta,\Pi,\Sigma,\Omega$
```

$\Gamma,\Delta,\Theta,\Pi,\Sigma,\Omega$

---

## 第8课：关系符号

```
$\le$ ≤  $\ge$ ≥  $\ne$ ≠
```

$\le,\ge,\ne$

---

## 第9课：矩阵 & 对齐公式

### 矩阵

`pmatrix` 圆括号矩阵；`bmatrix` 方括号矩阵；`&` 分隔列，`\\` 换行

```
$$
\begin{pmatrix}
1 & 2 \\
3 & 4
\end{pmatrix}
$$
```

$$ \begin{pmatrix} 1 & 2 \ 3 & 4 \end{pmatrix} $$

### 多行对齐 aligned

`&` 是对齐位置，一般放在等号前面

```
$$
\begin{aligned}
y &= 2x + 3 \\
y &= 5x - 1
\end{aligned}
$$
```

$$ \begin{aligned} y &= 2x + 3 \ y &= 5x - 1 \end{aligned} $$

### 分段函数 cases

```
$$
f(x)=
\begin{cases}
x, & x>0 \\
-x, & x\le 0
\end{cases}
$$
```

$$ f(x)= \begin{cases} x, & x>0 \ -x, & x\le 0 \end{cases} $$

---

## 第10课：括号

普通括号 `()`； 自动适配大小括号：`\left( ... \right)`

```
$$
\left(\frac{1}{2}\right)
$$
```

$$ \left(\frac{1}{2}\right) $$

---

# ⚠️ Obsidian 专属坑（一定要记住）

1. 公式外面的下划线 `_` 要写成 `\_`，公式内部 `_` 直接用
2. 公式里尽量不放中文，容易渲染失败
3. 原生不能自动编号公式，想要编号要装插件
4. 不要写 `\documentclass`、`\usepackage`，这些在 Obsidian 无效！
5. 源码模式看代码，切换预览模式才渲染公式

---

# 现在开始练习！

你先在 Obsidian 新建笔记，把下面这一段复制进去，切换预览看效果：

```
# 练习
行内公式：$x^2 + 3x + 2 = 0$

分式：$\frac{2}{3}$

求和：
$$
\sum_{i=1}^{100} i
$$

矩阵：
$$
\begin{bmatrix}
0 & 1 \\
1 & 0
\end{bmatrix}
$$
```

你看完之后，我出几道小题给你写，写完我帮你批改，怎么样？