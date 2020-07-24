1 Hci_driver.c

```c
static int hci_driver_send(struct net_buf *buf)
{
    u8_t type = bt_buf_get_type(buf);  //得到缓冲数据类型
    switch(type){
        case BT_BUF_ACL_OUT:
            err = acl_handle(buf);  //发送到外部的数据
        case BT_BUF_CMD:
            err = cmd_handle(buf);  //发送到 controller 的命令
    }
}

static int acl_handle(struct net_buf *buf)
{
    err = hci_acl_handle(buf, &evt);
}

int hci_acl_handle(struct net_buf *, struct net_buf **)
{
    acl = (void *)buf->data;
    radio_pdu_node_tx = radio_tx_mem_acquire();
    pdu_data = (struct pdu_data *)radio_pdu_node_tx->pdu_data;
}
```



数据接收：打开 controller 设备的时候创建了两个线程用于接收 BLE 数据。

```c
static struct {
    struct device *hf_clodk;
    
    /* 控制器到主机的事件数据队列 */
    void *link_rx_head;
    void *volatile link_rx_tail;
}

static void prio_recv_thread(...) {
    while(1) {
        while(radio_rx_get(&node_rx, &handle)){
            buf = net_buf_alloc(&hci_rx_pool, K_FOREVER);
            net_buf_reserve(buf, CONFIG_BT_HCI_RESERVE);
            bt_buf_set_type(buf, BT_BUF_EVT);
            bt_recv_prio(buf);
            if(node_rx){
                radio_rx_dequeue();
                k_fifo_put(&recv_fifo, node_rx);
                continue;
            }
            /* 等待数据的到来 */
            k_sem_take(&sem_prio_recv, K_FOREVER);
        }
    }
}

u8_t radio_rx_get() {
    
}

static void recv_thread(...) {
    while(1) {
        int err = k_poll(events, 2, K_FOREVER);
        if(events[1].state == K_POLL_STATE_FIFO_DATA_AVAILABLE){
            node_rx = k_fifo_get(events[1].fifo, 0);
        }
        process_hbuf(node_rx);
        bt_recv(buf);
    }
}

struct net_buf *process_hbuf(...) {
    
}

int bt_recv(...) {
    switch(bt_buf_get_type(buf)) {
        case BT_BUF_ACL_IN:
            hci_acl(buf);
        case BT_BUF_EVT:
            hci_event(buf);
    }
}

static void hci_acl(...) {
    conn = bt_conn_lookup_handle(acl(buf)->handle);
    
}


```

