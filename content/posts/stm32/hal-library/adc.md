+++
title = "ADC"
date = 2026-03-25
description = "整理 ADC 相关笔记。"

[taxonomies]
tags = ["stm32", "hal"]
+++
# 简介
ADC，全称：Analog-to-Digital Converter，指模拟/数字转换器
![Pasted image 20240416141900](../../../../uploads/stm32-hal-library/pasted-image-20240416141900.png)
# 配置步骤
![Pasted image 20240416142300](../../../../uploads/stm32-hal-library/pasted-image-20240416142300.png)
# 关键结构体
```c
typedef struct 
{ 
	ADC_TypeDef *Instance; 			/* ADC 寄存器基地址 */ 
	ADC_InitTypeDef Init; 				/* ADC 参数初始化结构体变量 */ 
	DMA_HandleTypeDef *DMA_Handle; 	/* DMA 配置结构体 */
	…… 
} ADC_HandleTypeDef;
typedef struct
 { 
	uint32_t DataAlign; 					/* 设置数据的对齐方式 */ 
	uint32_t ScanConvMode; 				/* 扫描模式 */ 
	FunctionalState ContinuousConvMode; 	/* 开启单次转换模式或者连续转换模式 */ 	
	uint32_t NbrOfConversion; 				/* 设置转换通道数目 */ 
	FunctionalState DiscontinuousConvMode; 	/* 是否使用规则通道组间断模式 */ 
	uint32_t NbrOfDiscConversion; 			/* 配置间断模式的规则通道个数 */ 
	uint32_t ExternalTrigConv; 				/* ADC 外部触发源选择 */ 
} ADC_InitTypeDef;

```

# 示例代码
```c
ADC_HandleTypeDef adc1;
ADC_ChannelConfTypeDef adc1_0;


void ADC1_Init(void)
{
    

    adc1.Instance                   = ADC1;
    adc1.Init.ContinuousConvMode    = ENABLE;//连续扫描模式
    adc1.Init.DataAlign             = ADC_DATAALIGN_RIGHT;//对齐方式
    //adc1.Init.DiscontinuousConvMode = ;//规则组间断模式
    adc1.Init.ExternalTrigConv      = ADC_SOFTWARE_START;//规则组外部触发
    //adc1.Init.NbrOfConversion       = ;//扫描模式下规则组通道数量
    //adc1.Init.NbrOfDiscConversion   = ;//规则组间断模式下每次间断的通道数
    adc1.Init.ScanConvMode          = ADC_SCAN_DISABLE;//扫描方式

    HAL_ADC_Init(&adc1);

    adc1_0.Channel=ADC_CHANNEL_0;
    adc1_0.Rank=ADC_REGULAR_RANK_1;
    adc1_0.SamplingTime=ADC_SAMPLETIME_41CYCLES_5;

    HAL_ADC_ConfigChannel(&adc1,&adc1_0);

    HAL_ADCEx_Calibration_Start(&adc1);//自校准
}

void HAL_ADC_MspInit(ADC_HandleTypeDef *hadc)
{
    if (hadc->Instance == ADC1) {
        __HAL_RCC_GPIOA_CLK_ENABLE();
        __HAL_RCC_ADC1_CLK_ENABLE();

        GPIO_InitTypeDef GPIO_Init;

        GPIO_Init.Pin   = GPIO_PIN_0;
        GPIO_Init.Mode  = GPIO_MODE_ANALOG;
        GPIO_Init.Speed = GPIO_SPEED_HIGH;

        HAL_GPIO_Init(GPIOA, &GPIO_Init);

        HAL_NVIC_SetPriority(ADC1_2_IRQn,3,0);
        HAL_NVIC_EnableIRQ(ADC1_2_IRQn);
    }
}
```
