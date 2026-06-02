+++
title = "DMA"
date = 2025-07-31
description = "整理 DMA 相关笔记。"

[taxonomies]
tags = ["stm32", "hal"]
+++
# 1.简介
DMA，全称Direct Memory Access，即直接存储器访问。
DMA传输 将数据从一个地址空间复制到另一个地址空间。
DMA传输无需CPU直接控制传输，也没有中断处理方式那样保留现场和恢复现场过程，通过硬件为RAM和IO设备开辟一条直接传输数据的通道，使得CPU的效率大大提高
作用：为CPU减负
# 2.DMA框图
![Pasted image 20240121190748](../../../../uploads/stm32-hal-library/pasted-image-20240121190748.png)
![Pasted image 20240121190811](../../../../uploads/stm32-hal-library/pasted-image-20240121190811.png)
不同外设向DMA的不同通道发送请求
# 3.DMA优先级
仲裁器管理DMA通道请求分为两个阶段：软件阶段(1)、硬件阶段(2)
第一阶段（软件阶段）：每个通道的优先级可在DMA_CCRx寄存器中设置，有四个等级：最高、高、中和低优先级。
第二阶段（硬件阶段）：如果两个请求有相同软件优先级，较低编号的通道比较高编号的通道有较高的优先级。
![Pasted image 20240121194922](../../../../uploads/stm32-hal-library/pasted-image-20240121194922.png)
![Pasted image 20240122221056](../../../../uploads/stm32-hal-library/pasted-image-20240122221056.png)

# 4.DMA相关HAL库驱动介绍（掌握）
| 驱动函数 | 功能描述 |
| ---- | ---- |
| __HAL_RCC_DMAx_CLK_ENABLE(…) | 使能DMAx时钟 |
| HAL_DMA_Init(…) | 初始化DMA |
| HAL_DMA_Start_IT(…) | 开始DMA传输 |
| <mark>__HAL_LINKDMA(…)</mark> | <mark>用来连接DMA和外设句柄</mark> |
| HAL_UART_Transmit_DMA(…) | 使能DMA发送，启动传输 |
| __HAL_DMA_GET_FLAG(…) | 查询DMA传输通道的状态 |
| __HAL_DMA_ENABLE(…) | 使能DMA外设 |
| __HAL_DMA_DISABLE(…) | 失能DMA外设 |
DMA外设相关结构体：
	·DMA_HandleTypeDef 
		`DMA_Channel_TypeDef  Instance`
		`DMA_InitTypeDef   Init`
	·DMA_InitTypeDef
		`uint32_t Direction			/* DMA传输方向 */`
		`uint32_t PeriphInc			/* 外设地址(非)增量 */`
		`uint32_t MemInc			/* 存储器地址(非)增量*/`
		`uint32_t PeriphDataAlignment	/* 外设数据宽度 */`
		`uint32_t MemDataAlignment	/* 存储器数据宽度 */`
		`uint32_t Mode				/* 操作模式 */`
		`uint32_t Priority				/* DMA通道优先级 */`
# 5.以DMA方式传输串口数据配置步骤
![Pasted image 20240122220615](../../../../uploads/stm32-hal-library/pasted-image-20240122220615.png)
# 6.关键函数
1.`#define __HAL_LINKDMA(__HANDLE__, __PPP_DMA_FIELD__, __DMA_HANDLE__)`
`__HANDLE__`：需要用到DMA的外设
`__PPP_DMA_FIELD_`:外设指向DMA的指针
- hdma[]
- TIM_DMA_ID_UPDATE      
TIM_DMA_ID_CC1         
TIM_DMA_ID_CC2         
TIM_DMA_ID_CC3         
TIM_DMA_ID_CC4         
TIM_DMA_ID_COMMUTATION 
TIM_DMA_ID_TRIGGER     
`__DMA_HANDLE__`：DMA控制结构体
# 7.示例代码
```
__HAL_RCC_DMA1_CLK_ENABLE();
uart1.dmatx.Instance=DMA1_Channel4;
uart1.dmarx.Init.Direction=DMA_MEMORY_TO_PERIPH;
uart1.dmarx.Init.MemDataAlignment=DMA_MDATAALIGN_BYTE;
uart1.dmarx.Init.PeriphDataAlignment=DMA_PDATAALIGN_BYTE;
uart1.dmarx.Init.MemInc=DMA_MINC_ENABLE;
uart1.dmarx.Init.PeriphInc=DMA_PINC_DISABLE;
uart1.dmarx.Init.Mode=DMA_NORMAL;
uart1.dmarx.Init.Priority=DMA_PRIORITY_MEDIUM;

__HAL_LINKDMA(huart,hdmatx, uart1.dmatx);
HAL_DMA_Init(&uart1.dmatx);

HAL_NVIC_SetPriority(DMA1_Channel4_IRQn,3,0);
HAL_NVIC_EnableIRQ(DMA1_Channel4_IRQn);

uart1.dmarx.Instance=DMA1_Channel5;
uart1.dmarx.Init.Direction=DMA_PERIPH_TO_MEMORY;
uart1.dmarx.Init.MemDataAlignment=DMA_MDATAALIGN_BYTE;
uart1.dmarx.Init.PeriphDataAlignment=DMA_PDATAALIGN_BYTE;
uart1.dmarx.Init.MemInc=DMA_MINC_ENABLE;
uart1.dmarx.Init.PeriphInc=DMA_PINC_DISABLE;
uart1.dmarx.Init.Mode=DMA_NORMAL;
uart1.dmarx.Init.Priority=DMA_PRIORITY_MEDIUM;

__HAL_LINKDMA(huart,hdmarx, uart1.dmarx);
HAL_DMA_Init(&uart1.dmarx);

HAL_NVIC_SetPriority(DMA1_Channel5_IRQn,3,0);
HAL_NVIC_EnableIRQ(DMA1_Channel5_IRQn);



void DMA1_Channel4_IRQHandler()
{
	HAL_DMA_IRQHandler(&uart1.dmatx);

}

void DMA1_Channel5_IRQHandler()
{
	HAL_DMA_IRQHandler(&uart1.dmarx);
}


```
