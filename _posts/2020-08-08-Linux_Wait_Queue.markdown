---
title:  "Linux等待队列（Wait Queue）"
date:   2020-08-08 09:00:00 +0800
categories: [Linux, common]
tags: [Linux, waitqueue]
---

## 1. Linux等待队列概述

Linux内核的**等待队列（Wait Queue）**是重要的数据结构，与进程调度机制紧密相关联，可以用来同步对系统资源的访问、异步事件通知、跨进程通信等。  
在Linux中，等待队列以循环链表为基础结构，包括两种数据结构：**等待队列头(wait queue head)**和**等待队列元素(wait queue)**，整个等待队列由等待队列头进行管理。下文将用内核源码（基于Linux kernel 5.2）对等待队列进行介绍，详细说明采用等待队列实现进程阻塞和唤醒的方法。  

## 2. 等待队列头和等待队列元素
等待队列以循环链表为基础结构，链表头和链表项分别为**等待队列头**和**等待队列元素**，分别用结构体`wait_queue_head_t`和`wait_queue_entry_t`描述（定义在`linux/wait.h`）。  

### 2.1 基本概念
```
struct wait_queue_head {
    spinlock_t          lock;
    struct list_head    head;
};

typedef struct wait_queue_head wait_queue_head_t;
```
```
typedef int (*wait_queue_func_t)(struct wait_queue_entry *wq_entry, unsigned mode, int flags, void *key);
int default_wake_function(struct wait_queue_entry *wq_entry, unsigned mode, int flags, void *key);

/* wait_queue_entry::flags */
#define WQ_FLAG_EXCLUSIVE   0x01
#define WQ_FLAG_WOKEN       0x02
#define WQ_FLAG_BOOKMARK    0x04

/*
 * A single wait-queue entry structure:
 */
struct wait_queue_entry {
    unsigned int        flags;
    void                *private;
    wait_queue_func_t   func;
    struct list_head    entry;
};

typedef struct wait_queue_entry wait_queue_entry_t;
```
等待队列头结构包括一个自旋锁和一个链表头。等待队列元素除了包括链表项，还包括：  
+ `flags`: 标识队列元素状态和属性  
+ `*private`: 用于指向关联进程`task_struct`结构体的指针 
+ `func`: 函数指针，用于指向等待队列被唤醒时的回调的唤醒函数  

以进程阻塞和唤醒的过程为例，等待队列的使用场景可以简述为：  
进程A因等待某些资源（依赖进程B的某些操作）而不得不进入阻塞状态，便将当前进程加入到等待队列Q中。进程B在一系列操作后，可通知进程A所需资源已到位，便调用唤醒函数`wake up`来唤醒等待队列上Q的进程，注意此时所有等待在队列Q上的进程均被置为可运行状态。  
借助上述描述场景，说明等待队列元素属性`flags`标志的作用，下文也将结合源码进行详细解读。  
(1) `WQ_FLAG_EXCLUSIVE`  
上述场景中看到，当某进程调用`wake up`函数唤醒等待队列时，队列上所有的进程均被唤醒，在某些场合会出现唤醒的所有进程中，只有某个进程获得了期望的资源，而其他进程由于资源被占用不得不再次进入休眠。如果等待队列中进程数量庞大时，该行为将影响系统性能。  
内核增加了“独占等待” (`WQ_FLAG_EXCLUSIVE`)来解决此类问题。一个独占等待的行为和通常的休眠类似，但有如下两个重要的不同：  
+ 等待队列元素设置`WQ_FLAG_EXCLUSIVE`标志时，会被添加到等待队列的尾部，而非头部。  
+ 在某等待队列上调用`wake up`时，执行独占等待的进程每次只会唤醒其中第一个（所有非独占等待进程仍会被同时唤醒）。  

(2) `WQ_FLAG_WOKEN`  
暂时还未理解，TODO  

(3) `WQ_FLAG_BOOKMARK`  
用于`wake up`唤醒等待队列时实现分段遍历，减少单次对自旋锁的占用时间。  

