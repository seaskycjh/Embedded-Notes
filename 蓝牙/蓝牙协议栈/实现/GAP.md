GAP

GAP 层包括 Hci_core.c 和 Conn.c。

使用蓝牙功能前必须先调用 bt_enable 函数。

1 蓝牙初始化

```c
struct bt_dev bt_dev = {
    .init = init_work;
}
static bt_ready_ct_t ready_cb;

int bt_enable(bt_read_cb_t cb)
{
    bt_setting_init();
    bt_set_name(CONFIG_BT_DEVICE_NAME);
    ready_cb = cb;
    k_thread_create(..., hci_rx_thread, ...); //创建发送数据线程
    k_thread_create(..., hci_tx_thread, ...); //创建接收数据线程
    bt_hci_ecc_init();
    bt_dev.drv->open();	//调用蓝牙设备驱动的 open 函数（见 Hci_driver.c）
    //若回调函数 cb 为空
    bt_init();
    //否则
    k_work_submit(&bt_dev.init);
    return 0;
}

static void init_work(struct k_work *work)
{
    bt_init();
    //若回调函数不为空
    ready_cb();	//执行回调函数
}

static int bt_init(void)
{
    hci_init();
    //若使能了蓝牙连接，则初始化蓝牙连接
    bt_conn_init();
    //如何使能了蓝牙加密
    k_delayed_work_init(&bt_dev.rpa_update);
    bt_finalize_init();	//完成初始化
}

//初始化蓝牙连接
int bt_conn_init(void)
{
    bt_att_init();
    bt_smp_init();
    bt_l2cap_init();
}

static void hci_rx_thread(...)
{
    while(1){
        //获取蓝牙设备接收到的数据
        buf = net_buf_get(&bt_dev.rx_queue);
        switch(bt_buf_get_type(buf)){
            case BT_BUF_ACL_IN:
                hci_acl(buf);
            case BT_BUF_EVT:
                hci_event(buf);
        }
        k_yield();
    }
}

static void hci_acl(struct net_buf *buf)
{
    bt_conn_lookup_handle(acl(buf)->handle);
}

static void hci_tx_thread(...)
{
    //初始化轮询事件组
    static struct k_poll_event events[1] = {...};
    while(1){
        //如果蓝牙已连接，则添加相关事件到轮询事件组
        ev_count += bt_conn_prepare_events(&events[1]);
        err = k_poll(events, ev_count, K_FOREVER);
        //处理轮询事件
        process_events(events, ev_count);
        k_yield();
    }
}

static void process_events(struct k_poll_event *ev, int count)
{
    for(; count; ev++, count--){
        switch(ev->state){
            case K_POLL_STATE_SIGNALED:
            case K_POLL_STATE_FIFO_DATA_AVAILABLE:
                //若 ev->tag == BT_EVENT_CMD_TX
                send_cmd();
                //若使能了蓝牙连接
                struct bt_conn *conn;
                //若 ev->tag == BT_EVENT_CONN_TX_NOTIFY
                bt_conn_notify_tx(conn);
                //若 ev->tag == BT_EVENT_CONN_TX_QUEUE
                bt_conn_process_tx(conn);
            case K_POLL_STATE_NOT_READY:
            default:
        }
    }
}

static void send_cmd(void)
{
    buf = net_buf_get(&bt_dev.cmd_tx_queue, K_NO_WAIT);
    k_sem_take(&bt_dev.ncmd_sem, K_FOREVER);
    err = bt_send(buf);
    if(err){
        k_sem_give(&bt_dev.ncmd_sem);
        //完成 HCI 命令发送
        hci_cmd_done(cmd(buf)->opcode, BT_HCI_ERR_UNSECIFIED, NULL);
    }
}

int bt_send(struct net_buf *buf)
{
    bt_monitor_send(...);
    //若使能了加密
    return bt_hci_ecc_send(buf);
    return bt_dev.drv->send(buf);	//调用蓝牙设备驱动的 send 函数
}

//蓝牙 HCI 驱动注册，由 Hci_driver.c 中的函数调用
int bt_hci_driver_register(const struct bt_hci_driver *drv)
{
    //若蓝牙驱动已存在，则返回错误码
    //若蓝牙驱动的 open 或 send 函数不存在，则返回错误码
    bt_dev.drv = drv;
    bt_monitor_new_index(...);
    return 0;
}
```

