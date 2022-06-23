对于较多数量的文件描述符的监听无论是select还是poll系统调用都显得捉襟见肘，poll每次都需要将所有的文件描述符复制到内核，内核本身不会对这些文件描述符加以保存，这样的设计就导致了poll的效率的低下。

而epoll则对此做了相应的改进，不是epoll_wait的时候才传入fd，而是通过epoll_ctl把所有fd传入内核，再一起”wait”，这就省掉了不必要的重复拷贝。

其次，在 epoll_wait时，也不是把current轮流的加入fd对应的设备等待队列，而是在设备等待队列醒来时调用一个回调函数（当然，这就需要“唤醒回调”机制），把产生事件的fd归入一个链表，然后返回这个链表上的fd。另外，epoll机制实现了自己特有的文件系统eventpoll filesystem。

### epoll初始化
当系统启动时，epoll会进行初始化操作
```c++
static int __init eventpoll_init(void)
{
mutex_init(&epmutex);

    /* Initialize the structure used to perform safe poll wait head wake ups */
    ep_poll_safewake_init(&psw);

    /* Allocates slab cache used to allocate "struct epitem" items */
    epi_cache = kmem_cache_create("eventpoll_epi", sizeof(struct epitem),
            0, SLAB_HWCACHE_ALIGN|EPI_SLAB_DEBUG|SLAB_PANIC,
            NULL);

    /* Allocates slab cache used to allocate "struct eppoll_entry" */
    pwq_cache = kmem_cache_create("eventpoll_pwq",
            sizeof(struct eppoll_entry), 0,
            EPI_SLAB_DEBUG|SLAB_PANIC, NULL);

    return 0;
}
```
### fs_initcall(eventpoll_init);
上面的代码实现一些数据结构的初始化,通过fs/eventpoll.c中的注释可以看出,有三种类型的锁机制使用场景:

1. epmutex(mutex):用户关闭文件描述符，但是没有调用EPOLL_CTL_DEL
2. ep->mtx(mutex):用户态与内核态的转换可能会睡眠
3. ep->lock(spinlock):内核态与具体设备中断过程中的转换,poll回调

接下来就是使用slab分配器动态分配内存，第一个结构为当系统中添加一个fd时，就创建一epitem结构体,内核管理的基本数据结构。

内核数据结构
epoll在内核主要维护了两个数据结构eventpoll与epitem，其中eventpoll表示每个epoll实例本身，epitem表示的是每一个IO所对应的的事件。
```c++
struct epitem {
/* RB tree node used to link this structure to the eventpoll RB tree */
struct rb_node rbn; /*用于挂载到eventpoll管理的红黑树*/

    /* List header used to link this structure to the eventpoll ready list */
    struct list_head rdllink; /*挂载到eventpoll.rdlist的事件就绪队列*/

    /*
     * Works together "struct eventpoll"->ovflist in keeping the
     * single linked chain of items.
     */
    struct epitem *next; /*用于主结构体中的链表*/

    /* The file descriptor information this item refers to */
    struct epoll_filefd ffd; /*该结构体对应的被监听的文件描述符信息(fd+file， 作为红黑树的key)*/

    /* Number of active wait queue attached to poll operations */
    int nwait;  /*poll(轮询操作)的事件个数

    /* List containing poll wait queues */
    struct list_head pwqlist; /*双向链表，保存被监视文件的等待队列，功能类似于select/poll中的poll_table；同一个文件上可能会监视多种事件，这些事件可能从属于不同的wait_queue中，所以需要使用链表

    /* The "container" of this item */
    struct eventpoll *ep; /*当前epitem的所有者（多个epitem从属于一个eventpoll）*/

    /* List header used to link this item to the "struct file" items list */
    struct list_head fllink; /*双向链表，用来链接被监视的文件描述符对应的struct file。因为file里有f_ep_link用来保存所有监视这个文件的epoll节点

    /* The structure that describe the interested events and the source fd */
    struct epoll_event event; /*注册感兴趣的事件，也就是用户空间的epoll_event
};
```
而每个epoll fd对应的主要数据结构为：
```c++
struct eventpoll {
/* Protect the this structure access */
spinlock_t lock; /*自旋锁，在kernel内部用自旋锁加锁，就可以同时多线(进)程对此结构体进行操作，主要是保护ready_list*/

    /*
     * This mutex is used to ensure that files are not removed
     * while epoll is using them. This is held during the event
     * collection loop, the file cleanup path, the epoll file exit
     * code and the ctl operations.
     */
    struct mutex mtx; /*防止使用时被删除*/

    /* Wait queue used by sys_epoll_wait() */
    wait_queue_head_t wq; /*sys_epoll_wait()使用的等待队列*/

    /* Wait queue used by file->poll() */
    wait_queue_head_t poll_wait; /*file->epoll()使用的等待队列*/

    /* List of ready file descriptors */
    struct list_head rdllist; /*事件就绪链表*/

    /* RB tree root used to store monitored fd structs */
    struct rb_root rbr; /*用于管理当前epoll关注的文件描述符（树根）*/

    /*
     * This is a single linked list that chains all the "struct epitem" that
     * happened while transfering ready events to userspace w/out
     * holding ->lock.
     */
    struct epitem *ovflist; /*在向用户空间传输就绪事件的时候，将同时发生事件的文件描述符链入到这个链表里面*/
};
```