1 初始化

```c
int ll_init(struct k_sem *sem_rx) {
    rand_init(...);
    rand_isr_init(...);
    clk_k32 = device_get_binding(...);
    cntr_init();
    mayfly_init();
    ticker_init();
    clk_m16 = device_get_binding(...);
    radio_init(...);
    IRQ_CONNECT(...);
    irq_enable(...);
}
```



2 中断处理

```c
static inline void isr_radio_state_tx(void) {
    switch(_radio.role) {
        case ROLE_ADV:
        case ROLE_SCAN:
        case ROLE_MASTER:
        case ROLE_SLAVE:
        case ROLE_NONE:
    }
}

static inline u32_t isr_rx_adv(...) {
    
}

static inline u32_t isr_rx_scan(...) {
    
}

static inline void isr_rx_conn(...) {
    
}

static void isr(void) {
    
}
```



3 功能实现

```c
u32_t radio_adv_enable(...) {
    
}

u32_t radio_scan_enable(...) {
    
}

u32_t radio_connect_enable(...) {
    
}

u8_t radio_rx_get(...) {
    
}
```

