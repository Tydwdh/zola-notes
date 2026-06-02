+++
title = "latex 基础"
date = 2026-03-25
description = "梳理 Makefile 的基本语法。"

[taxonomies]
tags = ["latex"]
+++
## 1. 基本写法

### 行内公式
用单个 `$` 包起来：
`$E = mc^2$`

显示效果：$E = mc^2$

---

### 独立公式

用两个 `$$` 包起来：

```latex
$$
E = mc^2
$$
```

显示效果：

$$
E = mc^2
$$

---

## 2. 上标和下标

```latex
$x^2$
$x_1$
$x_i^2$
$x_{ij}$
```

显示效果：

* $x^2$
* $x_1$
* $x_i^2$
* $x_{ij}$

> 多个字符做上标或下标时，要用 `{}` 括起来。

---

## 3. 分数

```latex
$\frac{a}{b}$
$\frac{x+1}{y-1}$
```

显示效果：

* $\frac{a}{b}$
* $\frac{x+1}{y-1}$

---

## 4. 根号

```latex
$\sqrt{x}$
$\sqrt[n]{x}$
```

显示效果：

* $\sqrt{x}$
* $\sqrt[n]{x}$

---

## 5. 希腊字母

```latex
$\alpha, \beta, \gamma, \theta, \lambda, \mu, \sigma, \omega$
$\Gamma, \Delta, \Theta, \Lambda, \Sigma, \Omega$
```

显示效果：

* $\alpha, \beta, \gamma, \theta, \lambda, \mu, \sigma, \omega$
* $\Gamma, \Delta, \Theta, \Lambda, \Sigma, \Omega$

---

## 6. 常见运算符

```latex
$a + b$
$a - b$
$a \times b$
$a \div b$
$a \cdot b$
```

显示效果：

* $a + b$
* $a - b$
* $a \times b$
* $a \div b$
* $a \cdot b$

---

## 7. 关系符号

```latex
$a = b$
$a \neq b$
$a < b$
$a > b$
$a \leq b$
$a \geq b$
```

显示效果：

* $a = b$
* $a \neq b$
* $a < b$
* $a > b$
* $a \leq b$
* $a \geq b$

---

## 8. 求和与积分

### 求和

```latex
$\sum_{i=1}^{n} i$
```

显示效果：

$$
\sum_{i=1}^{n} i
$$

### 连乘

```latex
$\prod_{i=1}^{n} i$
```

显示效果：

$$
\prod_{i=1}^{n} i
$$

### 积分

```latex
$\int_a^b f(x)\,dx$
```

显示效果：

$$
\int_a^b f(x),dx
$$

### 极限

```latex
$\lim_{x \to 0} \frac{\sin x}{x}$
```

显示效果：

$$
\lim_{x \to 0} \frac{\sin x}{x}
$$

---

## 9. 括号自动适配大小

```latex
$\left( \frac{a}{b} \right)$
$\left[ \sum_{i=1}^n i \right]$
```

显示效果：

* $\left( \frac{a}{b} \right)$
* $\left[ \sum_{i=1}^n i \right]$

---

## 10. 矩阵

```latex
$$
\begin{bmatrix}
1 & 2 \\
3 & 4
\end{bmatrix}
$$
```

显示效果：

$$
\begin{bmatrix}
1 & 2 \
3 & 4
\end{bmatrix}
$$

常见矩阵环境：

* `matrix`：无括号
* `pmatrix`：圆括号
* `bmatrix`：方括号
* `Bmatrix`：大括号
* `vmatrix`：单竖线
* `Vmatrix`：双竖线

---

## 11. 分段函数

```latex
$$
f(x)=
\begin{cases}
x^2, & x \ge 0 \\
-x, & x < 0
\end{cases}
$$
```

显示效果：

$$
f(x)=
\begin{cases}
x^2, & x \ge 0 \
-x, & x < 0
\end{cases}
$$

---

## 12. 对齐公式

```latex
$$
\begin{aligned}
a+b &= c \\
x+y &= z
\end{aligned}
$$
```

显示效果：

$$
\begin{aligned}
a+b &= c \
x+y &= z
\end{aligned}
$$