### 2.2 等待队列的创建和初始化  
等待队列头的定义和初始化有两种方式：`init_waitqueue_head(&wq_head)`和宏定义`DECLARE_WAIT_QUEUE_HEAD(name)`。  
```
#define init_waitqueue_head(wq_head)                            \
    do {                                                        \
        static struct lock_class_key __key;                     \
        __init_waitqueue_head((wq_head), #wq_head, &__key);     \
    } while (0)

void __init_waitqueue_head(struct wait_queue_head *wq_head, const char *name, struct lock_class_key *key)
{
    spin_lock_init(&wq_head->lock);
    lockdep_set_class_and_name(&wq_head->lock, key, name);
    INIT_LIST_HEAD(&wq_head->head);
}
```
```
#define DECLARE_WAIT_QUEUE_HEAD(name)                       \
    struct wait_queue_head name = __WAIT_QUEUE_HEAD_INITIALIZER(name)

#define __WAIT_QUEUE_HEAD_INITIALIZER(name) {               \
    .lock       = __SPIN_LOCK_UNLOCKED(name.lock),          \
    .head       = { &(name).head, &(name).head } }
```

### 2.3 等待队列元素的创建和初始化  
创建等待队列元素较为普遍的一种方式是调用宏定义`DECLARE_WAITQUEUE(name, task)`，将定义一个名为`name`的等待队列元素，`private`数据指向给定的关联进程结构体`task`，唤醒函数为`default_wake_function()`。后文介绍唤醒细节时详细介绍唤醒函数的工作。  
```
#define DECLARE_WAITQUEUE(name, tsk)                        \
    struct wait_queue_entry name = __WAITQUEUE_INITIALIZER(name, tsk)

#define __WAITQUEUE_INITIALIZER(name, tsk) {                \
    .private    = tsk,                                      \
    .func       = default_wake_function,                    \
    .entry      = { NULL, NULL } }
```

内核源码中还存在其他定义等待队列元素的方式，调用宏定义`DEFINE_WAIT(name)`和`init_wait(&wait_queue)`。  
这两种方式都将**当前进程(current)**关联到所定义的等待队列上，唤醒函数为`autoremove_wake_function()`，注意此函数与上述宏定义方式时不同（上述定义中使用default_wake_function）。  
下文也将介绍`DEFINE_WAIT()`和`DECLARE_WAITQUEUE()`在使用场合上的不同。  
```
#define DEFINE_WAIT(name)   DEFINE_WAIT_FUNC(name, autoremove_wake_function)

#define DEFINE_WAIT_FUNC(name, function)                    \
    struct wait_queue_entry name = {                        \
        .private    = current,                              \
        .func       = function,                             \
        .entry      = LIST_HEAD_INIT((name).entry),         \
    }
```
```
#define init_wait(wait)                                     \
    do {                                                    \
        (wait)->private = current;                          \
        (wait)->func = autoremove_wake_function;            \
        INIT_LIST_HEAD(&(wait)->entry);                     \
        (wait)->flags = 0;                                  \
    } while (0)
```

### 2.4 添加和移除等待队列  
内核提供了两个函数（定义在`kernel/sched/wait.c`）用于将等待队列元素`wq_entry`添加到等待队列`wq_head`中：`add_wait_queue()`和`add_wait_queue_exclusive()`。  
+ `add_wait_queue()`：在等待队列头部添加普通的等待队列元素（非独占等待，清除`WQ_FLAG_EXCLUSIVE`标志）。  
+ `add_wait_queue_exclusive()`：在等待队列尾部添加独占等待队列元素（设置了`WQ_FLAG_EXCLUSIVE`标志）。  

```
void add_wait_queue(struct wait_queue_head *wq_head, struct wait_queue_entry *wq_entry)
{
    unsigned long flags;

    // 清除WQ_FLAG_EXCLUSIVE标志
    wq_entry->flags &= ~WQ_FLAG_EXCLUSIVE;
    spin_lock_irqsave(&wq_head->lock, flags);
    __add_wait_queue(wq_head, wq_entry);
    spin_unlock_irqrestore(&wq_head->lock, flags);
}   

static inline void __add_wait_queue(struct wait_queue_head *wq_head, struct wait_queue_entry *wq_entry)
{
    list_add(&wq_entry->entry, &wq_head->head);
}
```
```
void add_wait_queue_exclusive(struct wait_queue_head *wq_head, struct wait_queue_entry *wq_entry)
{
    unsigned long flags;

    // 设置WQ_FLAG_EXCLUSIVE标志
    wq_entry->flags |= WQ_FLAG_EXCLUSIVE;
    spin_lock_irqsave(&wq_head->lock, flags);
    __add_wait_queue_entry_tail(wq_head, wq_entry);
    spin_unlock_irqrestore(&wq_head->lock, flags);
}

static inline void __add_wait_queue_entry_tail(struct wait_queue_head *wq_head, struct wait_queue_entry *wq_entry)
{
    list_add_tail(&wq_entry->entry, &wq_head->head);
}
```

