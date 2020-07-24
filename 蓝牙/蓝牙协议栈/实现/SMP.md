1 初始化

```c
int bt_smp_init(void) {
    struct bt_l2cap_fixed_chan chan = {
        .cid = BT_L2CAP_CID_SMP,
        .accept	= bt_smp_accept,
    };
    bt_l2cap_le_fixed_chan_register(&chan);
    bt_pub_key_gen(&pub_key_cb);
}

void bt_smp_recv(...) {
    //判断是否超时
    //判断接收的数据是否在 handlers 链表的范围内
    //判断是否接收到对应的命令（例如发送配对命令，那么则期待收到配对响应）
    //判断接收数据长度是否符合
    err = handlers[hdr->code].func(smp, buf);  //调用对应的处理函数
}

//handlers: line3380
```

2 配对

```c
/* MATSTER 发送配对请求 */
int bt_smp_send_pairing_req(struct bt_conn *conn) {
    //判断 SMP 是否超时
    //是否正在配对
    //安全等级能否实现
    smp_init(smp);
    req_buf = smp_create_pdu(conn, BT_SMP_CMD_PAIRING_REQ, sizeof(*req));
    req = net_buf_add(req_buf, sizeof(*req));
    req->auth_req = get_auth(BT_SMP_AUTH_DEFAULT);  //身份认证需求标志
    req->io_capability = get_io_capa();  //IO 能力标志
    req->max_key_size = BT_SMP_MAX_ENC_KEY_SIZE;  //128位的密钥大小
    req->init_key_dist = SEND_KEYS;  //主机分发给从机的密钥
	req->resp_key_dist = RECV_KEYS;  //
    //保存配对请求
    smp_send(smp, req_buf, NULL);  //发送配对请求
    //设置相应标志位（正在配对，接收配对相应）
    return 0;
}

/* SLAVE 请求配对，MASTER 收到请求后再发起配对 */
int bt_smp_send_security_req(struct bt_conn *conn) {
    bt_l2cap_send(conn, BT_L2CAP_CID_SMP, req_buf);
    atomic_set_bit(&smp->allowed_cmds, BT_SMP_CMD_PAIRING_FAIL);
}

/* MASTER 收到安全性改变请求 */
u8_t smp_security_request(...) {
    bt_smp_send_pairing_req(conn);
    atomic_set_bit(smp->flags, SMP_FLAG_SEC_REQ);
}

/* 从机收到配对请求 */
u8_t smp_pairing_req(...) {
    int ret = smp_init(smp);
    //保存配对请求
    //创建配对响应
    atomic_set_bit(smp->flags, SMP_FLAG_PAIRING);  //设置标志位，表示正在配对
    //没有使能 LE 安全连接，则采用遗留配对方法
    if(!atomic_test_bit(smp->flags, SMP_FLAG_SC))
        return legacy_pairing_rsp(smp, rsp->io_capability);
}

/* 从机执行遗留配对 */
u8_t legacy_pairing_req(...) {
    smp->method = legacy_get_pair_method(smp, remote_io);
    int ret = send_pairing_rsp(smp);  //发送配对响应
    //设置标志位，表示正在进行配对确认
    atomic_set_bit(&smp->allowed_cmds, BT_SMP_CMD_PAIRING_CONFIRM);
    return legacy_request_tk(smp);  //获取密钥
}

/* 从机发送配对响应 */
u8_t send_pairing_rsp(...) {
    smp_send(smp, rsp_buf, NULL);
}

/* 主机收到配对响应 */
u8_t smp_pairing_rsp() {
    //保存配对响应
    //根据响应设备的 IO 能力获取配对方法
    //没有使能 LE 安全连接，则采用遗留配对方法
    if(!atomic_test_bit(smp->flags, SMP_FLAG_SC))
        return legacy_pairing_rsp(smp, rsp->io_capability);
}

/* 主机进行 LE 遗留配对 */
u8_t legacy_pairing_rsp(...) {
    smp->method = legacy_get_pair_method(smp, remote_io);
    u8_t ret = legacy_request_tk(smp);  //执行用户注册的认证方法
    //用户输入密钥，则发送密钥到另一方
    if(!atomic_test_bit(smp->flags, SMP_FLAG_USER)) {
        atomic_set_bit(&smp->allowed_cmds, BT_SMP_CMD_PAIRING_CONFIRM);
		return legacy_send_pairing_confirm(smp);  //发送配对确认命令
    }
    atomic_set_bit(smp->flags, SMP_FLAG_CFM_DELAYED);  //设置配对延迟标志
    return 0;
}

/* 发送配对确认 */
u8_t legacy_send_pairing_confirm(...) {
    struct net_buf *buf;
    buf = smp_create_pdu(conn, BT_SMP_CMD_PAIRING_CONFIRM, sizeof(*req));
    smp_send(smp, buf, NULL);
    return 0;
}

/* 收到配对确认 */
u8_t smp_pairing_confirm(struct bt_smp *smp, struct net_buf *buf) {
    /* 计算确认值然后发送随机数 */
    return smp_send_pairing_random(smp);
}

/* 发送配对随机值 */
u8_t smp_send_pairing_random(...) {
    smp_send(...);
}

/* 收到配对随机值 */
u8_t smp_pairing_random(...) {
    return legacy_pairing_random(smp);
    #if defined(CONFIG_BT_PERIPHERAL)
    sc_smp_check_confirm(smp);
    smp_send_pairing_random(smp);
}
```