2 HCI 驱动

```c
//初始化蓝牙 HCI 驱动
static const struct bt_hci_driver drv = {
    .name = "Controller",
    .bus = BT_HCI_DRIVER_BUS_VIRTUAL,
    .open = hci_driver_open,
    .send = hci_driver_send,
};

static int hci_driver_open(void)
{
    ll_init(&sem_prio_recv);	//LL 层初始化
    //若定义了蓝牙 HCI 流控制
    hci_init(&hbuf_signal);
    //否则
    hci_init(NULL);
    k_thread_create(..., prio_recv_thread, ...);
    k_thread_create(..., recv_thread, ...);
    return 0;
}

```

3 CONN 初始化

```c

static struct bt_conn_cb *callback_list;
const struct bt_conn_auth_cb *bt_auth;
//注册连接回调函数，利用了链表的头插法
void bt_conn_cb_register(struct bt_conn_cb *cb)
{
    cb->_next = callback_list;
    callback_list = cb;
}

//只能注册一次
int bt_conn_auth_cb_register(const struct bt_conn_auth_cb *cb)
{
    //回调为空，返回
    //回调已存在，返回
    //cancel 回调必须存在
    bt_auth = cb;
    return 0;
}

//建立连接后准备轮询事件
int bt_conn_prepare_events(struct k_poll_events[])
{
    int i, ev_count = 0;
    k_poll_event_init(...);	//轮询事件初始化
    //为每个连接建立轮询事件
    for(i = 0; i < ARRAY_SIZE(conns); i++){
        struct bt_conn *conn = &conns[i];
        //若连接不可用，继续
        //若已断开连接，释放内存并继续
        //若连接状态不是已连接，继续
        //监听 conn 的 tx_notify
        k_poll_event_init(..., &conn->tx_notify);
        events[ev_count++].tag = BT_EVENT_CONN_TX_NOTIFY;
        //监听 conn 的 tx_queue，通知等方法就是将 buf 添加到 tx_queue
        k_poll_event_init(..., &conn->tx_queue);
        events[ev_count++].tag = BT_EVENT_CONN_TX_QUEUE;
    }
    return ev_count;
}

//处理已连接后的发送事件（如通知等）
void bt_conn_process_tx(struct bt_conn *conn)
{
    //若已断开连接，则清理连接数据
    but = net_buf_get(&conn->tx_queue, K_NO_WAIT);
    send_buf(conn, buf);
}

static bool send_buf(...)
{
    //若 buf 长度 <= conn 的 MTU，则直接发送
    return send_frag(conn, buf, ...);
    create_frag(conn, buf);
    send_frag(conn, frag, ...);
    while(buf->len > conn_mtu(conn)){
        create_frag(conn, buf);
        send_frag(conn, frag);
    }
    return send_frag(conn, buf);
}

static bool send_frag(...)
{
    k_sem_take(...);	//等待控制器能够接收 ACL 包
    notify_tx();
    bt_buf_set_type(buf, BT_BUF_ACL_OUT);
    bt_send(buf);	//最后调用 Hci_core.c 中的 bt_send，即调用蓝牙设备驱动的 send 函数
    return true;
}
```

4 广播

`bt_le_adv_start()` 函数实现：通过发送 HCI 命令给控制器实现相应功能，不需经过 ATT 等上层的封装，直接由 LL 层生成广播 PDU 并发送。

```c
int bt_le_adv_start(...)
{
    int err;
    /* 发送设置广播数据 HCI 命令，实际效果是让控制器设置 advertiser 的广播数据 */
    err = set_ad(BT_HCI_OP_LE_SET_ADV_DATA, ad, ad_len);
}

int set_ad(...) {
    buf = bt_hci_cmd_create(...);
    set_data = net_buf_add(buf, sizeof(*set_data));
    for(i = 0; i < ad_len; i++) {
        set_data->data[set_data->len++] = ad[i].data_len+1;
        set_data->data[set_data->len++] = ad[i].type;
        memcpy(&set_data->data[set_data->len], ad[i].data, ad[i].data_len);
        set_data->len += ad[i].data_len;
    }
    return bt_hci_cmd_send_sync(...);
    
}

/* 所有 LE HCI 命令最后都由此函数处理 */
int controller_cmd_handle()
```