`remove_wait_queue()`函数用于将等待队列元素`wq_entry`从等待队列`wq_head`中移除。  
```
void remove_wait_queue(struct wait_queue_head *wq_head, struct wait_queue_entry *wq_entry)
{
    unsigned long flags;

    spin_lock_irqsave(&wq_head->lock, flags);
    __remove_wait_queue(wq_head, wq_entry);
    spin_unlock_irqrestore(&wq_head->lock, flags);
}

static inline void
__remove_wait_queue(struct wait_queue_head *wq_head, struct wait_queue_entry *wq_entry)
{
    list_del(&wq_entry->entry);
}
```
添加和移除等待队列的示意图如下所示：  
![Wait_Queue]({{ "/assets/img/sample/wait_queue.svg"| relative_url }})

## 3. 等待事件  
内核中提供了等待事件`wait_event()`宏（以及它的几个变种），可用于实现简单的进程休眠，等待直至某个条件成立，主要包括如下几个定义：  
```
wait_event(wq_head, condition)
wait_event_timeout(wq_head, condition, timeout) 
wait_event_interruptible(wq_head, condition)
wait_event_interruptible_timeout(wq_head, condition, timeout)
io_wait_event(wq_head, condition)
```
上述所有形式函数中，`wq_head`是等待队列头（采用”值传递“的方式传输函数），`condition`是任意一个布尔表达式。使用`wait_event`，进程将被置于非中断休眠，而使用`wait_event_interruptible`时，进程可以被信号中断。  
另外两个版本`wait_event_timeout`和`wait_event_interruptible_timeout`会使进程只等待限定的时间（以**jiffy**表示，给定时间到期时，宏均会返回0，而无论`condition`为何值）。  

