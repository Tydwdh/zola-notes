+++
title = "USART"
date = 2026-05-19
description = "整理 USART 相关笔记。"

[taxonomies]
tags = ["stm32", "hal"]
+++
# 1.简介
•USART（Universal Synchronous/Asynchronous Receiver/Transmitter）通用同步/异步收发器

•USART是STM32内部集成的硬件外设，可根据数据寄存器的一个字节数据自动生成数据帧时序，从TX引脚发送出去，也可自动接收RX引脚的数据帧时序，拼接为一个字节数据，存放在数据寄存器里

•自带波特率发生器，最高达4.5Mbits/s

•可配置数据位长度（8/9）、停止位长度（0.5/1/1.5/2）

•可选校验位（无校验/奇校验/偶校验）

•支持同步模式、硬件流控制、DMA、智能卡、IrDA、LIN

•STM32F103C8T6 USART资源： USART1、 USART2、 USART3

# 2.USART框图
![Pasted image 20231109213931](../../../../uploads/stm32-hal-library/pasted-image-20231109213931.png)
![Pasted image 20231109213940](../../../../uploads/stm32-hal-library/pasted-image-20231109213940.png)
# 3.硬件电路
![Pasted image 20231109214315](../../../../uploads/stm32-hal-library/pasted-image-20231109214315.png)
TX和RX交叉连结
# 4.串口参数及时序
•波特率：串口通信的速率

•起始位：标志一个数据帧的开始，固定为低电平

•数据位：数据帧的有效载荷，1为高电平，0为低电平，低位先行

•校验位：用于数据验证，根据数据位计算得来

•停止位：用于数据帧间隔，固定为高电平

