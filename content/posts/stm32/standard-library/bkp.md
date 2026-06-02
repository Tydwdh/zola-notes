+++
title = "BKP"
date = 2026-05-19
description = "整理 BKP 相关笔记。"

[taxonomies]
tags = ["stm32", "standard-library"]
+++
# 1.BKP简介
•BKP（Backup Registers）备份寄存器
•BKP可用于存储用户应用程序数据。当VDD（2.0~3.6V）电源被切断，他们仍然由VBAT（1.8~3.6V）维持供电。当系统在待机模式下被唤醒，或系统复位或电源复位时，他们也不会被复位
•TAMPER引脚产生的侵入事件将所有备份寄存器内容清除
•RTC引脚输出RTC校准时钟、RTC闹钟脉冲或者秒脉冲
•存储RTC时钟校准寄存器
•用户数据存储容量：20字节（中容量和小容量）/ 84字节（大容量和互联型）
# 2.BKP基本结构
![Pasted image 20231101144054](../../../../uploads/stm32-standard-library/pasted-image-20231101144054.png)
一个数据寄存器有16位，存储两个字节
