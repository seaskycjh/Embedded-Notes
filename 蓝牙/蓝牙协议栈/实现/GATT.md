GATT

gatt 初始化：

```c
int bt_gatt_service_register(struct bt_gatt_service *svc)
{
    //判断参数是否有效
    bt_gatt_init();	//初始化 GATT 核心服务
    //不允许再次注册强制性服务（GAP 和 GATT）
    gatt_register(svc);	//注册服务
    db_changed();
    return 0;
}

void bt_gatt_init(void)
{
    atomic_cas(&init, 0, 1);  //判断是否已经初始化
    //如果使能了 GATT 缓存
    k_delayed_work_init(&db_hash_work, db_hash_process);
    k_delayed_work_submit(&db_hash_work, DB_HASH_TIMEOUT);
    //如果使能了 GATT 改变
    k_delayed_work_init(&gatt_sc.work, sc_process);
    k_delayed_work_init(&gatt_ccc_store.work, ccc_delayed_store);
}

//采用链表的尾插法
static int gatt_register(struct bt_gatt_service *svc)
{
    gatt_insert(svc, last_handle);
}

//始终根据 handle 升序来插入服务节点
static void gatt_insert(struct bt_gatt_service *svc, u16_t last_handle)
{
}

```



`bt_gatt_notify()` 函数实现：将数据发送到外部连接的设备，需要经过 ATT 和 L2CAP 层的封装，最后再由 LL 层生成 PDU 并发送出去。

```c
/* 定义缓冲池，缓冲数量为 3，缓冲长度为 65+4=69 字节 */
NET_BUF_POOL_DEFINE(acl_tx_pool, 3, 65, 4, NULL);

struct bt_l2cap_hdr {
    u16_t len;
    u16_t cid;
};

struct bt_att_notify {
    u16_t handle;
    u8_t value[0];  //value 数组的长度为 0，说明 value 的长度是可变的
};

struct bt_att_hdr {
    u8_t code;
};

int bt_gatt_notify(struct bt_conn *conn, const struct bt_gatt_attr *attr, const void *data, u16_t len) {
    if(conn){  //若指定了连接，则调用 gatt_notify
        return gatt_notify(conn, attr->handle, data, len);
    }
}

int gatt_notify(...) {
    struct net_buf *buf;  //数据包
    struct bt_att_notify *nfy;
    buf = bt_att_create_pdu(conn, BT_ATT_OP_NOTIFY, sizeof(*nfy)+len);
    
    nfy = net_buf_add(buf, sizeof(*nfy));  //添加 ATT 数据
    nfy->handle = sys_cpu_to_le_16(handle);
    
    net_buf_add(buf, len);  //为 nfy->value 分配存储空间
    memcpy(nfy->value, data, len);  //复制通知数据
    bt_l2cap_send(conn, BT_L2CAP_CID_ATT, buf);
    return 0;
}

struct net_buf *bt_att_create_pdu(...) {
    struct bt_att_hdr *hdr;
    struct net_buf *buf;
    
    buf = net_buf_alloc(&acl_tx_pool, K_FOREVER);  //申请 pdu 缓冲区
    size_t reserve = sizeof(struct bt_hci_acl_hdr) + CONFIG_BT_HCI_RESERVE;
    net_buf_reserve(buf, reserve);  //保留 pdu 头部空间
    
    hdr = net_buf_add(buf, sizeof(*hdr));
    hdr->code = BT_ATT_OP_NOTIFY;
    return buf;
}

void bt_l2cap_send(...) {
    struct bt_l2cap_hdr *hdr;
    
    hdr = net_buf_push(buf, sizeof(*hdr));  //添加 L2CAP 头部
    hdr->len = sys_cpu_to_le16(buf->len - sizeof(*hdr));
    hdr->cid = sys_cpu_to_le16(cid);
    
    conn_tx(buf)->cb = NULL;  //无回调函数
    net_buf_put(&conn->tx_queue, buf);  //将 pdu 缓冲添加到发送队列
}

//pdu: ll_header -> bt_l2cap_hdr -> bt_att_hdr -> bt_att_notify

hci_tx_thread() -> process_event() -> bt_conn_process_tx();
void bt_conn_process_tx(struct bt_conn *conn) {
    struct net_buf *buf;
    buf = net_buf_get(&conn->tx_queue, K_NO_WAIT);  //获得要发送的缓冲
    if(!send_buf(conn, buf)) net_buf_unref(buf);  //发送缓冲
}

static bool send_buf(struct bt_conn *conn, struct net_buf *buf) {
    if(buf->len <= conn_mtu(conn)) return send_frag();
}

send_frag() -> hci_driver_send() -> acl_handle() -> hci_acl_handle();

int hci_acl_handle(...) {
    pdu_data->ll_id = PDU_DATA_LLID_DATA_START;  //0x02
    pdu_data->len = len;
    memcpy(&pdu_data->payload.lldata[0], buf->data, len);
    
}

```

`bt_gatt_discover()` 函数实现：

```c
int bt_gatt_discover(...) {
    switch(params->type) {
        case BT_GATT_DISCOVER_PRIMARY:
        case BT_GATT_DISCOVER_SECONDARY:
            return gatt_find_type(conn, params);
        case BT_GATT_DISCOVER_INCLUDE:
        case BT_GATT_DISCOVER_CHARACTERISTIC:
            return gatt_read_type(conn, params);
        case BT_GATT_DISCOVER_DESCRIPTOR:
            return gatt_find_info(conn, params);
    }
}

static int gatt_find_type(...) {
    struct net_buf *buf = bt_att_create_pdu(...);
    struct bt_att *att = att_chan_get(conn);
    req = net_buf_add(buf, sizeof(*req));
    req->start_handle = params->start_handle;
    req->end_handle = params->end_handle;
    req->type = 0x2800;
    net_buf_add_le16(buf, params->uuid);
    /* 注意这里注册了 gatt_find_type_rsp 这个回调函数 */
    return gatt_send(conn, buf, gatt_find_type_rsp, params, NULL);
}

static void gatt_find_type_rsp(...) {
    
}

static int gatt_send(...) {
    struct bt_att_req *req = params;
    req->buf = buf;
    req->func = func;
    req->destroy = NULL;
    net_buf_put(&conn->tx_queue, buf);
}

static int acl_handle(struct net_buf *buf) {
    
    radio_tx_mem_enqueue()
    
}
```

