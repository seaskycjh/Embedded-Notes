1 初始化

```c
void bt_l2cap_init(void) {
    struct bt_l2cap_fixed_chan chan = {
        .cid = BT_L2CAP_CID_LE_SIG,  //LE 信号通道，CID 为 0x0005
        .accept = l2cap_accept,  //初始化通道的操作函数
    };
    //注册固定通道，即将 chan 添加到 le_channels 链表
    bt_l2cap_le_fixed_chan_register(&chan);
}

int l2cap_accept(struct bt_conn *conn, struct bt_l2cap_chan *chan) {
    static struct bt_l2cap_chan_ops ops = {
		.connected = l2cap_connected,
		.disconnected = l2cap_disconnected,
		.recv = l2cap_recv,
	};
    //为每个通道分配一个 bt_l2cap 结构体
}

/* 蓝牙接收到数据后最终会在此函数中进行处理 */
void l2cap_recv(struct bt_conn *conn, struct net_buf *buf) {
    cid = sys_le16_to_cpu(hdr->cid);
    chan = bt_l2cap_le_lookup_rx_cid(conn, cid);  //根据 CID 获取对应通道
    switch(hdr->code) {
        case BT_L2CAP_LE_CONN_REQ:
            
    }
}

/* 创建连接时会调用此函数来建立 L2CAP Channel */
void bt_l2cap_connected(struct bt_conn *conn) {
    SYS_SLIST_FOR_EACH_CONTAINER(&le_channels, fchan, node) {
        fchan->accept(conn, &chan);  //调用初始化时注册的 accept 函数来初始化通道
        ch->rx.cid = fchan->cid;
        ch->tx.cid = fchan->cid;
        l2cap_chan_add(conn, chan, NULL);  //将通道添加到连接的通道链表中
    }
}
```

