+++
title = "RTC"
date = 2026-05-19
description = "整理 RTC 相关笔记。"

[taxonomies]
tags = ["stm32", "standard-library"]
+++
# 1.RTC简介
•RTC（Real Time Clock）实时时钟

•RTC是一个独立的定时器，可为系统提供时钟和日历的功能

•RTC和时钟配置系统处于后备区域，系统复位时数据不清零，VDD（2.0~3.6V）断电后可借助VBAT（1.8~3.6V）供电继续走时

•32位的可编程计数器，可对应Unix时间戳的秒计数器

•20位的可编程预分频器，可适配不同频率的输入时钟

•可选择三种RTC时钟源：
	  HSE时钟除以128（通常为8MHz/128）
	  LSE振荡器时钟（通常为32.768KHz）（主要选择）
	  LSI振荡器时钟（40KHz）

# 2.RTC框图
![Pasted image 20231101144425](../../../../uploads/stm32-standard-library/pasted-image-20231101144425.png)

# 3.RTC基本框图
![Pasted image 20231101150547](../../../../uploads/stm32-standard-library/pasted-image-20231101150547.png)
# 4.注意事项
•执行以下操作将使能对BKP和RTC的访问：
  设置RCC_APB1ENR的PWREN和BKPEN，使能PWR和BKP时钟
  设置PWR_CR的DBP，使能对BKP和RTC的访问
  
•若在读取RTC寄存器时，RTC的APB1接口曾经处于禁止状态，则软件首先必须等待RTC_CRL寄存器中的RSF位（寄存器同步标志）被硬件置1

•必须设置RTC_CRL寄存器中的CNF位，使RTC进入配置模式后，才能写入RTC_PRL、RTC_CNT、RTC_ALR寄存器

•对RTC任何寄存器的写操作，都必须在前一次写操作结束后进行。可以通过查询RTC_CR寄存器中的RTOFF状态位，判断RTC寄存器是否处于更新中。仅当RTOFF状态位是1时，才可以写入RTC寄存器
