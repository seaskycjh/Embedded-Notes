线程

```c
struct k_thread {
    struct _thread_base base;
    struct _thread_entry *entry;
}
```

1 定义线程

```c
/* 定义一个线程 
 * 创建线程栈，创建线程对象，创建线程数据
 * 
 */
#define K_THREAD_DEFINE(name, stack_size, entry, p1, p2, p3, prio, options, delay) \
	K_THREAD_STACK_DEFINE(_k_thread_stack_##name, stack_size); \
	struct k_thread _k_thread_obj_##name; \
	struct _static_thread_data _k_thread_data_##name __aligned(4) \
		__in_section(_static_thread_data, static, name) = \
		_THREAD_INITIALIZER(&_k_thread_obj_##name, \
			_k_thread_stack_##name, stack_size, entry, \
			p1, p2, p3, prio, options, delay, NULL, 0); \
	const k_tid_t name = (k_tid_t)&_k_thread_obj_##name

#define _THREAD_INITIALIZER(thread, stack, stack_size, entry, \
							p1, p2, p3, prio, options, delay, abort, groups) \
	{                                                        \
	.init_thread = (thread), \
	.init_stack = (stack), \
	.init_stack_size = (stack_size),                         \
	.init_entry = (void (*)(void *, void *, void *))entry,   \
	.init_p1 = (void *)p1,                                   \
	.init_p2 = (void *)p2,                                   \
	.init_p3 = (void *)p3,                                   \
	.init_prio = (prio),                                     \
	.init_options = (options),                               \
	.init_delay = (delay),                                   \
	.init_abort = (abort),                                   \
	.init_groups = (groups),                                 \
	}
```

```c
void _new_thread(struct k_thread *thread, k_thread_stack_t  )
```

