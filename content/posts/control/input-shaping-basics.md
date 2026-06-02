+++
title = "输入整形基础：ZV、ZVD 与 MZV"
date = 2026-04-09
description = "整理 ZV、ZVD、MZV 输入整形器的基本公式、卷积形式和选型建议。"

[taxonomies]
tags = ["control", "input-shaping", "embedded"]
+++
## 1. ZV（Zero Vibration）整形器

### 基本原理
最简单的整形器，使用两个脉冲来消除残余振动。对频率误差非常敏感。

### 核心公式

$$
\omega_d = \omega_n \sqrt{1 - \zeta^2} \quad \text{（阻尼自然频率）}
$$

$$
T_d = \frac{2\pi}{\omega_d} = \frac{2\pi}{\omega_n \sqrt{1 - \zeta^2}} \quad \text{（阻尼振动周期）}
$$

$$
K = \exp\left(-\frac{\zeta\pi}{\sqrt{1 - \zeta^2}}\right) \quad \text{（振动衰减系数）}
$$

### 脉冲参数

$$
\begin{aligned}
A_1 &= \frac{1}{1 + K} \\
A_2 &= \frac{K}{1 + K}
\end{aligned}
$$

$$
\begin{aligned}
t_1 &= 0 \\
t_2 &= \frac{\pi}{\omega_n \sqrt{1 - \zeta^2}} = \frac{T_d}{2}
\end{aligned}
$$

### 卷积公式（实际应用）
给定原始速度命令序列 \(v[n]\)，整形后命令 \(v_{\text{shaped}}[n]\) 为：

$$
v_{\text{shaped}}[n] = A_1 \cdot v[n] + A_2 \cdot v\left[n - \left\lfloor \frac{t_2}{T_s} \right\rfloor\right]
$$

其中 \( T_s \) 为控制周期。

---

## 2. ZVD（Zero Vibration and Derivative）整形器

### 基本原理
使用三个脉冲，不仅消除振动，还消除振动对时间的导数，对频率误差的鲁棒性比ZV更好。

### 脉冲参数

$$
\begin{aligned}
A_1 &= \frac{1}{1 + 2K + K^2} \\
A_2 &= \frac{2K}{1 + 2K + K^2} \\
A_3 &= \frac{K^2}{1 + 2K + K^2}
\end{aligned}
$$

$$
\begin{aligned}
t_1 &= 0 \\
t_2 &= \frac{\pi}{\omega_n \sqrt{1 - \zeta^2}} = \frac{T_d}{2} \\
t_3 &= \frac{2\pi}{\omega_n \sqrt{1 - \zeta^2}} = T_d
\end{aligned}
$$

### 幅度总和验证
$$
\sum_{i=1}^{3} A_i = \frac{1 + 2K + K^2}{1 + 2K + K^2} = 1
$$

---

## 3. MZV（Modified Zero Vibration）整形器

### 基本原理
ZV的改进版本，通过在两个脉冲之间引入时间偏移来提高鲁棒性，同时保持较短的整形时间。

### 核心公式（MZV变形1）
这是3D打印中最常用的MZV形式：

$$
\begin{aligned}
A_1 &= \frac{1}{1 + K} \\
A_2 &= \frac{K}{1 + K} \\
A_3 &= -\frac{K}{(1 + K)^2} \quad \text{（负脉冲，幅度很小）}
\end{aligned}
$$

$$
\begin{aligned}
t_1 &= 0 \\
t_2 &= \frac{\pi}{\omega_n \sqrt{1 - \zeta^2}} = \frac{T_d}{2} \\
t_3 &= \frac{\pi}{\omega_n} \quad \text{（注意：不是 } T_d/2 \text{）}
\end{aligned}
$$

### 替代形式（MZV变形2）
另一种常见的MZV形式，使用三个正脉冲：

$$
K = \exp\left(-\frac{\zeta\pi}{\sqrt{1 - \zeta^2}}\right)
$$

$$
\alpha = \frac{\sqrt{K}}{1 + \sqrt{K}} \quad \text{（中间变量）}
$$

$$
\begin{aligned}
A_1 &= \alpha \\
A_2 &= 1 - 2\alpha \\
A_3 &= \alpha
\end{aligned}
$$

$$
\begin{aligned}
t_1 &= 0 \\
t_2 &= \frac{\pi}{\omega_n \sqrt{1 - \zeta^2}} \times \frac{1}{1 + \sqrt{K}} \\
t_3 &= \frac{\pi}{\omega_n \sqrt{1 - \zeta^2}} \times \frac{1}{1 + 1/\sqrt{K}}
\end{aligned}
$$

---

## 统一卷积公式

对于任意整形器，给定脉冲幅度向量 \( \mathbf{A} = [A_1, A_2, \dots, A_m] \) 和时间向量 \( \mathbf{t} = [t_1, t_2, \dots, t_m] \)，整形过程为：

$$
v_{\text{shaped}}[n] = \sum_{i=1}^{m} A_i \cdot v\left[n - \left\lfloor \frac{t_i}{T_s} \right\rfloor\right]
$$

其中 \( v[n] \) 为原始速度/加速度命令序列，\( T_s \) 为控制周期。

---

## 阻尼比估计公式

在3D打印中，当通过敲击测试测量频率时，还可以通过振动衰减计算阻尼比：

$$
\zeta = \frac{1}{\sqrt{1 + \left(\frac{2\pi}{\ln(r)}\right)^2}}
$$

其中 \(r = \frac{X_2}{X_1}\) 是相邻两个振动峰值的幅度比。


## 选择建议

1. **ZV**：最简单，整形时间最短，但对频率误差最敏感
2. **ZVD**：对频率误差的鲁棒性更好，但整形时间更长
3. **MZV**：在ZV和ZVD之间取得平衡，既有较好的鲁棒性，整形时间又相对较短
