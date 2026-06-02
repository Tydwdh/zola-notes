+++
title = "SPI"
date = 2025-07-31
description = "整理 SPI 相关笔记。"

[taxonomies]
tags = ["stm32", "hal"]
+++
# 1.简介
SPI：串行外设设备接口（Serial Peripheral Interface），是一种高速的，全双工，同步的通信总线。
四根通信线：SCK（Serial Clock）、MOSI（Master Output Slave Input）、MISO（Master Input Slave Output）、SS（Slave Select）
支持总线挂载多设备（一主多从）
# 硬件电路
所有SPI设备的SCK、MOSI、MISO分别连在一起
主机另外引出多条SS控制线，分别接到各从机的SS引脚
输出引脚配置为推挽输出，输入引脚配置为浮空或上拉输入
![Pasted image 20240402165119](../../../../uploads/stm32-hal-library/pasted-image-20240402165119.png)
# SPI时序基本单元
起始条件：SS从高电平切换到低电平
![Pasted image 20240402165315](../../../../uploads/stm32-hal-library/pasted-image-20240402165315.png)
终止条件：SS从低电平切换到高电平
![Pasted image 20240402165330](../../../../uploads/stm32-hal-library/pasted-image-20240402165330.png)
交换一个字节（模式0）
CPOL=0：空闲状态时，SCK为低电平
CPHA=0：SCK第一个边沿移入数据，第二个边沿移出数据
![Pasted image 20240402165439](../../../../uploads/stm32-hal-library/pasted-image-20240402165439.png)
# 工作模式
![Pasted image 20240406175225](../../../../uploads/stm32-hal-library/pasted-image-20240406175225.png)
# 关键结构体
```c
SPI_HandleTypeDef
{
SPI_TypeDef  *Instance
SPI_InitTypeDef   Init
}


SPI_InitTypeDef
{
	uint32_t Mode				/* SPI模式（主机）  */
uint32_t Direction			/* 工作方式（全双工） */
uint32_t DataSize			/* 帧格式（8位） */
uint32_t CLKPolarity			/* 时钟极性（CPOL = 0） */
uint32_t CLKPhase			/* 时钟相位 （CPHA = 0）*/
uint32_t NSS				/* SS控制方式（软件） */
uint32_t BaudRatePrescaler		/* SPI波特率预分频值 */
uint32_t FirstBit				/* 数据传输顺序（MSB）*/
uint32_t TIMode				/* 帧格式：Motorola / TI  */
uint32_t CRCCalculation		/* 设置硬件CRC校验 */
uint32_t CRCPolynomial		/*  设置CRC校验多项式 */
…
}
```


# 示例代码
```c
void SPI1_Init(void)
{

    spi1.Instance = SPI1;                                  // 使用SPI1
    spi1.Init.Mode = SPI_MODE_MASTER;                      // 主机模式
    spi1.Init.Direction = SPI_DIRECTION_2LINES;            // 双线模式，全双工
    spi1.Init.DataSize = SPI_DATASIZE_8BIT;                // 8位数据模式
    spi1.Init.BaudRatePrescaler = SPI_BAUDRATEPRESCALER_2; // PI1是APB2的时钟源72Mhz，预分频器选择2->36Mhz
    spi1.Init.CLKPolarity = SPI_POLARITY_LOW;              // 时钟极性低
    spi1.Init.CLKPhase = SPI_PHASE_1EDGE;                  // 第1边沿采样  MODE 0
    spi1.Init.NSS = SPI_NSS_SOFT;                          // 软件NSS管脚
    spi1.Init.FirstBit = SPI_FIRSTBIT_MSB;                 // 高位先发
    spi1.Init.TIMode = SPI_TIMODE_DISABLE;                 // 不使用TI模式
    spi1.Init.CRCCalculation = SPI_CRCCALCULATION_DISABLE; // 不计算CRC
    HAL_SPI_Init(&spi1);
}
// SPI1底层初始化函数
void HAL_SPI_MspInit(SPI_HandleTypeDef *hspi)
{
    if (hspi->Instance == SPI1)
    {
        // PA4=SPI1_NSS,PA5=SPI1_SCK PA6=SPI1_MISO, PA7=SPI1_MOSI,
        __HAL_RCC_SPI1_CLK_ENABLE();
        __HAL_RCC_GPIOA_CLK_ENABLE();
        GPIO_InitTypeDef GPIO_Init;

        GPIO_Init.Pin = GPIO_PIN_7 | GPIO_PIN_5;
        GPIO_Init.Mode = GPIO_MODE_AF_PP;
        GPIO_Init.Pull = GPIO_NOPULL;
        GPIO_Init.Speed = GPIO_SPEED_FREQ_HIGH;

        HAL_GPIO_Init(GPIOA, &GPIO_Init);

        GPIO_Init.Pin = GPIO_PIN_6;
        GPIO_Init.Mode = GPIO_MODE_AF_INPUT;
        GPIO_Init.Pull = GPIO_NOPULL;
        HAL_GPIO_Init(GPIOA, &GPIO_Init);

        GPIO_Init.Pin = GPIO_PIN_4;
        GPIO_Init.Mode = GPIO_MODE_OUTPUT_PP;
        GPIO_Init.Pull = GPIO_PULLUP;
        GPIO_Init.Speed = GPIO_SPEED_FREQ_LOW;

        HAL_GPIO_Init(GPIOA, &GPIO_Init);
    }
    else if (hspi->Instance == SPI2)
    {
    }
}
```
