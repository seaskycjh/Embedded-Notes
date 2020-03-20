### 1 简介

### 2 基础知识

2.1 LWIP协议栈

### 2 ESP8266模块

#### 2.1 硬件连接

对于ESP-01S模块，只需连接3.3V、GND、TX、RX四个串口基本端口。上电瞬间蓝灯闪烁一次。

注意：不能使用USB转TTL的3.3V电源进行供电，可以使用2节干电池或经过LDO转换后的3.3V电源。

#### 2.2 常用AT指令

**注：设置指令可加后缀 \_CUR（设置后不保存到Flash）或 \_DEF（设置后保存到Flash），不加后缀时默认保存到Flash。**

**基础 AT 指令：**

```
AT-测试AT指令是否可用
响应：OK

AT+RST-重启模块
响应：OK

AT+GMR-查看固件版本信息
响应：<AT版本信息>
	 <SDK版本信息>
	 <编译生成时间>
	 
ATE0/1-关闭/开启回显
响应：OK

AT+RESTORE-恢复出厂设置
响应：OK
说明：擦除所有保存到Flash的参数，恢复默认参数

AT+UART-设置UART配置并保存到Flash
查询：AT+UART?
响应：+UART:<波特率>,<数据位>,<停止位>,<校验位>,<流控制>
	 OK
指令：AT+UART=<波特率>,<数据位>,<停止位>,<校验位>,<流控制>
响应：OK
```

**基础 WiFi 指令：**

```
AT+CWMODE-设置WiFi模式并保存到Flash
查询：AT+CWMODE?
响应：+CWMMODE:<mode>
	 OK
设置：AT+CWMODE=<mode>
响应：OK
参数：mode：1(STA模式) 2(AP模式) 3(AP+STA模式)

AT+CWLAP-扫描可用AP
响应：+CWLAP:<ecn>,<ssid>,<rssi>,<mac>,<channel>,<freq offset>,<freq cali>,			 <pairwise_cipher>,<group_cipher>,<bgn>,<wps>
	 OK
	 
AT+CWJAP-连接AP并保存到Flash
查询：AT+CWJAP?
响应：+CWJAP:<ssid>,<bssid>,<channel>,<rssi>
	 OK
设置：AT+CWJAP=<ssid>,<pwd>[,<bssid>][,<pci_en>]
成功：OK
失败：+CWJAP_DEF:<error code>
	 FAIL
参数：<ssid>：字符串参数，AP 的 SSID
	 <bssid>：AP 的 MAC 地址
	 <channel>：信道号
	 <rssi>：信号强度
	 <pwd>：密码（最⻓64字节）
	 <pci_en>：选填参数，不允许连接WEP和open的路由器，可⽤于PCI认证
	 
AT+CWQAP-断开与AP的连接
响应：OK
 
AT+CWSAP-配置AP参数并保存到Flash（模块为AP模式时才可用）
查询：AT+CWSAP?
响应：+CWSAP_DEF:<ssid>,<pwd>,<chl>,<ecn>,<max	conn>,<ssid	hidden>
设置：AT+CWSAP_DEF=<ssid>,<pwd>,<chl>,<ecn>[,<max conn>][,<ssid hidden>]
参数：<ssid>：接⼊点名称（字符串）
	 <pwd>：密码（字符串）
	 <chl>：通道号
     <ecn>：加密⽅式，0(OPEN) 2(WPA_PSK) 3(WPA2_PSK4) 4(WPA_WPA2_PSK)
	 <max conn>：选填参数，允许连⼊AP的STA数⽬，取值范围[1, 8]
	 <ssid hidden>：选填参数，0(⼴播SSID) 1(不⼴播SSID)
	 
AT+CWLIF-查询连接到本AP的STA信息
响应：+CWLIF:<ip addr>,<mac>
	 OK
	 
AT+CWAUTOCONN-上电是否自动连接AP
设置：AT+CWAUTOCONN=<enable>
响应：OK
参数：enable：0(上电不⾃动连接AP) 1(上电⾃动连接AP)
```

**基础 TCP/IP 指令：**

