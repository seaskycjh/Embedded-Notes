## IIC协议

[TOC]

### 1 简介

#### 1.1 概述

IIC总线是一种由 PHILIPS 公司开发的两线式串行总线，用于连接微控制器及其外围设备。它是由数据线 SDA 和时钟 SCL 构成的串行总线，可发送和接收数据。是**串行同步半双工通信**。

#### 1.2 特点

- 从设备地址为7bit，所以一条IIC总线上最多可以接2的7次方 = 128个设备。
- 标准模式下传输速率为100kbit/s， 快速模式下传输速率为400kbit/s，高速模式下为3.4Mbit/s。
- IO必须被配置为开漏输出，这是为了实现**线与**，挂载多个设备。

### 2 协议

#### 2.1 信号

IIC 总线上可以接多个从设备，每个从设备都有一个自己的地址（addr）。地址是 7 位的，是由芯片决定的，不同芯片的地址一般不同。如果进行广播的话，可将地址设为 0x00。

**起始信号**：SCL 为高电平时，SDA 由高电平向低电平跳变，开始传送数据。

**结束信号**：SCL 为高电平时，SDA 由低电平向高电平跳变，结束传送数据。

**数据传输**：SDA上的数据必须在SCL为高电平时稳定，在SCL为低电平时改变（传输数据）。

**应答信号**：处理器发送完8bit数据后，将 SDA 配置为输入，因为IIC外接上拉电阻，所以这时候SDA为高电平，接收数据的 IC 在接收到 8bit 数据后，向处理器发出特定的低电平脉冲， 表示已收到数据。

#### 2.2 数据传输

IIC 传输数据是大端传输，每次传输一个字节（8bit）。Master 每发送完 8bit 数据后将 SDA 设置为输入，此时 Slave 必须发送 ACK（即将 SDA 拉低），若没有 ACK，SDA 会被置高，这会引起 IIC 传输停止或重启。

IIC 写寄存器：

```
Master 发送开始信号
Master 发送从设备地址（7bit）和写操作 0（1bit），等待 ACK
Slave 发送 ACK
Master 发送寄存器地址（8bit），等待 ACK
Slave 发送 ACK
Master 发送要写入的数据（8bit），等待 ACK
Slave 发送 ACK
第 6、7 步可重复多次，即顺序写多个寄存器
Master 发送结束信号
```

I2C 读寄存器：

```
Master 发送从设备地址（7bit）和写操作 1（1bit），等待 ACK
Slave 发送 ACK
Master 发送寄存器地址（8bit），等待 ACK
Slave 发送 ACK
Master 发送开始信号
Master 发送从设备地址（7bit）和读操作 1（1bit），等待 ACK
Slave 发送 ACK
Slave 发送寄存器中的数据（8bit）
Master 发送 ACK
第 8、9 步可重复多次，即顺序读多个寄存器
```



3 实现

```c
//IIC初始化
void IIC_Init(void){
    /* 初始化两个I/O口为推挽输出，并输出高电平 */
}
//产生起始信号
void IIC_Start(void){
    SDA_OUT();  //设置I/O口为输出模式
    IIC_SDA = 1;
    IIC_SCL = 1;
    delay_us(4);
    IIC_SDA = 0;  //发送起始信号
    delay_us(4);
    IIC_SCL = 0;  //准备发送或接收数据
}
//产生停止信号
void IIC_Stop(void){
    SDA_OUT();  //设置I/O口为输出模式
    IIC_SCL = 0;
    IIC_SDA = 0;
    delay_us(4);
    IIC_SCL = 1;
    IIC_SDA = 1;//发送结束信号
    delay_us(4);
}
//等待应答信号，接收应答成功返回1，否则返回0
int IIC_Wait_Ack(void){
    int t;
    SDA_IN();  //设置I/O口为输入模式
    IIC_SDA = 1;
    delay_us(1);
    IIC_SCL = 1;
    delay_us(1);
    //当READ_SDA=0时表示接收到应答信号
    while(READ_SDA){
        if(t++ > 250){  //接收超时
            IIC_Stop();
            return 1;
        }
    }
    IIC_SCL = 0;  //时钟线输出0
    return 0;
}
//发送一个字节
void IIC_Send_Byte(char txd){
    int i;
    SDA_OUT();
    IIC_SCL = 0;  //拉低时钟线开始数据传输
    for(i = 0; i < 8; i++){
        IIC_SDA = (txd & 0x80) >> 7;  //先发送高位
        txd <<= 1;
        delay_us(2);
    }
}
//读取一个字节
char IIC_Read_Byte(char ack){
    unsigned char i, recv = 0;
    SDA_IN();
    for(i=0;i<8;i++ ){
        IIC_SCL=0;
        delay_us(2);
        IIC_SCL=1;
        receive <<= 1;
        if(READ_SDA) recv++;  //从高位开始接收
        delay_us(1);
    }
    if (!ack) IIC_NAck();   //不发生应答信号
    else IIC_Ack();    //发送应答信号
    return recv;
}

//发送一个指令
void IIC_Write_Cmd(char cmd){
    IIC_Start();
    IIC_Send_Byte(0x78);	//从设备地址为高7位，即 0x78>>1 = 0x3c
    IIC_Wait_Ack();
    IIC_Send_Byte(0x00);	//0x00 为命令寄存器地址
    IIC_Wait_Ack();
    IIC_Send_Byte(cmd);
    IIC_Wait_Ack();
    IIC_Stop();
}

```

