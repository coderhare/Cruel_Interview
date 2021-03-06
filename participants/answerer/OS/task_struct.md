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

### 4 任务权限
任务权限主要包括以下两个变量，real_cred是指可以操作本任务的对象，而red是指本任务可以操作的对象。
```c++
/* Objective and real subjective task credentials (COW): */
const struct cred __rcu         *real_cred;
/* Effective (overridable) subjective task credentials (COW): */
const struct cred __rcu         *cred;
```
cred定义如下所示
```c++
struct cred {
......
kuid_t          uid;            /* real UID of the task */
kgid_t          gid;            /* real GID of the task */
kuid_t          suid;           /* saved UID of the task */
kgid_t          sgid;           /* saved GID of the task */
kuid_t          euid;           /* effective UID of the task */
kgid_t          egid;           /* effective GID of the task */
kuid_t          fsuid;          /* UID for VFS ops */
kgid_t          fsgid;          /* GID for VFS ops */
......
kernel_cap_t    cap_inheritable; /* caps our children can inherit */
kernel_cap_t    cap_permitted;  /* caps we're permitted */
kernel_cap_t    cap_effective;  /* caps we can actually use */
kernel_cap_t    cap_bset;       /* capability bounding set */
kernel_cap_t    cap_ambient;    /* Ambient capability set */
......
} __randomize_layout;
```
从这里的定义可以看出，大部分是关于用户和用户所属的用户组信息。

uid和 gid，注释是 real user/group id。一般情况下，谁启动的进程，就是谁的 ID。但是权限审核的时候，往往不比较这两个，也就是说不大起作用。
euid 和 egid，注释是 effective user/group id。一看这个名字，就知道这个是起“作用”的。当这个进程要操作消息队列、共享内存、信号量等对象的时候，其实就是在比较这个用户和组是否有权限。
fsuid 和fsgid，也就是 filesystem user/group id。这个是对文件操作会审核的权限。
在Linux中，我们可以通过chmod u+s program命令更改更改euid和fsuid来获取权限。

除了以用户和用户组控制权限，Linux 还有另一个机制就是 capabilities。

原来控制进程的权限，要么是高权限的 root 用户，要么是一般权限的普通用户，这时候的问题是，root 用户权限太大，而普通用户权限太小。有时候一个普通用户想做一点高权限的事情，必须给他整个 root 的权限。这个太不安全了。于是，引入新的机制 capabilities，用位图表示权限，在capability.h可以找到定义的权限。我这里列举几个。
```c++
#define CAP_CHOWN            0
#define CAP_KILL             5
#define CAP_NET_BIND_SERVICE 10
#define CAP_NET_RAW          13
#define CAP_SYS_MODULE       16
#define CAP_SYS_RAWIO        17
#define CAP_SYS_BOOT         22
#define CAP_SYS_TIME         25
#define CAP_AUDIT_READ          37
#define CAP_LAST_CAP         CAP_AUDIT_READ
```
对于普通用户运行的进程，当有这个权限的时候，就能做这些操作；没有的时候，就不能做，这样粒度要小很多。

### 5 运行统计
运行统计从宏观来说也是一种状态变量，但是和任务状态不同，其存储的主要是运行时间相关的成员变量，具体如下所示
```c++
u64        utime;//用户态消耗的CPU时间
u64        stime;//内核态消耗的CPU时间
unsigned long      nvcsw;//自愿(voluntary)上下文切换计数
unsigned long      nivcsw;//非自愿(involuntary)上下文切换计数
u64        start_time;//进程启动时间，不包含睡眠时间
u64        real_start_time;//进程启动时间，包含睡眠时间
2.6 进程调度
进程调度部分较为复杂，会单独拆分讲解，这里先简单罗列成员变量。

//是否在运行队列上
int        on_rq;
//优先级
int        prio;
int        static_prio;
int        normal_prio;
unsigned int      rt_priority;
//调度器类
const struct sched_class  *sched_class;
//调度实体
struct sched_entity    se;
struct sched_rt_entity    rt;
struct sched_dl_entity    dl;
//调度策略
unsigned int      policy;
//可以使用哪些CPU
int        nr_cpus_allowed;
cpumask_t      cpus_allowed;
struct sched_info    sched_info;
```
### 7 信号处理
信号处理相关的数据结构如下所示
```c++
/* Signal handlers: */
struct signal_struct    *signal;
struct sighand_struct    *sighand;
sigset_t      blocked;
sigset_t      real_blocked;
sigset_t      saved_sigmask;
struct sigpending    pending;
unsigned long      sas_ss_sp;
size_t        sas_ss_size;
unsigned int      sas_ss_flags;
```
这里将信号分为三类

