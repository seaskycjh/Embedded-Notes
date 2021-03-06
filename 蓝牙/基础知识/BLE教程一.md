## BLE教程一

[TOC]

------

### 1 BLE技术特点

1.1 高可靠性

1.2 低成本、低功耗

1.3 快速启动，瞬间连接

1.4 传输距离极大提高

1.5 高安全性

### 2 蓝牙状态及连接过程

#### 2.1 蓝牙状态

蓝牙具有5种状态：

**待机状态**（standby）：没有连接任何设备，没有传输或发送数据。

**广播状态**（Advertising）：周期性广播状态。

**扫描状态**（Scanning）：主动寻找正在广播的设备。

**发起连接状态**（Initiating）：主动发起连接。

**连接状态**（Connected）：已经连接。

#### 2.2 蓝牙角色

主设备（Master）：

从设备（Slave）：

#### 2.3 蓝牙连接过程

| Master       | Slave        |
| ------------ | ------------ |
| 待机模式     | 待机模式     |
| 扫描模式     | 广播模式     |
| 扫描请求     | 扫描回应     |
| 连接请求     |              |
| 连接参数请求 |              |
|              | 参数更新请求 |
| 参数更新回应 |              |
| 建立连接     |              |
| 连接事件     |              |

#### 2.4 连接事件和参数

**连接事件**（Connection Events）：主设备和从设备建立连接后，所有数据通信都在连接事件中进行。每个连接事件都先由 Master 发起，再由 Slave 回复。

两个设备在切换信道后发送和接收数据成为一个连接事件。

连接参数：主要有三个，这三者间必须满足公式：超时时间 > (1+从机忽略) * (连接间隔)。

**连接间隔**（Connection interval）：BLE的两个设备的连接中使用调频机制，两个设备使用特定的信道发送和接收数据，过一段时间后再使用新的信道。是指一个连接事件的开始到下一个连接事件的开始的时间间隔。以 1.25ms 为单元，范围是 6~3200 即 7.5ms~4s 之间。

**从机忽略**（Slave Latency）：允许从设备在没有数据要发的情况下，跳过一定数目的连接事件（即在这些连接事件中不必回复主设备的包），这样就能更加省电。范围是 0~499。

**超时时间**（Supervision Timeout）：设定一个超时时间，如果 BLE 在这个时间内没有发生通信的话，就自动断开。单位是 10ms，范围是 10~3200 即 100ms~32s。

### 3 蓝牙广播协议

蓝牙广播相关参数有：

广播间隔：

广播类型：

自身地址类型：

定向地址类型：

定向地址：

广播信道：

广播过滤策略：

广播数据：

响应数据：

#### 3.1 广播类型

**可连接的非定向广播**（Connectable Undirected Event Type）：一种用途最广的广播类型，包括广播数据和扫描响应数据，表示当前设备可以接受其他任何设备的连接请求。这种广播可以在没有连接的情况下发出。

**可连接的定向广播**（Connectable Directed Event Type）：定向广播类型是为了尽可能快的建立连接，这种报文包含两个地址：广播者的地址和发起者的地址。发起者收到发给自己的定向广播报文之后，可以立即发送连接请求作为回应。

定向广播有特殊的时序要求，完整的广播事件必须每 3.75ms 重复一次，如此快的发送会使报文充满广播信道，从而导致该区域内的其他设备无法进行广播，因此定向广播不可以持续 1.28s      以上的时间。当使用定向广播时，设备不能被主动扫描。

**不可连接的非定向广播**（Non-connectable Undirected Event Type）：仅仅发送广播数据，而不想被扫描或连接。

**可扫描的非定向广播**（Scannable Undirected Event Type）：也称可发现广播，不能用于发起连接，但允许其他设备扫描该广播设备。

**注意**：定向和非定向针对的是广播的对象，可连接和不可连接是指是否接受连接请求，可扫描广播类型是指回应扫描请求。

