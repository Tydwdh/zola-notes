+++
title = "TIM"
date = 2025-07-31
description = "整理 TIM 相关笔记。"

[taxonomies]
tags = ["stm32", "hal"]
+++
# 1.定时器概述
使用精准的时基，通过硬件的方式，实现定时功能
![Pasted image 20240210093516](../../../../uploads/stm32-hal-library/pasted-image-20240210093516.png)
# 2.定时器分类

|定时器类型|主要功能|
|---|---|
|基本定时器|没有输入输出通道，常用作时基，即定时功能|
|通用定时器|具有多路独立通道，可用于输入捕获/输出比较，也可用作时基|
|高级定时器|除具备通用定时器所有功能外，还具备带死区控制的互补信号输出、刹车输入等功能（可用于电机控制、数字电源设计等）|
# 3.定时器特性（F1）

|           |        定时器型号        | 定时器计数位数 |      计数模式      | 计数器重装值(装载寄存器) | 产生请求源 | 捕获/比较通道 | 互补输出 |
| :-------: | :-----------------: | :-----: | :------------: | :-----------: | :---: | :-----: | ---- |
| **基本定时器** |      TIM6 TIM7      |   16    |       递增       |    1~65536    |  可以   |    0    | 无    |
| **通用定时器** | TIM2 TIM3 TIM4 TIM5 |   16    | 递增上、递减、中央对齐方式  |    1~65536    |  可以   |    4    | 无    |
| **高级定时器** |      TIM1 TIM8      |   16    | 递增上、递减、中央对齐方式式 |    1~65536    |  可以   |    4    | 有    |
**stm32c8t6**可用定时器
- 高级：TIM1
- 通用：TIM2，TIM3，TIM4
- 基本：无

# 4.定时器溢出时间计算方法
![Pasted image 20240210102320](../../../../uploads/stm32-hal-library/pasted-image-20240210102320.png)

# 5.定时器中断实验配置步骤
![Pasted image 20240210102359](../../../../uploads/stm32-hal-library/pasted-image-20240210102359.png)
# 6.关键寄存器
![Pasted image 20240321184632](../../../../uploads/stm32-hal-library/pasted-image-20240321184632.png)
# 7.关键结构体
```c
typedef struct 
{ 
    TIM_TypeDef *Instance;            /* 外设寄存器基地址 */ 
    TIM_Base_InitTypeDef Init;     /* 定时器初始化结构体*/
     ...
}TIM_HandleTypeDef;

typedef struct 
{ 
    uint32_t Prescaler;                      /* 预分频系数 */ 
    uint32_t CounterMode;             /* 计数模式 */ 
    uint32_t Period;                           /* 自动重载值 ARR */ 
    uint32_t ClockDivision;             /* 时钟分频因子 */ 
    uint32_t RepetitionCounter;   /* 重复计数器寄存器的值 */ 
    uint32_t AutoReloadPreload; /* 自动重载预装载使能 */
} TIM_Base_InitTypeDef;

```

# 8.初始化代码
```c
TIM_HandleTypeDef htim1;

void Timer1_Init(uint16_t arr,uint16_t psc,uint8_t rep)
{
    htim1.Instance=TIM1;
    
    htim1.Init.Period=arr-1;
    htim1.Init.Prescaler=psc-1;
    htim1.Init.CounterMode=TIM_COUNTERMODE_UP;
    htim1.Init.ClockDivision=TIM_CLOCKDIVISION_DIV1;//信号防干扰
    htim1.Init.RepetitionCounter=rep;
    htim1.Init.AutoReloadPreload=TIM_AUTORELOAD_PRELOAD_ENABLE;
    
   

	HAL_TIM_Base_Init(&htim1);      
	
	__HAL_TIM_CLEAR_FLAG(&htim1,TIM_FLAG_UPDATE);//HAL_TIM_Base_Init(&htim1);会产生更新事件
	
}

void HAL_TIM_Base_MspInit(TIM_HandleTypeDef *htim)
{
    if(htim->Instance<mark>TIM1)
    {
        __HAL_RCC_TIM1_CLK_ENABLE();
        //HAL_NVIC_SetPriority(TIM1_UP_IRQn,3,0);
        //HAL_NVIC_EnableIRQ(TIM1_UP_IRQn);
    }
}

```
# 9.代码示例
## 1.TIM轮询方式
```c
HAL_TIM_Base_Start(&htim1);
	HAL_GPIO_WritePin(GPIOA,GPIO_PIN_0,GPIO_PIN_SET);

	while(1)
	{
		if(__HAL_TIM_GET_FLAG(&htim1,TIM_FLAG_UPDATE))
		{
			__HAL_TIM_CLEAR_FLAG(&htim1,TIM_FLAG_UPDATE);
            HAL_GPIO_WritePin(GPIOA,GPIO_PIN_0, (GPIO_PinState)!HAL_GPIO_ReadPin(GPIOA,GPIO_PIN_0));
			HAL_TIM_Base_Stop(&htim1);
			HAL_TIM_Base_DeInit(&htim1);
		}
		
	}
```

## 2.TIM中断方式
```c
void TIM1_UP_IRQHandler(void)
{
  HAL_TIM_IRQHandler(&htim1);
}

int main()
{
	HAL_Init();
	SystemTime_Init();
	LED_Init();
	Timer1_Init(7200,10000,0);
	HAL_GPIO_WritePin(GPIOA,GPIO_PIN_0,GPIO_PIN_SET);
	HAL_TIM_Base_Start_IT(&htim1);
	while(1)
	{

	}
}

void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
	if(htim->Instance</mark>TIM1)
	{
		HAL_GPIO_WritePin(GPIOA,GPIO_PIN_0,(GPIO_PinState)!HAL_GPIO_ReadPin(GPIOA,GPIO_PIN_0));
	}
}

```
## 3.DMA改变单次定时时长
DMA配置
```c
tim1_dma.Instance=DMA1_Channel5;
tim1_dma.Init.Direction=DMA_MEMORY_TO_PERIPH;
tim1_dma.Init.PeriphInc=DMA_PINC_DISABLE;
tim1_dma.Init.MemInc=DMA_MINC_ENABLE;
tim1_dma.Init.PeriphDataAlignment=DMA_PDATAALIGN_HALFWORD;
tim1_dma.Init.MemDataAlignment=DMA_MDATAALIGN_HALFWORD;
tim1_dma.Init.Mode=DMA_NORMAL;
tim1_dma.Init.Priority=DMA_PRIORITY_HIGH;
__HAL_LINKDMA(&htim1,hdma[TIM_DMA_ID_UPDATE],tim1_dma);
HAL_DMA_Init(&tim1_dma);
HAL_NVIC_SetPriority(DMA1_Channel5_IRQn,3,0);
HAL_NVIC_EnableIRQ(DMA1_Channel5_IRQn);
```
启动DMA中断
```c
HAL_TIM_Base_Start_DMA(&htim1,TIM_DMA_ID_UPDATE,(uint32_t*)arrbuff,4);
```
可以使用DMA在计时过程中更改arr的值
个人觉得用处不大，完全可以在中断中自己设置arr的值，
```c
__HAL_TIM_SetAutoreload(&htim1,arr);
```
