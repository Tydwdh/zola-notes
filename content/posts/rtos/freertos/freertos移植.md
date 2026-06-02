+++
title = "FreeRTOS移植"
date = 2025-07-10
description = "整理 FreeRTOS移植 相关笔记。"

[taxonomies]
tags = ["freertos", "rtos"]
+++
# 核心文件

![Pasted image 20240418185113](../../../../uploads/freertos/pasted-image-20240418185113.png)
官方示例中
![Pasted image 20240418185145](../../../../uploads/freertos/pasted-image-20240418185145.png)

# FreeRTOSConfig.h文件的必要修改
加入以下代码
```c
//必须的宏定义
#define xPortPendSVHandler		PendSV_Handler//中断函数
#define vPortSVCHandler			SVC_Handler//中断函数
#define INCLUDE_xTaskGetSchedulerState	1
```

# stm32f1xx_it.c文件的必要修改
1.删掉函数**PendSV_Handler**和**SVC_Handler**函数
2.修改**SysTick_Handler**函数
```c
void SysTick_Handler(void)
{

  HAL_IncTick();
#if (INCLUDE_xTaskGetSchedulerState  == 1 )
  if (xTaskGetSchedulerState() != taskSCHEDULER_NOT_STARTED)
  {
#endif/* INCLUDE_xTaskGetSchedulerState */
    xPortSysTickHandler();
#if (INCLUDE_xTaskGetSchedulerState  == 1 )
  }
#endif/* INCLUDE_xTaskGetSchedulerState */

}
```

