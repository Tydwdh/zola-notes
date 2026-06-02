+++
title = "SPI通信外设"
date = 2026-05-19
description = "整理 SPI通信外设 相关笔记。"

[taxonomies]
tags = ["stm32", "standard-library"]
+++
# 1.外设简介
- STM32内部集成了硬件SPI收发电路，可以由硬件自动执行时钟生成、数据收发等功能，减轻CPU的负担
- 可配置8位/16位数据帧、高位先行/低位先行
- 时钟频率： fPCLK / (2, 4, 8, 16, 32, 64, 128, 256)
- 支持多主机模型、主或从操作
- 可精简为半双工/单工通信
- 支持DMA
- 兼容I2S协议(音频传输协议)
-  STM32F103C8T6 硬件SPI资源：SPI1(72MHZ，APB2总线)、SPI2(36MHZ,APB1总线)

# 2.SPI框图
![Pasted image 20231030164752](../../../../uploads/stm32-standard-library/pasted-image-20231030164752.png)
# 3.SPI基本框图
![Pasted image 20231030170110](../../../../uploads/stm32-standard-library/pasted-image-20231030170110.png)
- **TDR**：这是发送数据寄存器。当你想要发送数据时，你需要将数据写入此寄存器。

- **TXE**：这是发送缓冲区空闲标志。当此标志为'1'时，表示发送缓冲区为空，可以写入下一个待发送的数据进入缓冲区中。当写入SPI_DR时，TXE标志被清除。

- **RDR**：这是接收数据寄存器。当你接收到数据时，数据会被放入此寄存器。

- **RXNE**：这是接收缓冲区非空标志。当此标志为'1'时，表示接收缓冲区中包含有效的接收数据。读取SPI数据寄存器可以清除此标志。
# 4.两种传输方法

## 1.主模式全双工连续传输(速度快)
![Pasted image 20231030172319](../../../../uploads/stm32-standard-library/pasted-image-20231030172319.png)
## 2.非连续传输(推荐)
![Pasted image 20231030172432](../../../../uploads/stm32-standard-library/pasted-image-20231030172432.png)
