ATB110X相关问题收集

1 sample\bluetooth\peripheral_dats

在 hci.h 文件中，#define BT_GAP_INIT_CONN_INT_MIN 0x0018  /* 30 ms    */

2 ZS110A入门指南

DAPLINK

安装驱动 mbedWinSerial_16466。

串口连接 Tx Rx：串口线用不了，CH340串口线可以用。

Flash烧写：拷贝文件 ATB110X_SPI0.FLM，在项目文件夹下有。

3 BLE入门实例开发

![image-20200320155832137](C:\Users\chenjianhua\AppData\Roaming\Typora\typora-user-images\image-20200320155832137.png)

![image-20200320160023214](E:\Embedded-Notes\问题收集\image-20200320160023214.png)



![image-20200321112919480](E:\Embedded-Notes\问题收集\image-20200321112919480.png)



```c
.user_data = (&(struct _bt_gatt_ccc){
    .cfg = _cfg,
    .cfg_len = ARRAY_SIZE(_cfg),
    .cfg_changed = _cfg_changed,
})
```

