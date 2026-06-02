+++
title = "TIM PWM模式"
date = 2025-07-31
description = "整理 TIM PWM模式 相关笔记。"

[taxonomies]
tags = ["stm32", "hal"]
+++
# 1.简介
PWM（Pulse Width Modulation）脉冲宽度调制
在具有惯性的系统中，可以通过对一系列脉冲的宽度进行调制，来等效地获得所需要的模拟参量，常应用于电机控速等领域
PWM参数：
     频率 = 1 / TS            
     占空比 = TON / TS           
     分辨率 = 占空比变化步距 
     ![Pasted image 20240327220619](../../../../uploads/stm32-hal-library/pasted-image-20240327220619.png)

# 配置步骤
![Pasted image 20240327221444](../../../../uploads/stm32-hal-library/pasted-image-20240327221444.png)

# 关键结构体
```c
typedef struct 
{ 
   uint32_t OCMode; 	  /* 输出比较模式选择 */
   uint32_t Pulse; 	            /* 设置比较值 */
   uint32_t OCPolarity;       /* 设置输出比较极性 */
   uint32_t OCNPolarity;    /* 设置互补输出比较极性 */
   uint32_t OCFastMode;   /* 使能或失能输出比较快速模式 */
   uint32_t OCIdleState;     /* 空闲状态下OC1输出 */
   uint32_t OCNIdleState;  /* 空闲状态下OC1N输出 */ 
} TIM_OC_InitTypeDef;
```


# 示例代码
```c
htim2.Instance=TIM2;
    
    htim2.Init.Period=arr-1;
    htim2.Init.Prescaler=psc-1;
    htim2.Init.CounterMode=TIM_COUNTERMODE_UP;
    htim2.Init.ClockDivision=TIM_CLOCKDIVISION_DIV1;//信号防干扰
    htim2.Init.AutoReloadPreload=TIM_AUTORELOAD_PRELOAD_ENABLE;

    HAL_TIM_PWM_Init(&htim2);
    __HAL_TIM_CLEAR_FLAG(&htim1,TIM_FLAG_UPDATE);

    htim2_ch1_config.OCMode=TIM_OCMODE_PWM1;
    htim2_ch1_config.Pulse=500;
    htim2_ch1_config.OCPolarity=TIM_OCPOLARITY_HIGH;
    htim2_ch1_config.OCFastMode=TIM_OCFAST_DISABLE;

    HAL_TIM_PWM_ConfigChannel(&htim2,&htim2_ch1_config,TIM_CHANNEL_1);
```