![Pasted image 20231109214346](../../../../uploads/stm32-hal-library/pasted-image-20231109214346.png)
![Pasted image 20231109214351](../../../../uploads/stm32-hal-library/pasted-image-20231109214351.png)
# 5.配置步骤
![Pasted image 20240106180654](../../../../uploads/stm32-hal-library/pasted-image-20240106180654.png)
# 6.代码示例
```c
// 定义一个UART_HandleTypeDef类型的结构体变量uart
UART_HandleTypeDef uart;

// USART初始化函数，参数为波特率
void USART_Init(uint32_t bandrate)
{
	// 设置USART实例为USART1
	uart.Instance=USART1;
	// 设置波特率
	uart.Init.BaudRate=bandrate;
	// 设置数据位长度为8位
	uart.Init.WordLength=UART_WORDLENGTH_8B;
	// 设置停止位为1位
	uart.Init.StopBits=UART_STOPBITS_1;
	// 设置无奇偶校验
	uart.Init.Parity=UART_PARITY_NONE;
	// 设置模式为收发模式
	uart.Init.Mode=UART_MODE_TX_RX;
	// 设置无硬件流控制
	uart.Init.HwFlowCtl=UART_HWCONTROL_NONE;
	// 设置过采样为16倍过采样
	uart.Init.OverSampling=UART_OVERSAMPLING_16;
	
	// 调用HAL库函数，初始化uart
	HAL_UART_Init(&uart);
}

// HAL库函数，用于初始化UART硬件
void HAL_UART_MspInit(UART_HandleTypeDef *huart)
{
	// 判断是否为USART1
	if(huart->Instance==USART1)
	{
		// 使能GPIOA时钟
		__HAL_RCC_GPIOA_CLK_ENABLE();
		// 使能USART1时钟
		__HAL_RCC_USART1_CLK_ENABLE();
		// 定义一个GPIO_InitTypeDef类型的结构体变量
		GPIO_InitTypeDef GPIO_InitStruct;
			
		// 将GPIOA的9和10引脚设置为高电平
		HAL_GPIO_WritePin(GPIOA,GPIO_PIN_9|GPIO_PIN_10,1);
			
		// 设置GPIO模式为复用推挽输出
		GPIO_InitStruct.Mode=GPIO_MODE_AF_PP;
		// 设置引脚为9号引脚
		GPIO_InitStruct.Pin=GPIO_PIN_9;
		// 设置无上拉下拉
		GPIO_InitStruct.Pull=GPIO_NOPULL;
		// 设置GPIO速度为高速
		GPIO_InitStruct.Speed=GPIO_SPEED_FREQ_HIGH;
		
		// 初始化GPIOA
		HAL_GPIO_Init(GPIOA,&GPIO_InitStruct);
			
		// 设置GPIO模式为复用输入
		GPIO_InitStruct.Mode=GPIO_MODE_AF_INPUT;
		// 设置引脚为10号引脚
		GPIO_InitStruct.Pin=GPIO_PIN_10;
		// 设置无上拉下拉
		GPIO_InitStruct.Pull=GPIO_NOPULL;
			
		// 初始化GPIOA
		HAL_GPIO_Init(GPIOA,&GPIO_InitStruct);
	}
}

```
# 7.三种接收方式
- 轮询接收(效率低)
	```c
		while(1)
	{
		switch(HAL_UART_Receive(&uart,Buff,RX_SIZE,200))
		{
			case HAL_OK:  
				HAL_UART_Transmit(&uart,Buff,RX_SIZE,200);
				break;
			
			case HAL_TIMEOUT: 
				if(uart.RxXferCount != RX_SIZE-1)
				{
					HAL_UART_Transmit(&uart,Buff,RX_SIZE-1-uart.RxXferCount,200);
				}
			break;
		}
	}
```
- 多指针缓存
```c
UCB uart1;  // 定义一个 UCB 结构体的实例
UART_HandleTypeDef uart;  // 定义一个 UART 的句柄
uint8_t rx_state;  // 定义一个变量来保存 UART 的接收状态

uint8_t TXBUFF [TX_SIZE];  // 定义一个数组来保存要发送的数据
uint8_t RXBUFF [RX_SIZE];  // 定义一个数组来保存接收到的数据

// 初始化 UART 的状态和数据缓冲区的信息
void USART_PTR_Init(UCB *uartb)
{
	uartb->RxInPtr = &uartb->RXLocation[0];
	uartb->RxOutPtr =&uartb->RXLocation[0];
	uartb->RxEndPtr =&uartb->RXLocation[9];
	uartb->RX_Count=0;
	uartb->RxInPtr->start = RXBUFF;
	uartb->TxInPtr = &uartb->TXLocation[0];
	uartb->TxOutPtr =&uartb->TXLocation[0];
	uartb->TxEndtr =&uartb->TXLocation[9];
	uartb->TX_Count=0;
	uartb->TxInPtr->start = TXBUFF;
	
	__HAL_UART_ENABLE_IT(&uartb->uart, UART_IT_IDLE);  // 启用 UART 的空闲中断
	HAL_UART_Receive_IT(&uartb->uart,uartb->RxInPtr->start,RX_MAX_SIZE);  // 启动 UART 的中断接收
}

// 发送数据
void USART_TXData(UCB *uartb,uint8_t *data,uint32_t data_len)
{
	if((TX_SIZE - uartb->TX_Count)>=data_len)
	{
		uartb->TxInPtr->start=&TXBUFF[uartb->TX_Count];
	}else
	{
		uartb->TX_Count=0;
		uartb->TxInPtr->start = TXBUFF;
	}
	memcpy(uartb->TxInPtr->start,data,data_len);
	uartb->TX_Count +=data_len;
	uartb->TxInPtr->end=&TXBUFF[uartb->TX_Count-1];
	if(++uartb->TxInPtr==uartb->TxEndtr)
	{
		uartb->TxInPtr=&uartb->TXLocation[0];
	}
}

// 初始化 UART
void USART_Init(uint32_t bandrate)
{
	uart1.uart.Instance=USART1;
	uart1.uart.Init.BaudRate=bandrate;
	uart1.uart.Init.WordLength=UART_WORDLENGTH_8B;
	uart1.uart.Init.StopBits=UART_STOPBITS_1;
	uart1.uart.Init.Parity=UART_PARITY_NONE;
	uart1.uart.Init.Mode=UART_MODE_TX_RX;
	uart1.uart.Init.HwFlowCtl=UART_HWCONTROL_NONE;
	uart1.uart.Init.OverSampling=UART_OVERSAMPLING_16;
	
	HAL_UART_Init(&uart1.uart);  // 初始化 UART
	
	USART_PTR_Init(&uart1);  // 初始化 UART 的状态和数据缓冲区的信息
}

// 初始化 UART 的硬件资源
void HAL_UART_MspInit(UART_HandleTypeDef *huart)
{
	if(huart->Instance==USART1)
	{
		__HAL_RCC_GPIOA_CLK_ENABLE();  // 启用 GPIOA 的时钟
		__HAL_RCC_USART1_CLK_ENABLE();  // 启用 USART1 的时钟
		GPIO_InitTypeDef GPIO_InitStruct;  // 定义一个 GPIO 的初始化结构体
			
		HAL_GPIO_WritePin(GPIOA,GPIO_PIN_9|GPIO_PIN_10,GPIO_PIN_SET);  // 设置 GPIOA 的 9 和 10 引脚为高电平
		GPIO_InitStruct.Mode=GPIO_MODE_AF_PP;  // 设置 GPIO 的模式为复用推挽输出
		GPIO_InitStruct.Pin=GPIO_PIN_9;  // 设置 GPIO 的引脚为 9
		GPIO_InitStruct.Pull=GPIO_NOPULL;  // 设置 GPIO 的上拉/下拉模式为无上拉/下拉
		GPIO_InitStruct.Speed=GPIO_SPEED_FREQ_HIGH;  // 设置 GPIO 的速度为高速
		
		HAL_GPIO_Init(GPIOA,&GPIO_InitStruct);  // 初始化 GPIOA 的 9 引脚
			
		GPIO_InitStruct.Mode=GPIO_MODE_AF_INPUT;  // 设置 GPIO 的模式为复用输入
		GPIO_InitStruct.Pin=GPIO_PIN_10;  // 设置 GPIO 的引脚为 10
			
		HAL_GPIO_Init(GPIOA,&GPIO_InitStruct);  // 初始化 GPIOA 的 10 引脚
		
		HAL_NVIC_SetPriority(USART1_IRQn,3,0);  // 设置 USART1 中断的优先级
		HAL_NVIC_EnableIRQ(USART1_IRQn);  // 启用 USART1 的中断
	}
}

// UART 接收完成的回调函数
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
	if(huart->Instance<mark>USART1)
	{
	
	}
}

// UART 发生错误的回调函数
void HAL_UART_ErrorCallback(UART_HandleTypeDef *huart)
{
	if(huart->Instance</mark>USART1)
	{
	
	}
}

// UART 发送完成的回调函数
void HAL_UART_TxCpltCallback(UART_HandleTypeDef *huart)
{
	if(huart->Instance==USART1)
	{
		uart1.TXState=0;  // 设置 UART 的发送状态为 0
	}
}

// UART 接收被中止的回调函数
void HAL_UART_AbortReceiveCpltCallback(UART_HandleTypeDef *huart)
{
	if(huart->Instance==USART1)
	{
		uart1.RxInPtr->end = &RXBUFF[uart1.RX_Count - 1];  // 设置接收数据缓冲区的结束位置
		
		if(++uart1.RxInPtr==uart1.RxEndPtr)  // 如果接收数据缓冲区的输入指针等于结束指针
		{
			uart1.RxInPtr = &uart1.RXLocation[0];  // 将接收数据缓冲区的输入指针设置为开始位置
		}
		if(RX_SIZE-uart1.RX_Count<RX_MAX_SIZE)  // 如果接收到的数据的数量小于最大接收数量
		{
			uart1.RX_Count=0;  // 将接收到的数据的数量设置为 0
			uart1.RxInPtr->start=RXBUFF;  // 将接收数据缓冲区的开始位置设置为 RXBUFF
		}else
		{
			uart1.RxInPtr->start = &RXBUFF[uart1.RX_Count];  // 将接收数据缓冲区的开始位置设置为接收到的数据的数量
		}
		HAL_UART_Receive_IT(&uart1.uart,uart1.RxInPtr->start,RX_MAX_SIZE);  // 启动 UART 的中断接收
	}
}



while(1)
	{
		if(uart1.RxOutPtr!=uart1.RxInPtr)
		{
			USART_TXData(&uart1,uart1.RxOutPtr->start,uart1.RxOutPtr->end-uart1.RxOutPtr->start+1);
			if(++uart1.RxOutPtr==uart1.RxEndPtr)
			{
				uart1.RxOutPtr=&uart1.RXLocation[0];
			}
		}
		if((uart1.TxOutPtr != uart1.TxInPtr)&&(uart1.TXState==0))
		{
			uart1.TXState=1;
			HAL_UART_Transmit_IT(&uart1.uart,uart1.TxOutPtr->start,uart1.TxOutPtr->end-uart1.TxOutPtr->start+1);
			if(++uart1.TxOutPtr==uart1.TxEndtr)
			{
				uart1.TxOutPtr=&uart1.TXLocation[0];
			}
		}
		
	}
```
- DMA
```c

```





# 莫名BUG
## HAL_UARTEx_ReceiveToIdle_DMA
size需要两倍空间