> `&` 用来指定对齐位置。

---

## 13. 空格与省略号

### 空格

```latex
a\,b
a\;b
a\quad b
a\qquad b
```

### 省略号

```latex
$\cdots$
$\ldots$
```

显示效果：

* $\cdots$
* $\ldots$

---

## 14. 常用函数写法

```latex
$\sin x$
$\cos x$
$\tan x$
$\log x$
$\ln x$
$\exp x$
```

显示效果：

* $\sin x$
* $\cos x$
* $\tan x$
* $\log x$
* $\ln x$
* $\exp x$

> 常见函数要写成 `\sin`、`\log` 这种形式，不要直接写 `sin`。

---

## 15. 向量、帽子、横线

```latex
$\vec{a}$
$\hat{x}$
$\bar{x}$
$\overline{AB}$
```

显示效果：

* $\vec{a}$
* $\hat{x}$
* $\bar{x}$
* $\overline{AB}$

---

## 16. 常见集合符号

```latex
$\in$
$\notin$
$\subset$
$\subseteq$
$\cup$
$\cap$
$\emptyset$
$\forall$
$\exists$
```

显示效果：

* $\in$
* $\notin$
* $\subset$
* $\subseteq$
* $\cup$
* $\cap$
* $\emptyset$
* $\forall$
* $\exists$

---

## 17. 多行换行规则

在矩阵、对齐、分段函数里：

* 用 `\\` 换行
* 用 `&` 分列或对齐

示例：

```latex
$$
\begin{aligned}
f(x) &= x^2 + 1 \\
g(x) &= 2x - 3
\end{aligned}
$$
```

---

## 18. 转义字符

有些符号本身有特殊含义，需要转义：

```latex
\$
\%
\_
\{
\}
```

例如想显示美元符号：

```latex
$\$100$
```

---

## 19. 常见错误

### 1）上下标多个字符没加 `{}`

错误：

```latex
$x^10$
```

实际只会让 `1` 成为上标。

正确：

```latex
$x^{10}$
```

---

### 2）函数名直接写字母

错误：

```latex
$sin x$
```

正确：

```latex
$\sin x$
```

---

### 3）括号大小不匹配

推荐写法：

```latex
\left( \frac{a}{b} \right)
```

---

## 20. 实用例子

### 勾股定理

```latex
$$
a^2 + b^2 = c^2
$$
```

### 一元二次方程求根公式

```latex
$$
x=\frac{-b\pm\sqrt{b^2-4ac}}{2a}
$$
```

### 欧拉公式

```latex
$$
e^{i\theta}=\cos\theta+i\sin\theta
$$
```

### 高斯公式

```latex
$$
\sum_{i=1}^{n} i = \frac{n(n+1)}{2}
$$
```

---

## 21. Markdown 中使用 LaTeX 的提醒

不同平台对公式支持不一样，常见情况是：

* 行内公式：`$...$`
* 块公式：`$$...$$`

如果不显示公式，通常有两个原因：

1. 当前 Markdown 平台不支持 LaTeX
2. 公式分隔符写错了，或者少了括号 / 反斜杠

---

## 22. 最常用模板

### 行内公式模板

```latex
$公式内容$
```

### 块公式模板

```latex
$$
公式内容
$$
```

### 分数模板

```latex
$\frac{分子}{分母}$
```

### 求和模板

```latex
$\sum_{i=1}^{n} 表达式$
```

### 积分模板

```latex
$\int_a^b 表达式 \, dx$
```

### 矩阵模板

```latex
$$
\begin{bmatrix}
a & b \\
c & d
\end{bmatrix}
$$
```

---

## 23. 一句话记忆

* 上标：`^`
* 下标：`_`
* 分数：`\frac{}{}`
* 根号：`\sqrt{}`
* 求和：`\sum`
* 积分：`\int`
* 希腊字母：`\alpha \beta \gamma`
* 对齐：`aligned`
* 矩阵：`bmatrix`

```

你要的话，我也可以顺手再给你整理一个 **更适合 Obsidian / Typora / GitHub 的精简版 md 笔记**。
```