```
AT+CIPSTATUS-查询网络连接信息
响应：STATUS:<stat>
	 +CIPSTATUS:<link ID>,<type>,<remote IP>,<remote port>,<local port>,<tetype>
参数：<stat>：ESP8266 Station 接⼝的状态
	 2：已连接AP，获得IP地址
	 3：已建⽴TCP或UDP传输
	 4：断开⽹络连接
	 5：未连接AP
	 <link ID>：⽹络连接ID(0~4)，⽤于多连接的情况
	 <type>：字符串，"TCP" 或者 "UDP"
	 <remote IP>：字符串，远端 IP 地址
	 <remote port>：远端端⼝值
	 <local	port>：本地端⼝值
	 <tetype>：0(作为客户端) 1(作为服务器)

AT+CIFSR-查询本地IP地址
响应：+CIFSR:APIP,<SoftAP IP address>
	 +CIFSR:APMAC,<SoftAP MAC address>
	 +CIFSR:STAIP,<Station IP address>
	 +CIFSR:STAMAC,<Station MAC address>
	 OK
注意：需连上AP后，才可以查询

AT+CIPMUX-设置多连接
查询：AT+CIPMUX?
设置指令：AT+CIPMUX=<mode>
参数：<mode>：0-单连接模式 1-多连接模式

AT+CIPSERVER-建立TCP服务器
设置：AT+CIPSERVER=<mode>[,<port>]
响应：OK
参数：<mode>：0-关闭服务器 1-建⽴服务器
	 <port>：端⼝号，默认为 333
注意：多连接情况下，才能开启 TCP 服务器

AT+CIPSTART-建立TCP/UDP/SSL连接
单连接：AT+CIPSTART=<type>,<remote IP>,<remote port>[,<TCP keep	alive>]
多连接：AT+CIPSTART=<link ID>,<type>,<remote IP>,<remote port>[,<TCP keep alive>]
响应：成功：OK 失败：ERROR
	 连接已存在：ALREADY CONNECTED
参数：<link ID>：⽹络连接 ID (0 ~ 4)，⽤于多连接的情况
	 <type>：字符串参数，连接类型，"TCP"，"UDP"或"SSL"
	 <remote IP>：字符串参数，远端 IP 地址
	 <remote port>：远端端⼝号
	 <TCP keep alive>：TCP keep-alive 侦测时间，默认关闭此功能
	 0：关闭此功能
	 1~7200：侦测时间，单位为 1s
	 
AT+CIPMODE-设置传输模式
查询：AT+CIPMODE?
设置：AT+CIPMODE=<mode>
参数：<mode>：0-普通传输模式 1-透传模式，仅⽀持 TCP单连接和UDP固定通信对端的情况
	 
AT+CIPSEND-发送数据
单连接：AT+CIPSEND=<length>
多连接：AT+CIPSEND=<link ID>,<length>
UDP传输：AT+CIPSEND=[<link ID>,]<length>[,<remote IP>,<remote port>]
透传模式：AT+CIPSEND
响应：先换⾏返回 >，然后开始接收数据
	 未建立连接或连接断开返回 ERROR
	 数据发送成功返回 SEND OK
	 数据发送失败返回 SEND FAIL
参数：<link ID>：⽹络连接 ID 号 (0 ~ 4)，⽤于多连接的情况
	 <length>：数字参数，表明发送数据的⻓度，最⼤⻓度为 2048
	 <remote IP>]：UDP 传输可以设置对端 IP
	 <remote port>]：UDP 传输可以设置对端端⼝
	 
AT+CIPCLOSE-关闭TCP/UDP/SSL传输
多连接：AT+CIPCLOSE=<link ID>
参数：<link ID>：需要关闭的连接ID 号。当ID为5时，关闭所有连接

AT+PING-ping功能
设置：AT+PING=<IP>
响应：成功：+<time> OK
	 失败：+timeout ERROR
参数：<IP>：字符串参数，IP 地址
	 <time>：ping 响应时间
	 
AT+CIPDINFO—接收⽹络数据时是否提示对端 IP 和端⼝
设置：AT+CIPDINFO=<mode>
参数：<mode>：0-不显示对端 IP 和端⼝ 1-显示对端 IP 和端⼝

+IPD-接收网络数据
单连接：+IPD,<len>[,<remote	IP>,<remote	port>]:<data>
多连接：+IPD,<link ID>,<len>[,<remote IP>,<remote port>]:<data>
参数：<remote IP>：⽹络通信对端 IP，由指令 AT+CIPDINFO=1 使能显示
	 <remote port>：⽹络通信对端端⼝，由指令 AT+CIPDINFO=1 使能
	 <link ID>：收到⽹络连接的 ID 号
	 <len>：数据⻓度
	 <data>：收到的数据
```

2.2 TCP相关例程

**STA模式：**

ESP8266作为Client：

```
AT+CWMODE=1
AT+CWJAP="FAE","12345678"
AT+CIPSTART="TCP","192.168.0.105",8888
AT+CIPSEND=5
```

ESP8266作为Sever：

```
AT+CWMODE=1
AT+CWJAP="FAE","12345678"
AT+CIPMUX=1
AT+CIPSERVER=1,8888
AT+CIFSR
AT+CIPSEND=0,5
```

**AP模式：**

ESP8266开启WIFI热点

ESP8266作为Client：

```
AT+CWMODE=2
AT+CWSAP="ESP8266","12345678",11,3
AT+CIPSTART="TCP","192.168.4.2",8888
AT+CIPSEND=5
```

ESP8266 Server：

```

```