5 连接

`bt_conn_create_le()` 函数实现：

```c
struct bt_conn *bt_conn_create_le(const bt_addr_le_t *peer, 
    const struct bt_le_conn_param *param) {
    /* 根据所要连接的设备地址查找已存在的连接 */
    struct bt_conn *conn = bt_conn_lookup_addr_le(peer);
    if(conn) return conn;
    /* 若连接不存在，则创建一个新的连接 */
    conn = bt_conn_add_le(peer);
    conn = conn_new();  //从连接列表中找到未引用的连接项
    bt_addr_le_copy(&conn->le.resp_addr, peer);
    bt_conn_set_param_le(conn, param);
    conn->state = BT_CONN_CONNECT_SCAN;
    bt_conn_ref(conn);
    bt_le_scan_update(true);  //开启被动扫描，之后接收到广播报告事件
}

/* 判断地址是否为可解析地址，即最高两位是否为 01 */
#define BT_ADDR_IS_RPA(a) (((a)->val[5] & 0xc0) == 0x40)

/* 判断可解析地址的 irk 与给出的 irk 是否匹配 */
bool bt_rpa_irk_matches(const u8_t irk[16], const bt_addr_t *addr) {
    u8_t hash[3];
    int err = ah(irk, addr->val+3, hash);  //通过 ah 生成 hash 码
    return !memcmp(addr->val, hash, 3);  //将生成的 hash 码与 addr 的 hash 码进行比较
}

void le_conn_update(struct k_work *work) {
    bt_conn_le_param_update(conn, param);
}

void le_adv_report(struct net_buf *buf) {
    check_pending_conn(addr, &info->addr, info->evt_type);
}

/* 发送 HCI 连接命令后，会收到连接完成事件 */
void le_conn_complete(struct net_buf *buf) {
    k_fifo_init(&conn->tx_queue);
    k_fifo_init(&conn->tx_notify);
    sys_slist_init(&conn->channels);
    bt_l2cap_connected(conn);
    notify_connected(conn);
}


```

6 扫描

`bt_le_scan_start()` 函数实现：先发送 LE Set Scan Parameters 命令，再发送 LE Set Scan Enable  命令，之后生成 LE Advertising Reports  事件。

```c
int bt_le_scan_start(...) {
    start_le_scan(...);
    scan_dev_found_cb = cb;
}

static void le_adv_report(struct net_buf *buf) {
    u8_t num_reports = net_buf_pull_u8(buf);
    struct bt_hci_evt_le_advertising_info *info;
    while(num_reports--) {
        info = (void *)buf->data;
        net_buf_pull(buf, sizeof(*info));
        s8_t rssi = info->data[info->length];
        bt_addr_le_t *addr = find_id_addr(&info->addr);
        scan_dev_found_cb(addr, rssi, info->evt_type, &buf->b);
        net_buf_pull(buf, info->length + sizeof(rssi));
    }
}
```

7 接收数据

```c
void hci_rx_thread(void) {
    while(1) {
        buf = net_buf_get(&bt_dev.rx_queue, K_FOREVER);
        switch(bt_buf_get_type(buf)) {
            case BT_BUF_ACL_IN:
            case BT_BUF_EVT:
        }
        k_yield();
    }
}

void hci_acl(struct net_buf *buf) {
    struct bt_hci_acl_hdr *hdr = (void *)buf->data;
    net_buf_pull(buf, sizeof(*hdr));
    conn = bt_conn_lookup_handle(handle);
    bt_conn_recv(conn, buf, flags);
}

/* L2CAP 对数据进行分割 */
void bt_conn_recv(...) {
    switch(flags) {
        case BT_ACL_START:
        case BT_ACL_CONT:
    }
}
```

8 配对

```c
int bt_conn_security(struct bt_conn *conn, bt_security_t sec) {
    int err = start_security(conn);
}

int start_security(...) {
    switch(conn->role) {
        case BT_HCI_ROLE_MASTER:  //主机
            bt_keys_find();
            if(conn->required_sec_level>BT_SECURITY_MEDIUM)
                return bt_smp_send_pairing_req(conn);
        case BT_HCI_ROLE_SLAVE:  //从机
            return bt_smp_send_pairing_req(conn);
    }
}
```

