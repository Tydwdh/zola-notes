+++
title = "I2C"
date = 2025-07-31
description = "整理 I2C 相关笔记。"

[taxonomies]
tags = ["stm32", "hal"]
+++
# 1.简介
•I2C（Inter IC Bus）是由Philips公司开发的一种通用数据总线

•两根通信线：SCL（Serial Clock）、SDA（Serial Data）
SDA（Serial Data）：这是数据线，用于在设备间传输串行数据。所有连接到I2C总线上的设备的串行数据线SDA都接到总线的SDA上。

SCL（Serial Clock Line）：这是时钟线，用于产生同步时钟脉冲。所有连接到I2C总线上的设备的时钟线SCL都接到总线的SCL上。主设备负责控制通信，通过对数据传输进行初始化/终止化，来发送数据并产生所需的同步时钟脉冲。

•同步，半双工

•带数据应答

•支持总线挂载多设备（一主多从、多主多从）

# 2.硬件电路
•所有I2C设备的SCL连在一起，SDA连在一起

•设备的SCL和SDA均要配置成复用开漏输出模式 GPIO_MODE_AF_OD2.GPIO

•SCL和SDA各添加一个上拉电阻，阻值一般为4.7KΩ左右
![Pasted image 20231109133627](../../../../uploads/stm32-hal-library/pasted-image-20231109133627.png)
# 3.基本时序
- 起始条件：SCL高电平期间，SDA从高电平切换到低电平 
- 终止条件：SCL高电平期间，SDA从低电平切换到高电平
	![Pasted image 20231109134513](../../../../uploads/stm32-hal-library/pasted-image-20231109134513.png)
-  发送一个字节：SCL低电平期间，主机将数据位依次放到SDA线上（高位先行），然后释放SCL，从机将在SCL高电平期间读取数据位，所以SCL高电平期间SDA不允许有数据变化，依次循环上述过程8次，即可发送一个字节
	![Pasted image 20231109134220](../../../../uploads/stm32-hal-library/pasted-image-20231109134220.png)
- 接收一个字节：SCL低电平期间，从机将数据位依次放到SDA线上（高位先行），然后释放SCL，主机将在SCL高电平期间读取数据位，所以SCL高电平期间SDA不允许有数据变化，依次循环上述过程8次，即可接收一个字节（主机在接收之前，需要释放SDA）
	![Pasted image 20231109134416](../../../../uploads/stm32-hal-library/pasted-image-20231109134416.png)
- 发送应答：主机在接收完一个字节之后，在下一个时钟发送一位数据，数据0表示应答，数据1表示非应答
- 接收应答：主机在发送完一个字节之后，在下一个时钟接收一位数据，判断从机是否应答，数据0表示应答，数据1表示非应答（主机在接收之前，需要释放SDA,上拉电阻影响下SDA默认为高，而从机拉低SDA就是确认收到数据即ACK，否则NACK）
	![Pasted image 20231109134623](../../../../uploads/stm32-hal-library/pasted-image-20231109134623.png)


I2C总线通信时每个字节为8位长度，数据传送时，先传送最高位，后传送低位，发送器发送完一个字节数据后接收器必须发送1位应答位来回应发送器，即一帧共有9位。

I2C每次发送数据必须是8位。

MSB固定，先发高位，再发低位。
# 4.硬件外设简介
•STM32内部集成了硬件I2C收发电路，可以由硬件自动执行时钟生成、起始终止条件生成、应答位收发、数据收发等功能，减轻CPU的负担

•支持多主机模型

•支持7位/10位地址模式

•支持不同的通讯速度，标准速度(高达100 kHz)，快速(高达400 kHz)

•支持DMA

•兼容SMBus协议

•STM32F103C8T6 硬件I2C资源：I2C1、I2C2
![Pasted image 20231109135937](../../../../uploads/stm32-hal-library/pasted-image-20231109135937.png)
![Pasted image 20231109140001](../../../../uploads/stm32-hal-library/pasted-image-20231109140001.png)
![Pasted image 20231109140037](../../../../uploads/stm32-hal-library/pasted-image-20231109140037.png)
![Pasted image 20231109140046](../../../../uploads/stm32-hal-library/pasted-image-20231109140046.png)
# 配置步骤
![Pasted image 20240323210432](../../../../uploads/stm32-hal-library/pasted-image-20240323210432.png)
# 主要结构体
```c
typedef struct
{

  I2C_TypeDef                *Instance;   
  I2C_InitTypeDef            Init;   
  ...
 }I2C_HandleTypeDef
 typedef struct
{
  uint32_t ClockSpeed;       //指定时钟频率。

  uint32_t DutyCycle;        // 指定I2C快速模式的占空比。

  uint32_t OwnAddress1;      // 指定第一个设备自身地址。

  uint32_t AddressingMode;   // 指定选择7位还是10位寻址模式。

  uint32_t DualAddressMode;  // 指定是否选择双寻址模式。

  uint32_t OwnAddress2;      //如果选择了双寻址模式，指定第二个设备自身地址此参数只能是7位地址

  uint32_t GeneralCallMode;  // 指定是否选择广播呼叫模式。

  uint32_t NoStretchMode;    // 指定是否选择无拉伸模式。
} I2C_InitTypeDef;
   
 
```
