HID服务

1 HID设备描述符

为了把一个设备识别为HID类别，设备在定义描述符的时候必须遵守HID规范。HID设备必须定义HID描述符，此外，设备和主机间的通信是通过报告的形式来实现的，所以还必须定义报告描述符。HID描述符是关联于接口（而不是端点）的，所以设备不需要为每个端点都提供一个HID描述符。

2 报告描述符

报告描述符已 item 形式排列组合而成，无固定长度，用户可自定义长度以及每一 bit 的含义。item 类型分为三种：main，global 和 local，其中 main 类型由分为 5 种 tag：

- input item tag：表示从设备的一个或多个类似控制管道得到的数据。
- output item tag：表示发送给一个或多个类似控制管道的数据。
- feature item tag：表示设备的输入输出不面向最终用户。
- collection item tag：表示一个有意义的input，output 和 feature 的组合项目。
- end collection item tag：表示一个 collection 的终止。

一个报告描述符可能包含多个 main item，为准确描述来自一个控制管道的数据，一个报告描述符必须包含以下内容：

- input（output，feature）
- usage
- usage page
- logical minimum
- logical maximum
- report size
- report count

item的数据格式有两种，分别为短 item 和 长 item。

短 item 格式：占 3 个字节，后两个字节为数据。

| 字节0      |                                                              |
| ---------- | ------------------------------------------------------------ |
| bSize[0:1] | 0：0个字节                                                   |
| bType[2:3] | 0：main 1：global 2：local 3：保留                           |
| bTag[4:7]  | 8：input 9：output A：collection B：feature C：end collection |

举个例子：

```c
{0x05, 0x0c};
//0x05 即为 00000101，bSize = 1 代表数据长度为 1 个字节，bType = 1 代表 item 类型是 global
```

长 item 格式：

2.1 Usage Page

Consumer Page（0x0C）：All controls on the Consumer page are application-specific. That is, they affect a specific device, not the system as a whole.