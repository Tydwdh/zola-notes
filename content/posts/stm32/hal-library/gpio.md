+++
title = "GPIO"
date = 2025-07-31
description = "整理 GPIO 相关笔记。"

[taxonomies]
tags = ["stm32", "hal"]
+++
# 1.GPIO简介
•GPIO（General Purpose Input Output）通用输入输出口

•可配置为8种输入输出模式

•引脚电平：0V~3.3V，部分引脚可容忍5V

•输出模式下可控制端口输出高低电平，用以驱动LED、控制蜂鸣器、模拟通信协议输出时序等

•输入模式下可读取端口的高低电平或电压，用于读取按键输入、外接模块电平信号输入、ADC电压采集、模拟通信协议接收数据等
•速度为2MHZ，10MHZ，50MHZ
# 2.GPIO基本结构
![Pasted image 20231107212808](../../../../uploads/stm32-hal-library/pasted-image-20231107212808.png)
# 3.GPIO位结构
![Pasted image 20231107212851](../../../../uploads/stm32-hal-library/pasted-image-20231107212851.png)
# 4.GPIO模式
| 模式名称                    | 性质   | 特征                                 |
| ----------------------- | ---- | ---------------------------------- |
| 浮空输入GPIO_NOPULL         | 数字输入 | 可读取引脚电平，若引脚悬空，则电平不确定               |
| 上拉输入GPIO_PULLUP         | 数字输入 | 可读取引脚电平，内部连接上拉电阻，悬空时默认高电平          |
| 下拉输入GPIO_PULLDOWN       | 数字输入 | 可读取引脚电平，内部连接下拉电阻，悬空时默认低电平          |
| 模拟输入                    | 模拟输入 | GPIO无效，引脚直接接入内部ADC(与上下拉模式共用一个位，冲突) |
| 开漏输出GPIO_MODE_OUTPUT_OD | 数字输出 | 可输出引脚电平，高电平为高阻态，低电平接VSS            |
| 推挽输出GPIO_MODE_OUTPUT_PP | 数字输出 | 可输出引脚电平，高电平接VDD，低电平接VSS            |
| 复用开漏输出GPIO_MODE_AF_OD   | 数字输出 | 由片上外设控制，高电平为高阻态，低电平接VSS            |
| 复用推挽输出GPIO_MODE_AF_PP   | 数字输出 | 由片上外设控制，高电平接VDD，低电平接VSS            |
# 5.配置步骤
1. 使能时钟
\_\_HAL_RCC_GPIOx_CLK_ENABLE()
2. 设置工作模式
   HAL_GPIO_Init()
3. 设置输出状态（可选）
4. 读取输入状态（可选）
# 6.相关hal库函数简介
1. `HAL_RCC_GPIOx_CLK_ENABLE();`：此函数用于开启GPIO时钟。
	
2. `void HAL_GPIO_Init(GPIO_TypeDef *GPIOx, GPIO_InitTypeDef *GPIO_Init);`：此函数用于初始化GPIOx指定的GPIO端口。GPIO_Init是一个指向GPIO_InitTypeDef结构的指针，该结构指定了要配置的特定引脚及其模式。
```c
typedef struct

{
	  uint32_t Pin;        /* 引脚号 */
	  uint32_t Mode;       /* 模式设置 */
	  uint32_t Pull;       /* 上拉下拉设置 */
	  uint32_t Speed;      /* 速度设置 */
} GPIO_InitTypeDef;
```
3. `void HAL_GPIO_DeInit(GPIO_TypeDef *GPIOx, uint32_t GPIO_Pin);`：此函数用于重置GPIOx指定的GPIO端口的所有用于GPIO_Pin指定的引脚的寄存器。
    
3. `GPIO_PinState HAL_GPIO_ReadPin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`：此函数用于读取GPIOx指定的GPIO端口上GPIO_Pin指定的引脚的状态。
    
4. `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);`：此函数用于设置GPIOx指定的GPIO端口上GPIO_Pin指定的引脚的状态为PinState。
    
5. `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`：此函数用于切换GPIOx指定的GPIO端口上GPIO_Pin指定的引脚的状态。
    
6. `HAL_StatusTypeDef HAL_GPIO_LockPin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin);`：此函数用于锁定GPIOx指定的GPIO端口上GPIO_Pin指定的引脚，一旦引脚被锁定，只有通过复位才能解锁。
    
7. `void HAL_GPIO_EXTI_IRQHandler(uint16_t GPIO_Pin);`：此函数用于处理GPIO_Pin指定的引脚的中断请求。
    
8. `void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin);`：此函数在发生GPIO_Pin指定的引脚的中断时被调用，它是一个回调函数，用户可以在此函数中添加自己的代码以响应中断。
# 7.代码示例
```c
void GPIO_Init(void)  // 定义一个函数，用于初始化GPIO
{
  // 定义一个GPIO_InitTypeDef类型的结构体变量GPIO_InitStruct
  GPIO_InitTypeDef GPIO_InitStruct ;  
  HAL_GPIO_WritePin(GPIOA,  GPIO_Pin_0, 0);
  // 使能GPIOA时钟
  __HAL_RCC_GPIOA_CLK_ENABLE();  
  // 设置要配置的GPIO脚为第0脚
  GPIO_InitStruct.Pin = GPIO_PIN_0;  
  // 设置GPIO模式为推挽输出
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;  
  // 设置上拉/下拉模式为不上拉也不下拉
  GPIO_InitStruct.Pull = GPIO_NOPULL;  
  // 设置GPIO速度为低速
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;  
  // 调用函数，使用GPIO_InitStruct中的配置参数初始化GPIOA的第0脚
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);  
}
```


# 8.stm32f1引脚定义
![STM32F103C8T6引脚定义](../../../../uploads/stm32-hal-library/stm32f103c8t6%E5%BC%95%E8%84%9A%E5%AE%9A%E4%B9%89.png)


