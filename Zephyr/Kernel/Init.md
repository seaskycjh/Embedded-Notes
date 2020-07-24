系统初始化

1 _PrepC

```c
void _PrepC(void)
{
    _Cstart();
}

void _Cstart(void)
{
    prepare_multithreading(dummy_thread);
    _sys_device_do_config_level(_SYS_INIT_LEVEL_PRE_KERNEL_1);
	_sys_device_do_config_level(_SYS_INIT_LEVEL_PRE_KERNEL_2);
    switch_to_main_thread();
}
/* 准备多线程 */
static void prepare_multithreading(struct k_thread *dummy_thread)
{
    _IntLibInit();
    _new_thread(_main_thread, ..., _main, ...);
    _new_thread(_idle_thread, ..., idle, ...);
    kernel_arch_init();
}
```

2 _main

```c
/* 此例程通过调用其余的初始化函数来完成内核初始化，然后调用应用程序的 main() 例程 */
static void _main(void *unused1, void *unused2, void *unused3)
{
    /* 内核级别驱动初始化 */
    _sys_device_do_config_level(_SYS_INIT_LEVEL_POST_KERNEL);
    /* 应用级别驱动初始化 */
    _sys_device_do_config_level(_SYS_INIT_LEVEL_APPLICATION);
    //启动延时
    _init_static_threads();
    main();
}
```