1. 阻塞暂不处理的信号（blocked）
2. 等待处理的信号（pending）
3. 正在通过信号处理函数处理的信号（sighand）

信号处理函数默认使用用户态的函数栈，也可以开辟新的栈专门用于信号处理，这就是 sas_ss_xxx 这三个变量的作用。

### 8 内存管理
内存管理部分成员变量如下所示
```c++
struct mm_struct                *mm;
struct mm_struct                *active_mm;
```
### 9 文件与文件系统
```c++
/* Filesystem information: */
struct fs_struct                *fs;
/* Open file information: */
struct files_struct             *files;
```
### 10 内核栈
内核栈相关的成员变量如下所示。为了介绍清楚其作用，我们需要从为什么需要内核栈开始逐步讨论。
```c++
struct thread_info    thread_info;
void  *stack;
```
当进程产生系统调用时，会利用中断陷入内核态。而内核态中也存在着各种函数的调用，因此我们需要有内核态函数栈。Linux 给每个 task 都分配了内核栈。在 32 位系统上 arch/x86/include/asm/page_32_types.h，是这样定义的：一个 PAGE_SIZE是 4K，左移一位就是乘以 2，也就是 8K。
```c++
#define THREAD_SIZE_ORDER  1
#define THREAD_SIZE    (PAGE_SIZE << THREAD_SIZE_ORDER)
内核栈在 64 位系统上 arch/x86/include/asm/page_64_types.h，是这样定义的：在 PAGE_SIZE 的基础上左移两位，也即 16K，并且要求起始地址必须是 8192 的整数倍。

#ifdef CONFIG_KASAN
#define KASAN_STACK_ORDER 1
#else
#define KASAN_STACK_ORDER 0
#endif

#define THREAD_SIZE_ORDER  (2 + KASAN_STACK_ORDER)
#define THREAD_SIZE  (PAGE_SIZE << THREAD_SIZE_ORDER)
```
内核栈的结构首先是预留的8个字节，然后是存储寄存器，最后存储thread_info结构体。


这个结构是对 task_struct 结构的补充。因为 task_struct 结构庞大但是通用，不同的体系结构就需要保存不同的东西，所以往往与体系结构有关的，都放在 thread_info 里面。在内核代码里面采用一个 union将thread_info和stack 放在一起，在 include/linux/sched.h 中定义用以表示内核栈。由代码可见，这里根据架构不同可能采用旧版的task_struct直接放在内核栈，而新版的均采用thread_info，以节约空间。
```c++
union thread_union {
#ifndef CONFIG_ARCH_TASK_STRUCT_ON_STACK
    struct task_struct task;
#endif
#ifndef CONFIG_THREAD_INFO_IN_TASK
    struct thread_info thread_info;
#endif
    unsigned long stack[THREAD_SIZE/sizeof(long)];
};
```
另一个结构 pt_regs，定义如下。其中，32 位和 64 位的定义不一样。
```c++
#ifdef __i386__
struct pt_regs {
unsigned long bx;
unsigned long cx;
unsigned long dx;
unsigned long si;
unsigned long di;
unsigned long bp;
unsigned long ax;
unsigned long ds;
unsigned long es;
unsigned long fs;
unsigned long gs;
unsigned long orig_ax;
unsigned long ip;
unsigned long cs;
unsigned long flags;
unsigned long sp;
unsigned long ss;
};
#else
struct pt_regs {
unsigned long r15;
unsigned long r14;
unsigned long r13;
unsigned long r12;
unsigned long bp;
unsigned long bx;
unsigned long r11;
unsigned long r10;
unsigned long r9;
unsigned long r8;
unsigned long ax;
unsigned long cx;
unsigned long dx;
unsigned long si;
unsigned long di;
unsigned long orig_ax;
unsigned long ip;
unsigned long cs;
unsigned long flags;
unsigned long sp;
unsigned long ss;
/* top of stack page */
};
#endif
```
内核栈和task_struct是可以互相查找的，而这里就需要用到task_struct中的两个内核栈相关成员变量了。

