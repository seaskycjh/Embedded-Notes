设备驱动模型

1 设备定义

设备定义即创建一个设备对象并将其设置为引导时初始化（引导系统时会从设备列表中按优先性逐个调用设备的初始化函数）。设备定义通过调用 `device.h` 中的宏来实现，即定义了一个 device_config 和一个 device 静态变量。

```c
/* 设备配置结构体 */
struct device_config {
    char *name;  //设备名
    int (*init)(struct device *device);  //设备初始化函数
    int (*device_pm_control)(struct device *device, u32_t command, void *context);
    const void *config_info;  //设备配置信息
};
/* 设备结构体 */
struct device {
    struct device_config *config;
    const void *driver_api;  //驱动 api
    void *driver_data;  //驱动数据
};

#define _CONCAT(x,y) x ## y
#define _STRINGIFY(x) #x
#define STRINGIFY(s) _STRINGIFY(s)
/* 用于说明这段代码是有用的，即使没用到编译器也不会警告 */
#define __used __attribute__((__used__))

#define DEVICE_DEFINE(dev_name, drv_name, init_fn, pm_control_fn, \
		      data, cfg_info, level, prio, api) \
	\
	static struct device_config _CONCAT(__config_, dev_name) __used \
	__attribute__((__section__(".devconfig.init"))) = { \
		.name = drv_name, \
		.init = (init_fn), \
		.device_pm_control = (pm_control_fn), \
		.config_info = (cfg_info) \
	}; \
	static struct device _CONCAT(__device_, dev_name) __used \
	__attribute__((__section__(".init_" #level STRINGIFY(prio)))) = { \
		 .config = &_CONCAT(__config_, dev_name), \
		 .driver_api = api, \
		 .driver_data = data \
	}

/* 实例，例如定义了一个 UART 驱动，则展开后如下 */
DEVICE_DEFINE(uart, "UART", &uart_init, uart_ctrl, &uart_data, &uart_cfg, 						  PRE_KERNEL_1, CONFIG_KERNEL_INIT_PRIORITY_DEVICE, NULL);
/* 将结构体放入名为 .devconfig.init 的输入段 */
static struct device_config __config_uart = {
    .name = "UART",
    .init = (&uart_init),
    .device_pm_control = (uart_ctrl),
    .config_info = (&uart_cfg)
};
/* 将结构体放入名为 .init_PRE_KERNEL_150 的输入段 */
static struct device __device_uart = {
    .config = &__config_uart,
    .driver_api = NULL,
    .driver_data = &uart_data
};

/* 初始化系统设备 */
void _sys_device_do_config_level(int level)
{
	struct device *info;
	for (info = config_levels[level]; info < config_levels[level+1]; info++){
		struct device_config *device = info->config;
		device->init(info);  //调用设备的初始化函数
	}
}

/* 根据设备名获取设备结构体指针 */
struct device *device_get_binding(const char *name)
{
	struct device *info;
    /* 从存储设备结构体的数组中查找对应名字的设备 */
	for (info = __device_init_start; info != __device_init_end; info++) {
		if (info->driver_api && !strcmp(name, info->config->name)) {
			return info;
		}
	}
	return NULL;
}

```

2 中断

```c
/**
 * @brief 初始化一个中断处理
 * @param IRQ 行号
 * @param 中断优先级
 * @param 中断服务例程的地址
 * @param 中断配置标志
 */
#define IRQ_CONNECT(irq_p, priority_p, isr_p, isr_param_p, flags_p) \
	_ARCH_IRQ_CONNECT(irq_p, priority_p, isr_p, isr_param_p, flags_p)

#define _ARCH_IRQ_CONNECT(irq_p, priority_p, isr_p, isr_param_p, flags_p) \
({ \
	_ISR_DECLARE(irq_p, 0, isr_p, isr_param_p); \
	_irq_priority_set(irq_p, priority_p, flags_p); \
	irq_p; \
})

#define _ISR_DECLARE(irq, flags, func, param) \
    _sw_isr_table[irq].arg = (void *)param; \
    _sw_isr_table[irq].isr = (void (*)(void *))func; 
```

