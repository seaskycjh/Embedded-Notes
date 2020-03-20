## SPI协议

[TOC]

### 1 简介

#### 1.1 概述

SPI 是串行外围设备接口。SPI 接口主要应用在 EEPROM，FLASH，实时时钟，AD 转换器，数字信号处理器和数字信号解码器之间。是一种**串行同步全双工**通信方式。

SPI 接口一般使用 4 条线通信：

MISO：主设备数据输入，从设备数据输出。

MOSI：主设备数据输出，从设备数据输入。

SCLK：时钟信号，由主设备产生。

CS：从设备片选信号，由主设备控制。

#### 1.2 特点

- 可以同时发出和接收串行数据。
- 可以当作主机或从机工作。
- 提供频率可编程时钟。
- 发送结束中断标志。
- 写冲突保护，总线竞争保护等。

### 2 协议

时钟极性（CPOL）对传输协议没有重大的影响。如果 CPOL=0，串行同步时钟的空闲状态为低电平；如果 CPOL=1，串行同步时钟的空闲状态为高电平。

时钟相位（CPHA）能够配置用于选择两种不同的传输协议之一进行数据传输。如果 CPHA=0，在串行同步时钟的第一个跳变沿（上升或下降）数据被采样；如果 CPHA=1，在串 行同步时钟的第二个跳变沿（上升或下降）数据被采样。

### 3 实现

```c
/* 配置相关引脚 */
/* 初始化SPI，设置SPI工作模式 */
typedef struct{
    uint16_t SPI_Direction;  //设置通信方式
    uint16_t SPI_Mode;  //设置主从模式
    uint16_t SPI_DataSize;  //8/16位帧格式选择
    uint16_t SPI_CPOL;  //设置时钟极性
    uint16_t SPI_CPHA;  //设置时钟相位
    uint16_t SPI_NSS;  //设置 NSS 信号由硬件（NSS管脚）还是软件控制
    uint16_t SPI_BaudRatePrescaler;  //设置SPI波特率预分频值（SPI的时钟的参数）
    uint16_t SPI_FirstBit;  //设置数据传输顺序是MSB位在前还是LSB位在前
    uint16_t SPI_CRCPolynomial;  //设置 CRC 校验多项式，
}SPI_InitTypeDef;
```