详细介绍`wait_event`函数的实现原理。  
```
#define wait_event(wq_head, condition)                      \
    do {                                                    \
        might_sleep();                                      \
        // 如果condition满足，提前返回                       \
        if (condition)                                      \
           break;                                           \
        __wait_event(wq_head, condition);                   \
    } while (0)
 
#define __wait_event(wq_head, condition)                    \
     (void)___wait_event(wq_head, condition, TASK_UNINTERRUPTIBLE, 0, 0, schedule())


/* 定义等待队列元素，并将元素加入到等待队列中
 * 循环判断等待条件condition是否满足，若条件满足，或者接收到中断信号，等待结束，函数返回
 * 若condition满足，返回0；否则返回-ERESTARTSYS
 */
#define ___wait_event(wq_head, condition, state, exclusive, ret, cmd)       \
({                                                          \
     __label__ __out;                                       \
     struct wait_queue_entry __wq_entry;                    \
     long __ret = ret;          /* explicit shadow */       \
                                                            \
     // 初始化等待队列元素__wq_entry，关联当前进程，根据exclusive参数初始化属性标志 \
     // 唤醒函数为autoremove_wake_function()                                        \
     init_wait_entry(&__wq_entry, exclusive ? WQ_FLAG_EXCLUSIVE : 0);    \
     // 等待事件循环                                        \
     for (;;) {                                             \
        // 如果进程可被信号中断并且刚好有信号挂起，返回-ERESTARTSYS     \
        // 否则，将等待队列元素加入等待队列，并且设置进程状态，返回0    \
        long __int = prepare_to_wait_event(&wq_head, &__wq_entry, state);\
                                                            \
        // 当前进程让出调度器前，判断condition是否成立。若成立，提前结束，后续将返回0   \
        if (condition)                                      \
            break;                                          \
                                                            \
        // 当前进程让出调度器前，判断当前进程是否接收到中断信号（或KILL信号）       \
        // 如果成立，将提前返回-ERESTARTSYS                 \
        if (___wait_is_interruptible(state) && __int) {     \
            __ret = __int;                                  \
            goto __out;                                     \
        }                                                   \
                                                            \
        // 此处实际执行schedule()，当前进程让出调度器       \
        cmd;                                                \
     }                                                      \
     // 设置进程为可运行状态，并且将等待队列元素从等待队列中删除    \
     finish_wait(&wq_head, &__wq_entry);                    \
     __out:  __ret;                                         \
})  

void init_wait_entry(struct wait_queue_entry *wq_entry, int flags) 
{
    wq_entry->flags = flags;
    wq_entry->private = current;
    wq_entry->func = autoremove_wake_function;
    INIT_LIST_HEAD(&wq_entry->entry);
}

long prepare_to_wait_event(struct wait_queue_head *wq_head, struct wait_queue_entry *wq_entry, int state)
{
    unsigned long flags;
    long ret = 0;

    spin_lock_irqsave(&wq_head->lock, flags);
    // 返回非0值条件：可被信号中断并且确实有信号挂起
    if (signal_pending_state(state, current)) {
        // 将等待队列元素从等待队列中删除，返回-ERESTARTSYS
        list_del_init(&wq_entry->entry);
        ret = -ERESTARTSYS;
    } else {
        // 判断wq_entry->entry是否为空，即等待队列元素是否已经被添加到等待队列中
        if (list_empty(&wq_entry->entry)) {
            // WQ_FLAG_EXCLUSIVE标志被设置时，将等待队列元素添加到等待队列尾部（独占等待）
            // 否则，将等待队列元素添加到等待队列头部。同2.1中对WQ_FLAG_EXCLUSIVE标志介绍。
            if (wq_entry->flags & WQ_FLAG_EXCLUSIVE)
                __add_wait_queue_entry_tail(wq_head, wq_entry);
            else
                __add_wait_queue(wq_head, wq_entry);
        }
        // 改变当前进程的状态
        set_current_state(state);
    }
    spin_unlock_irqrestore(&wq_head->lock, flags);

    return ret;
}

// 用state_value改变当前的进程状态，并且执行了一次内存屏障
// 注意，只是改变了调度器处理该进程的方式，但尚未使该进程让出处理器
#define set_current_state(state_value)              \
    do {                            \
        WARN_ON_ONCE(is_special_task_state(state_value));\
        current->task_state_change = _THIS_IP_;     \
        smp_store_mb(current->state, (state_value));    \
    } while (0)

/*  设置进程为可运行状态，并且将等待队列元素从等待队列中删除  */
void finish_wait(struct wait_queue_head *wq_head, struct wait_queue_entry *wq_entry)
{
    unsigned long flags;
    // 将当前进程状态改为可运行状态(TASK_RUNNING)
    // 类似set_current_state()，差别在于未进行内存屏障
    __set_current_state(TASK_RUNNING);

    // 等待队列元素若仍在等待队列中，则将其删除
    if (!list_empty_careful(&wq_entry->entry)) {
        spin_lock_irqsave(&wq_head->lock, flags);
        list_del_init(&wq_entry->entry);
        spin_unlock_irqrestore(&wq_head->lock, flags);
    }
}
```
经过源码分析可以看到，`wait_event`使进程进入非中断休眠状态，循环等待直至特定条件满足，否则进程继续保持休眠状态。

可以简单总结出使用等待队列使进程休眠的一般步骤：  
+ 将当前进程关联的等待队列元素加入到等待队列中。`__add_wait_queue()`/`__add_wait_queue_entry_tail()`  
+ 设置当前进程状态（可中断TASK_INTERRUPTIBLE或不可中断TASK_UNINTERRUPTIBLE）。`set_current_state()`  
+ 判断资源是否得到，或是否捕获中断信号。  
+ 进程让出调度器，进入休眠状态。`schedule()`  
+ 资源得到满足时，将等待队列元素从等待队列中移除。  

