+++
title = "低通滤波器公式笔记"
date = 2025-11-03
description = "记录一阶 IIR、二阶 IIR 和 FIR 低通滤波器的基本差分方程与系数获取方式。"

[taxonomies]
tags = ["filter", "control", "embedded"]
+++
# 1. 一阶 IIR 低通滤波器（First-Order IIR LPF）

**差分方程**

$$
y[n] = \alpha \cdot x[n] + (1 - \alpha) \cdot y[n-1]
$$
> 每次调用：只需 1 次乘法、1 次加法、1 次赋值。

---

### 2. 高阶 IIR 低通滤波器（以二阶节 Biquad 为例，可级联）

**差分方程（Direct Form I）**：
$$
y[n] = b_0 x[n] + b_1 x[n-1] + b_2 x[n-2] - a_1 y[n-1] - a_2 y[n-2]
$$

> 注意：此处的 \(a_1, a_2\) 是**带符号的系数**（即公式中为减号）。
> 说明：
> - 这是 **Direct Form I**，数值稳定性较好。
> - 若用 **Direct Form II**，状态变量更少（仅 2 个），但对量化噪声更敏感，嵌入式中慎用。
> - 高阶（如 4 阶）可级联两个二阶节（SOS）。

---
### 3. FIR 低通滤波器（N 阶，对称系数，线性相位）

**差分方程**：
$$
y[n] = \sum_{k=0}^{N-1} h[k] \cdot x[n - k]
$$
> 优化建议：
> - 若 `N` 是常数且较小（如 8、16），编译器可自动展开循环。
> - 系数 `h` 可通过窗函数法（如 Hamming + sinc）或 `scipy.signal.firwin` 生成。

---

### 补充：系数如何获取？

- **一阶 IIR**：  
  $$
  \alpha = \frac{2\pi f_c / f_s}{1 + 2\pi f_c / f_s}
  $$
fc为截至频率，fs为采样频率

对于一阶低通滤波器，**群延迟（group delay）近似为**：
$$
T=\frac{1-\alpha}{\alpha}
$$
