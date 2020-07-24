1 初始化

```c
/* ATT 层初始化，注册 ATT 通道 */
void bt_att_init(void) {
    struct bt_l2cap_fixed_chan chan = {
        .cid = BT_L2CAP_CID_ATT,  //ATT 固定通道，CID 为0x0004
        .accept = bt_att_accept,
    };
    bt_l2cap_le_fixed_chan_register(&chan);  //将通道添加到通道链表
    bt_gatt_init();
}

/* 这里定义了一个 handlers 数组，用来处理接收到的 ATT 数据包 */
static const struct att_handler {
    u8_t op;  //ATT 操作码
    u8_t expect_len;
    att_type_t type;  //ATT 数据包类型
    u8_t (*func)(struct bt_att *att, struct net_buf *buf);  //处理函数
} handlers[] = {
    {BT_ATT_OP_READ_REQ,
     sizeof(struct bt_att_read_req),
     ATT_REQUEST, att_read_req},
    {...},
};

int bt_att_accept(...) {
    struct bt_l2cap_chan_ops ops = {
		.connected = bt_att_connected,
		.disconnected = bt_att_disconnected,
		.recv = bt_att_recv,
    };
}

u8_t read_cb()
```

2 数据接收

3 数据发送