## 4. 等待队列唤醒  
前文已经简单提到，`wake_up`函数可用于将等待队列上的所有进程唤醒，和`wait_event`相对应，`wake_up`函数也包括多个变体。主要包括：  
```
wake_up(&wq_head)
wake_up_interruptible(&wq_head)
wake_up_nr(&wq_head, nr)
wake_up_interruptible_nr(&wq_head, nr)
wake_up_interruptible_all(&wq_head)
```

### 4.1 wake_up()  

`wake_up`可以用来唤醒等待队列上的所有进程，而`wake_up_interruptible`只会唤醒那些执行可中断休眠的进程。因此约定，`wait_event`和`wake_up`搭配使用，而`wait_event_interruptible`和`wake_up_interruptible`搭配使用。  
前文提到，对于独占等待的进程，`wake_up`只会唤醒第一个独占等待进程。`wake_up_nr`函数提供功能，它能唤醒给定数目`nr`个独占等待进程，而不是只有一个。

`wake_up`函数的实现如下：  
```
#define TASK_NORMAL         (TASK_INTERRUPTIBLE | TASK_UNINTERRUPTIBLE)
// 可以看出wake_up函数将唤醒TASK_INTERRUPTIBLE和TASK_UNINTERRUPTIBLE的所有进程
#define wake_up(x)          __wake_up(x, TASK_NORMAL, 1, NULL)

void __wake_up(struct wait_queue_head *wq_head, unsigned int mode, int nr_exclusive, void *key)
{
    __wake_up_common_lock(wq_head, mode, nr_exclusive, 0, key);
}

static void __wake_up_common_lock(struct wait_queue_head *wq_head, unsigned int mode,
        int nr_exclusive, int wake_flags, void *key)
{
    unsigned long flags;
    wait_queue_entry_t bookmark;

    bookmark.flags = 0;
    bookmark.private = NULL;
    bookmark.func = NULL;
    INIT_LIST_HEAD(&bookmark.entry);

    // 第一次尝试调用__wake_up_common()，如果需要进行BOOKMARK过程，bookmark.flags会被置为WQ_FLAG_BOOKMARK
    spin_lock_irqsave(&wq_head->lock, flags);
    nr_exclusive = __wake_up_common(wq_head, mode, nr_exclusive, wake_flags, key, &bookmark);
    spin_unlock_irqrestore(&wq_head->lock, flags);

    // 如果还有需要处理的元素，那么bookmark.flags肯定置上WQ_FLAG_BOOKMARK；否则，在一个loop内便处理完成
    while (bookmark.flags & WQ_FLAG_BOOKMARK) {
        spin_lock_irqsave(&wq_head->lock, flags);
        nr_exclusive = __wake_up_common(wq_head, mode, nr_exclusive, wake_flags, key, &bookmark);
        spin_unlock_irqrestore(&wq_head->lock, flags);
    }
}

#define WAITQUEUE_WALK_BREAK_CNT 64

static int __wake_up_common(struct wait_queue_head *wq_head, unsigned int mode,
        int nr_exclusive, int wake_flags, void *key, wait_queue_entry_t *bookmark)
{
    wait_queue_entry_t *curr, *next;
    int cnt = 0;
    
    // 判断自旋锁已经被持有
    lockdep_assert_held(&wq_head->lock);

    // 如果bookmark元素中标志`WQ_FLAG_BOOKMARK`已被设置，则curr被设置为bookmark下一个元素
    // 同时将bookmark从等待队列中删除，bookmark->flags清零
    // 否则，curr设置为等待队列wq_head的第一个元素（实际上为第一次调用__wake_up_common）
    if (bookmark && (bookmark->flags & WQ_FLAG_BOOKMARK)) {
        curr = list_next_entry(bookmark, entry);

        list_del(&bookmark->entry);
        bookmark->flags = 0;
    } else
        curr = list_first_entry(&wq_head->head, wait_queue_entry_t, entry);

    if (&curr->entry == &wq_head->head)
        return nr_exclusive;

    // 在等待队列头指向的链表上，从curr指向的元素开始依次遍历元素
    list_for_each_entry_safe_from(curr, next, &wq_head->head, entry) {
        unsigned flags = curr->flags;
        int ret;

        // 跳过标记为WQ_FLAG_BOOKMARK的元素，等待队列元素被置上WQ_FLAG_BOOKMARK？
        if (flags & WQ_FLAG_BOOKMARK)
            continue;

        // 调用等待队列元素绑定的唤醒回调函数
        // 注意，具体唤醒何种进程(TASK_INTERRUPTIBLE/TASK_UNINTERRUPTIBLE)，作为参数传递给唤醒函数处理
        // 当进程不符合唤醒条件时，ret为0，详见try_to_wake_up()
        ret = curr->func(curr, mode, wake_flags, key);
        if (ret < 0)
            break;

        // 如果当前等待队列元素为独占等待，并且独占等待个数已经等于nr_exclusive，提前退出循环
        // 如2.1所述，独占等待进程被加入到等待队列的尾部，因此此时表明所有唤醒工作已经完成
        if (ret && (flags & WQ_FLAG_EXCLUSIVE) && !--nr_exclusive)
            break;
 
        // 连续唤醒的进程数目达到指定数目WAITQUEUE_WALK_BREAK_CNT（仍有进程元素需要处理），
        // 标记bookmark->flags为WQ_FLAG_BOOKMARK，同时将下一个要处理的元素添加到bookmark作为头节点的链表尾部，并退出遍历循环
        // 通过这种机制，实现了进程分批次唤醒，避免了等待队列中自旋锁被持有时间过长
        if (bookmark && (++cnt > WAITQUEUE_WALK_BREAK_CNT) &&
                (&next->entry != &wq_head->head)) {
            bookmark->flags = WQ_FLAG_BOOKMARK;
            list_add_tail(&bookmark->entry, &next->entry);
            break;
        }
    }

    return nr_exclusive;
}
```
`wake_up`函数会遍历等待队列上的所有元素（包括TASK_INTERRUPTIBLE和TASK_UNINTERRUPTIBLE)），根据`nr_exclusive`参数的要求唤醒进程，同时实现了分批次唤醒工作。最终会回调等待队列元素所绑定的唤醒函数。

