## BLE_Profile设计

[TOC]

------

1 概述

1.1 简介

BLE Profile 即为 GATT(Generic Attribute) profile，中文名通用 属性配置文件。GATT 定义了两种角色：GATT Server 和 GATT Client。GATT Server 是向 GATT Client 提供数据服务的设备，GATT Client 是从 GATT Server 读写应用数据的设备。

1.2 概述

Profile 由一个或多个 Service 组成。一个 Service 由 Characteristic 和对其他服务的 Reference 组成。Characteristic  包括 type、value、Properties 和 Permissions，还可能包括一个或多个 Descriptions。

接下来以 BAS 服务为例介绍 BLE_Profile 设计。

### 2 准备工作

#### 2.1 链接

- 下载 Profile 规格：https://www.bluetooth.com/specifications/gatt
- 查看 Service 服务：https://www.bluetooth.com/specifications/gatt/services
- 查看 Characteristic 特性：https://www.bluetooth.com/specifications/gatt/characteristics
- 查看 Descriptors 描述符：https://www.bluetooth.com/specifications/gatt/descriptors

#### 2.2 查看 Service

| Name                                                         | Uniform Type Identifier               | Assigned Number | Specification |
| ------------------------------------------------------------ | ------------------------------------- | --------------- | ------------- |
| [Battery Service](https://www.bluetooth.com/wp-content/uploads/Sitecore-Media-Library/Gatt/Xml/Services/org.bluetooth.service.battery_service.xml) | org.bluetooth.service.battery_service | 0x180F          | GSS           |

点击链接查看可知其 uuid 是 0x180F，仅有一个特性是 Battery Level；此特性有一个强制的读属性和可选的通知属性，有两个描述符，Characteristic Presentation Format 用于多服务实例，有一个强制读属性，Client Characteristic Configuration 用于通知或表示，有一个强制读属性和一个强制写属性。

#### 2.3 查看 Characteristic

| Name                                                         | Uniform Type Identifier                    | Assigned Number | Specification |
| ------------------------------------------------------------ | ------------------------------------------ | --------------- | ------------- |
| [Battery Level](https://www.bluetooth.com/wp-content/uploads/Sitecore-Media-Library/Gatt/Xml/Characteristics/org.bluetooth.characteristic.battery_level.xml) | org.bluetooth.characteristic.battery_level | 0x2A19          | GSS           |

点击链接查看可知其 uuid 是 0x2A19，值是 8 位无符号整数，最小值为 0，最大值为 100，101 ~ 255 保留。

#### 2.4 查看 Descriptors

| Name                                                         | Uniform Type Identifier                                      | Assigned Number | Specification |
| ------------------------------------------------------------ | ------------------------------------------------------------ | --------------- | ------------- |
| [Client Characteristic Configuration](https://www.bluetooth.com/wp-content/uploads/Sitecore-Media-Library/Gatt/Xml/Descriptors/org.bluetooth.descriptor.gatt.client_characteristic_configuration.xml) | org.bluetooth.descriptor.gatt.client_characteristic_configuration | 0x2902          | GSS           |

点击链接查看可知其 uuid 是 0x2902，值是 16 位数据类型，位 0 表示 Notifications 开启（置1）/禁止（清0），位 1 表示 Indications 开启（置1）/禁止（清0），其余位保留；最小值为 0，最大值为 3。

### 3 代码编写

#### 3.1 Server

GATT Server 一般为从角色，通过广播提供服务，GATT Client 通过扫描广播连接到 GATT Server，获取 GATT Server 提供的服务。一般过程为先蓝牙使能，使能成功后调用回调函数，在回调函数中初始化服务以及启动广播，然后注册连接回调函数，等待连接。

```c

/* 应用程序入口 */
void app_main(void){
    int err;
    /* 使能蓝牙 */
    err = bt_enable(bt_ready);
    if(err){
        printk("Bluetooth init failed (err %d)\n", err);
        return;
    }
    
    /* 注册连接回调函数 */
    bt_conn_cb_register(&conn_callbacks);
    
    while(1){
        k_sleep(MSEC_PER_SEC);
    }
}
```

#### 3.2 Profile

BLE Profile 编写大致都分为两个过程，声明服务和注册服务，特性的声明和操作函数包含在声明服务的过程中。

```c
/* 1 声明 BAS 服务 */
static struct bt_gatt_attr attrs[] = {
    /* 声明主服务为 BAS */
	BT_GATT_PRIMARY_SERVICE(BT_UUID_BAS),
    /* 声明特性 Battery Level，属性为读属性和通知属性 */
	BT_GATT_CHARACTERISTIC(BT_UUID_BAS_BATTERY_LEVEL,
			       BT_GATT_CHRC_READ | BT_GATT_CHRC_NOTIFY),
    /* 声明特性描述符，有读权限，读函数是 read_blvl，值是 battery */
	BT_GATT_DESCRIPTOR(BT_UUID_BAS_BATTERY_LEVEL, BT_GATT_PERM_READ,
			   read_blvl, NULL, &battery),
    /* 声明客户端特性描述符，客户端发起 ccc 写操作时，blvl_ccc_cfg_changed 被调用 */
	BT_GATT_CCC(blvl_ccc_cfg, blvl_ccc_cfg_changed),
};

/* 为 1 开启通知，为 0 关闭通知 */
static u8_t notify;
/* 根据客户端写入的值来开启/关闭通知 */
static void blvl_ccc_cfg_changed(const struct bt_gatt_attr *attr, u16_t value){
    notify = (value == BT_GATT_CCC_NOTIFY) ? 1 : 0;
}

/* 读取电量函数 */
static ssize_t read_blvl(struct bt_conn *conn, 
                         const struct bt_gatt_attr *attr,
                         void *buf, u16_t len, u16_t offset){
    const char *value = attr->user_data;
    return bt_gatt_attr_read(conn, attr, buf, len, offset, value, sizeof(*value));
}

/* 2 注册 BAS 服务 */
void bas_init(void){
    bt_gatt_service_register(&bas_svc);
}

/* 3 电量通知 */
void bas_notify(void){
    if(!notify) return;
    /* 模拟电池电量 */
    battery--;
    if(!battery) battery = 100;
    /* 发送通知 */
    bt_gatt_notify(NULL, &attrs[2], &battery, sizeof(battery));
}
```

