1 FIFO 的实现

FIFO 的实现利用了系统的单链表，注意 `k_fifo_put()` 的实现，是通过将 `data` 指针转换为 `struct _snode *` 指针来插入的，因此 `data` 指针指向的数据的前四个字节会发生改变（变为指向的下一节点的地址），这就是为什么在使用 FIFO 时用户自定义的数据项要保留一个字长的空间，这个空间就是用于存储 `next` 指针的。

```c
#define k_fifo_put(fifo, data)  \
	k_queue_append((struct k_queue *)fifo, data)

#define k_fifo_get(fifo, timeout)  \
	k_queue_get((struct k_queue *)fifo, timeout)

void k_queue_append(...) {
    return k_queue_insert(queue, queue->data_q.tail, data);
}

void k_queue_insert(struct k_queue *queue, void *prev, void *data) {
    sys_slist_insert(&queue->data_q, prev, data);
}

void *k_queue_get(struct k_queue *queue, s32_t timeout) {
    void *data = sys_slist_get_not_empty(&queue->data_q);
    return data;
}

/* _snode 结构体只包含了一个指针，因此使用单链表时一般将 _snode 作为数据项的第一个成员变量 */
struct _snode {
    struct _snode *next;
}

/* 使用示例 */
struct data {
    sys_snode_t *node;
    //user_data;
}

/* 链表插入节点，分三种情况：头部、尾部、中间 */
void sys_slist_insert(sys_slist_t *list, sys_snode_t *prev, sys_snode_t *node) {
    if(!prev) sys_slist_prepend(list, node);  //前节点为空
    else if(!prev->next) sys_slist_append(list, node);  //插入链表尾部
    else {  //插入
        node->next = prev->next;
        prev->next = node;
    }
}

/* 链表获取表头节点并删除（出队） */
sys_snode_t *sys_slist_get_not_empty(sys_slist_t *list) {
    sys_snode *node = list->head;  //获取表头节点
    list->head = node->next;
    if(list->tail == node) list->tail = list->head;  //若尾节点，表为空（队首=队尾=NULL）
    return node;
}
```