前文已经提到，定义等待队列元素时主要涉及到两种唤醒回调函数：  
+ `default_wake_function()`：宏定义`DECLARE_WAITQUEUE(name, tsk)`使用的唤醒函数。  
+ `autoremove_wake_function()`：`DEFINE_WAIT(name)`，`init_wait(wait)`和`wait_event`中调用的`init_wait_entry()`使用此唤醒函数。   


### 4.2 default_wake_function() 
```
int default_wake_function(wait_queue_entry_t *curr, unsigned mode, int wake_flags, void *key)
{
    return try_to_wake_up(curr->private, mode, wake_flags);
}

static int
try_to_wake_up(struct task_struct *p, unsigned int state, int wake_flags)
{
    unsigned long flags;
    int cpu, success = 0;

    raw_spin_lock_irqsave(&p->pi_lock, flags);
    smp_mb__after_spinlock();
    // 此处对进程的状态进行筛选，跳过不符合状态的进程（TASK_INTERRUPTIBLE/TASK_UNINTERRUPTIBLE）
    if (!(p->state & state))
        goto out;

    trace_sched_waking(p);

    /* We're going to change ->state: */
    success = 1;
    cpu = task_cpu(p);

    smp_rmb();
    if (p->on_rq && ttwu_remote(p, wake_flags))
        goto stat;

    ... ...

    // Try-To-Wake-Up
    ttwu_queue(p, cpu, wake_flags);
stat:
    ttwu_stat(p, cpu, wake_flags);
out:
    raw_spin_unlock_irqrestore(&p->pi_lock, flags);

    return success;
}

static void ttwu_queue(struct task_struct *p, int cpu, int wake_flags)
{
    struct rq *rq = cpu_rq(cpu);
    struct rq_flags rf;

    ... ...
    rq_lock(rq, &rf);
    update_rq_clock(rq);
    ttwu_do_activate(rq, p, wake_flags, &rf);
    rq_unlock(rq, &rf);
}

static void
ttwu_do_activate(struct rq *rq, struct task_struct *p, int wake_flags,
        struct rq_flags *rf)
{
    int en_flags = ENQUEUE_WAKEUP | ENQUEUE_NOCLOCK;

    lockdep_assert_held(&rq->lock);

    ... ...
    activate_task(rq, p, en_flags);
    ttwu_do_wakeup(rq, p, wake_flags, rf);
}

/*
 * Mark the task runnable and perform wakeup-preemption.
 */
static void ttwu_do_wakeup(struct rq *rq, struct task_struct *p, int wake_flags,
        struct rq_flags *rf)
{
    check_preempt_curr(rq, p, wake_flags);
    p->state = TASK_RUNNING;
    trace_sched_wakeup(p);
    ... ...
}
``` 
从函数调用过程中可以看到，`default_wake_function()`实现唤醒进程的过程为：  
```
default_wake_function() --> try_to_wake_up() --> ttwu_queue() --> ttwu_do_activate() --> ttwu_do_wakeup()
```
值得一提的是，**`default_wake_function()`的实现中并未将等待队列元素从等待队列中删除**。因此，编写程序时不能忘记添加步骤将等待队列元素从等待队列元素中删除。  

