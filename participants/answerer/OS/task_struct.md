无论是线程还是进程，在内核层面统一称为任务，由一个统一的结构`task_struct`来管理。

### 1 任务ID
任务ID是任务的唯一标识，在tast_struct中，主要涉及以下几个ID
```c++
    pid_t pid;
    pid_t tgid;
    struct task_struct *group_leader;
```
之所以有pid(process id)，tgid(thread group ID)以及group_leader，是因为线程和进程在内核中是统一管理，视为相同的任务（task）。

任何一个进程，如果只有主线程，那 pid 和tgid相同，group_leader 指向自己。但是，如果一个进程创建了其他线程，那就会有所变化了。线程有自己的pid，tgid 就是进程的主线程的 pid，group_leader 指向的进程的主线程。因此根据pid和tgid是否相等我们可以判断该任务是进程还是线程。

### 2 亲缘关系
除了0号进程以外，其他进程都是有父进程的。全部进程其实就是一颗进程树，相关成员变量如下所示
```c++
    struct task_struct __rcu *real_parent; /* real parent process */
    struct task_struct __rcu *parent; /* recipient of SIGCHLD, wait4() reports */
    struct list_head children;      /* list of my children */
    struct list_head sibling;       /* linkage in my parent's children list */
```
parent 指向其父进程。当它终止时，必须向它的父进程发送信号。
children 指向子进程链表的头部。链表中的所有元素都是它的子进程。
sibling 用于把当前进程插入到兄弟链表中。
通常情况下，real_parent 和 parent 是一样的，但是也会有另外的情况存在。例如，bash 创建一个进程，那进程的 parent 和 real_parent 就都是 bash。如果在 bash 上使用 GDB 来 debug 一个进程，这个时候 GDB 是 parent，bash 是这个进程的 real_parent。

### 3 任务状态
任务状态部分主要涉及以下变量
```c+++
    volatile long state;    /* -1 unrunnable, 0 runnable, >0 stopped */
    int exit_state;
    unsigned int flags;
```
其中状态state通过设置比特位的方式来赋值，具体值在include/linux/sched.h中定义
```c++
    /* Used in tsk->state: */
    #define TASK_RUNNING                    0
    #define TASK_INTERRUPTIBLE              1
    #define TASK_UNINTERRUPTIBLE            2
    #define __TASK_STOPPED                  4
    #define __TASK_TRACED                   8
    /* Used in tsk->exit_state: */
    #define EXIT_DEAD                       16
    #define EXIT_ZOMBIE                     32
    #define EXIT_TRACE                      (EXIT_ZOMBIE | EXIT_DEAD)
    /* Used in tsk->state again: */
    #define TASK_DEAD                       64
    #define TASK_WAKEKILL                   128
    #define TASK_WAKING                     256
    #define TASK_PARKED                     512
    #define TASK_NOLOAD                     1024
    #define TASK_NEW                        2048
    #define TASK_STATE_MAX                  4096
    
    #define TASK_KILLABLE           (TASK_WAKEKILL | TASK_UNINTERRUPTIBLE)
```
TASK_RUNNING并不是说进程正在运行，而是表示进程在时刻准备运行的状态。当处于这个状态的进程获得时间片的时候，就是在运行中；如果没有获得时间片，就说明它被其他进程抢占了，在等待再次分配时间片。在运行中的进程，一旦要进行一些 I/O 操作，需要等待 I/O 完毕，这个时候会释放 CPU，进入睡眠状态。

在 Linux 中，有两种睡眠状态。

一种是 TASK_INTERRUPTIBLE，可中断的睡眠状态。这是一种浅睡眠的状态，也就是说，虽然在睡眠，等待 I/O 完成，但是这个时候一个信号来的时候，进程还是要被唤醒。只不过唤醒后，不是继续刚才的操作，而是进行信号处理。当然程序员可以根据自己的意愿，来写信号处理函数，例如收到某些信号，就放弃等待这个 I/O 操作完成，直接退出；或者收到某些信息，继续等待。
另一种睡眠是 TASK_UNINTERRUPTIBLE，不可中断的睡眠状态。这是一种深度睡眠状态，不可被信号唤醒，只能死等 I/O 操作完成。一旦 I/O 操作因为特殊原因不能完成，这个时候，谁也叫不醒这个进程了。你可能会说，我 kill 它呢？别忘了，kill 本身也是一个信号，既然这个状态不可被信号唤醒，kill 信号也被忽略了。除非重启电脑，没有其他办法。因此，这其实是一个比较危险的事情，除非程序员极其有把握，不然还是不要设置成 TASK_UNINTERRUPTIBLE。
于是，我们就有了一种新的进程睡眠状态，TASK_KILLABLE，可以终止的新睡眠状态。进程处于这种状态中，它的运行原理类似 TASK_UNINTERRUPTIBLE，只不过可以响应致命信号。由于TASK_WAKEKILL 用于在接收到致命信号时唤醒进程，因此TASK_KILLABLE即在TASK_UNINTERUPTIBLE的基础上增加一个TASK_WAKEKILL标记位即可。
TASK_STOPPED是在进程接收到 SIGSTOP、SIGTTIN、SIGTSTP或者 SIGTTOU 信号之后进入该状态。

TASK_TRACED 表示进程被 debugger 等进程监视，进程执行被调试程序所停止。当一个进程被另外的进程所监视，每一个信号都会让进程进入该状态。

一旦一个进程要结束，先进入的是 EXIT_ZOMBIE 状态，但是这个时候它的父进程还没有使用wait() 等系统调用来获知它的终止信息，此时进程就成了僵尸进程。EXIT_DEAD 是进程的最终状态。EXIT_ZOMBIE 和 EXIT_DEAD 也可以用于 exit_state。

上面的进程状态和进程的运行、调度有关系，还有其他的一些状态，我们称为标志。放在 flags字段中，这些字段都被定义成为宏，以 PF 开头。
```c++
#define PF_EXITING    0x00000004
#define PF_VCPU      0x00000010
#define PF_FORKNOEXEC    0x00000040
```
PF_EXITING 表示正在退出。当有这个 flag 的时候，在函数 find_alive_thread() 中，找活着的线程，遇到有这个 flag 的，就直接跳过。

PF_VCPU 表示进程运行在虚拟 CPU 上。在函数 account_system_time中，统计进程的系统运行时间，如果有这个 flag，就调用 account_guest_time，按照客户机的时间进行统计。

PF_FORKNOEXEC 表示 fork 完了，还没有 exec。在 _do_fork ()函数里面调用 copy_process()，这个时候把 flag 设置为 PF_FORKNOEXEC()。当 exec 中调用了 load_elf_binary() 的时候，又把这个 flag 去掉。

