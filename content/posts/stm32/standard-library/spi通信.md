+++
title = "SPI通信"
date = 2026-05-19
description = "整理 SPI通信 相关笔记。"

[taxonomies]
tags = ["stm32", "standard-library"]
+++
# SPI通信

SPI（Serial Peripheral Interface）是一种同步串行数据总线，通常有一个主设备和一个或多个从设备。SPI使用以下四条线进行通信：

- **SCK (Serial Clock)**：这是由主设备驱动的串行时钟。它用于同步主设备和从设备之间的数据传输。

- **MOSI (Master Output, Slave Input)**：这是从主设备向外设发送的数据线。当主设备需要向从设备发送数据时，数据会通过此线路发送。

- **MISO (Master Input, Slave Output)**：这是从外设向主设备发送数据的线。当从设备需要向主设备发送数据时，数据会通过此线路发送。

- **SS (Slave Select)**：这是外设选择线，由主设备发送，以控制与哪个从设备通信。此线输入0代表选取，输入1代表未选取。


这种通信方式允许全双工操作，即可以同时进行数据的发送和接收。

# 硬件电路

•所有SPI设备的SCK、MOSI、MISO分别连在一起

•主机另外引出多条SS控制线，分别接到各从机的SS引脚

•输出引脚配置为推挽输出，输入引脚配置为浮空或上拉输入
![Pasted image 20231028155631](../../../../uploads/stm32-standard-library/pasted-image-20231028155631.png)
![Pasted image 20231028155649](../../../../uploads/stm32-standard-library/pasted-image-20231028155649.png)
SPI通信的本质是一个字节的交换

# SPI时序基本单元

•起始条件：SS从高电平切换到低电平

    ![Pasted image 20231028155724](../../../../uploads/stm32-standard-library/pasted-image-20231028155724.png)

•终止条件：SS从低电平切换到高电平

    ![Pasted image 20231028155747](../../../../uploads/stm32-standard-library/pasted-image-20231028155747.png)
# SPI通信模式

SPI通信有四种模式，这四种模式由时钟极性（CPOL）和时钟相位（CPHA）的不同配置形成。具体如下：

- **模式0 (CPOL=0; CPHA=0)**：时钟空闲状态为低电平，数据在上升沿采样，并在下降沿移出。（主要使用）
    
	![Pasted image 20231028155814](../../../../uploads/stm32-standard-library/pasted-image-20231028155814.png)
    
- **模式1 (CPOL=0; CPHA=1)**：时钟空闲状态为低电平，数据在下降沿采样，并在上升沿移出。
    
    ![Pasted image 20231028155919](../../../../uploads/stm32-standard-library/pasted-image-20231028155919.png)
    
- **模式2 (CPOL=1; CPHA=0)**：时钟空闲状态为高电平，数据在上升沿采样，并在下降沿移出。
    
    ![Pasted image 20231028155933](../../../../uploads/stm32-standard-library/pasted-image-20231028155933.png)
    
- **模式3 (CPOL=1; CPHA=1)**：时钟空闲状态为高电平，数据在下降沿采样，并在上升沿移出。
    
    ![Pasted image 20231028155946](../../../../uploads/stm32-standard-library/pasted-image-20231028155946.png)