### 4.3 autoremove_wake_function()  
```
int autoremove_wake_function(struct wait_queue_entry *wq_entry, unsigned mode, int sync, void *key)
{
    int ret = default_wake_function(wq_entry, mode, sync, key);

    if (ret)
        list_del_init(&wq_entry->entry);

    return ret;
}
```
`autoremove_wake_function()`相比于`default_wake_function()`，在成功执行进程唤醒工作后，会自动将等待队列元素从等待队列中移除。  

## 5. 源码实例
等待队列在内核中有着广泛的运用，此处以MMC驱动子系统中`mmc_claim_host()`和`mmc_release_host()`来说明等待队列的运用实例。  
`mmc_claim_host()`的功能为：借助等待队列申请获得MMC主控制器(host)的使用权，相对应，`mmc_release_host()`则是放弃host使用权，并唤醒所有等待队列上的进程。  
```
static inline void mmc_claim_host(struct mmc_host *host)
{
    __mmc_claim_host(host, NULL, NULL);
}

int __mmc_claim_host(struct mmc_host *host, struct mmc_ctx *ctx, atomic_t *abort)
{
    struct task_struct *task = ctx ? NULL : current;

    // 定义等待队列元素，关联当前进程，唤醒回调函数为default_wake_function()
    DECLARE_WAITQUEUE(wait, current);
    unsigned long flags;
    int stop;
    bool pm = false;

    might_sleep();

    // 将当前等待队列元素加入到等待队列host->wq中
    add_wait_queue(&host->wq, &wait);
    spin_lock_irqsave(&host->lock, flags);
    while (1) {
        // 当前进程状态设置为 TASK_UPINTERRUPTIBLE，此时仍未让出CPU
        set_current_state(TASK_UNINTERRUPTIBLE);
        stop = abort ? atomic_read(abort) : 0;
        // 真正让出CPU前判断等待的资源是否已经得到
        if (stop || !host->claimed || mmc_ctx_matches(host, ctx, task))
            break;
        spin_unlock_irqrestore(&host->lock, flags);
        // 调用调度器，让出CPU，当前进程可进入休眠
        schedule();
        spin_lock_irqsave(&host->lock, flags);
    }
    // 从休眠中恢复，设置当前进程状态为可运行(TASK_RUNNING)
    set_current_state(TASK_RUNNING);
    if (!stop) {
        host->claimed = 1;
        mmc_ctx_set_claimer(host, ctx, task);
        host->claim_cnt += 1;
        if (host->claim_cnt == 1)
            pm = true;
    } else
        // 可利用abort参数执行一次等待队列唤醒工作
        wake_up(&host->wq);
    spin_unlock_irqrestore(&host->lock, flags);

    // 等待队列结束，将等待队列元素从等待队列中移除
    remove_wait_queue(&host->wq, &wait);

    if (pm)
        pm_runtime_get_sync(mmc_dev(host));

    return stop;
}

void mmc_release_host(struct mmc_host *host)
{
    unsigned long flags;

    WARN_ON(!host->claimed);

    spin_lock_irqsave(&host->lock, flags);
    if (--host->claim_cnt) {
        /* Release for nested claim */
        spin_unlock_irqrestore(&host->lock, flags);
    } else {
        host->claimed = 0;
        host->claimer->task = NULL;
        host->claimer = NULL;
        spin_unlock_irqrestore(&host->lock, flags);

        // 唤醒等待队列host->wq上的所有进程
        wake_up(&host->wq);
        pm_runtime_mark_last_busy(mmc_dev(host));
        if (host->caps & MMC_CAP_SYNC_RUNTIME_PM)
            pm_runtime_put_sync_suspend(mmc_dev(host));
        else
            pm_runtime_put_autosuspend(mmc_dev(host));
    }
}
```
从源码实现过程可以看到，此实例中等待队列的使用和第3节中总结得基本过程一致，使用到的函数依次为：  
+ `DECLARE_WAITQUEUE(wait, current)`  
+ `add_wait_queue(&host->wq, &wait)`  
+ `set_current_state(TASK_UNINTERRUPTIBLE)`  
+ `schedule()`  
+ `set_current_state(TASK_RUNNING)`  
+ `remove_wait_queue(&host->wq, &wait)`  

## 6. 另一种休眠方式
回顾上文的介绍，2.3节中介绍了另外一种初始化等待队列元素的方式`DEFINE_WAIT`，而至目前仍未见使用。实际上此宏定义和另一个函数搭配使用：`prepare_to_wait()`。  
```
void
prepare_to_wait(struct wait_queue_head *wq_head, struct wait_queue_entry *wq_entry, int state)
{
    unsigned long flags;

    wq_entry->flags &= ~WQ_FLAG_EXCLUSIVE;
    spin_lock_irqsave(&wq_head->lock, flags);
    if (list_empty(&wq_entry->entry))
        __add_wait_queue(wq_head, wq_entry);
    set_current_state(state);
    spin_unlock_irqrestore(&wq_head->lock, flags);
}
```
可以看到`prepare_to_wait()`实际做的事情也就是将等待队列元素加入到等待队列中，然后更新当前进程状态。可以看出此过程依旧符合之前介绍的等待队列一般使用流程，只是内核源码将部分流程封装成为此函数。  
`prepare_to_wait()`配合`finish_wait()`函数可实现等待队列。  

## 7. 总结
综上文分析，等待队列的使用主要有三种方式：  
(1) **等待事件方式**  
`wait_event()`和`wake_up()`函数配合，实现进程阻塞睡眠和唤醒。  
(2) **手动休眠方式1**  
```
DECLARE_WAIT_QUEUE_HEAD(queue);
DECLARE_WAITQUEUE(wait, current);

for (;;) {
    add_wait_queue(&queue, &wait);
    set_current_state(TASK_INTERRUPTIBLE);
    if (condition)
        break;
    schedule();
    remove_wait_queue(&queue, &wait);
    if (signal_pending(current))
        return -ERESTARTSYS;
}
set_current_state(TASK_RUNNING);
remove_wait_queue(&queue, &wait);
```
(3) **手动休眠方式2（借助内核封装函数）**  
```
DELARE_WAIT_QUEUE_HEAD(queue);
DEFINE_WAIT(wait);

while (! condition) {
    prepare_to_wait(&queue, &wait, TASK_INTERRUPTIBLE);
    if (! condition)
        schedule();
    finish_wait(&queue, &wait)
}
```

## 参考资料
[1] LINUX 设备驱动程序（LDD3），2012年  
[2] Linux设备驱动开发详解（基于最新的Linux4.0内核），宋宝华编著，2016年  
[3] linux设备驱动模型：[https://blog.csdn.net/qq_40732350/article/details/82992904](https://blog.csdn.net/qq_40732350/article/details/82992904)  
[4] Linux 等待队列 (wait queue)：[https://xyfu.me/posts/236f51d8/](https://xyfu.me/posts/236f51d8/)  
[5] Linux Wait Queue 等待队列：[https://www.cnblogs.com/gctech/p/6872301.html](https://www.cnblogs.com/gctech/p/6872301.html)  
[6] 源码解读Linux等待队列：[http://gityuan.com/2018/12/02/linux-wait-queue/](http://gityuan.com/2018/12/02/linux-wait-queue/)  
[7] Driver porting: sleeping and waking up：[https://lwn.net/Articles/22913/](https://lwn.net/Articles/22913/)  
