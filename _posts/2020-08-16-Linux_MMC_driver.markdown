---
title:  "Linux MMC 驱动子系统简述（源码剖析）"
date:   2020-08-16 09:00:00 +0800
categories: [Linux, driver]
tags: [Linux, mmc]
---

## 1. Linux MMC 驱动子系统

块设备是Linux系统中的基础外设之一，而MMC/SD存储设备是一种典型的块设备。Linux内核设计了MMC子系统，用于管理MMC/SD设备。

MMC子系统的框架结构如下图所示，其中`core layer`根据MMC/SD设备协议标准实现了协议。`card layer`与Linux的块设备子系统对接，实现块设备驱动以及完成请求，具体协议经过`core layer`的接口，最终通过`host layer`完成传输，对MMC设备进行实际的操作。和MMC设备硬件相对应，`host`和`card`可以分别理解为MMC device的两个子设备：MMC主设备和MMC从设备，其中`host`为集成于MMC设备内部的MMC controller，`card`为MMC设备内部实际的存储设备。  
![MMC Subsystem]({{ "/assets/img/sample/mmc_subsystem.svg"| relative_url }})

Linux系统中，使用两个结构体`struct mmc_host`和`struct mmc_card`分别描述`host`和`card`，其中`host`设备被封装成`platform_device`注册到Linux驱动模型中。整体而言，（Linux驱动模型框架下）MMC驱动子系统包括三个部分：  
+ MMC总线（`mmc_bus`）  
+ 封装在`platform_device`下的host设备  
+ 依附于MMC总线的MMC驱动（`mmc_driver`）  

下文将通过内核源码（Linux Kernel 5.2）对MMC驱动子系统进行简述，并通过MMC驱动的实际案例说明MMC驱动编写的一般步骤，同时分析驱动模型下完成驱动、设备绑定的过程。如对Linux设备驱动模型不熟悉，可以参考另一篇博文：[Linux设备驱动模型简述（源码剖析）](https://www.cnblogs.com/hueyxu/p/13659262.html)。  


## 2. MMC 总线的注册

MMC总线的注册和`platform`总线的注册方法相同，均是调用`bus_register()`函数。函数的调用入口位于`mmc/core/core.c`，通过`mmc_init()`实现，此处主要关注MMC的部分。  
```
/*  drivers/mmc/core/core.c  */
subsys_initcall(mmc_init);

static int __init mmc_init(void)
{
    int ret;

    ret = mmc_register_bus();
    if (ret)
        return ret;

    ret = mmc_register_host_class();
    if (ret)
        goto unregister_bus;

    ret = sdio_register_bus();
    if (ret)
        goto unregister_host_class;

    return 0;

unregister_host_class:
    mmc_unregister_host_class();
unregister_bus:
    mmc_unregister_bus();
    return ret;
}
```
```
/***********************************************************
 * mmc bus 总线注册
 ***********************************************************/
static struct bus_type mmc_bus_type = {
    .name       = "mmc",
    .dev_groups = mmc_dev_groups,
    .match      = mmc_bus_match,
    .uevent     = mmc_bus_uevent,
    .probe      = mmc_bus_probe,
    .remove     = mmc_bus_remove,
    .shutdown   = mmc_bus_shutdown,
    .pm         = &mmc_bus_pm_ops,
};

int mmc_register_bus(void)
{
    return bus_register(&mmc_bus_type);
}
```
```
/***********************************************************
 * mmc_host class 类注册
 ***********************************************************/
static struct class mmc_host_class = {
    .name           = "mmc_host",
    .dev_release    = mmc_host_classdev_release,
};

int mmc_register_host_class(void)
{
    return class_register(&mmc_host_class);
}
```

主要包括两个方面：  
+ 利用`bus_register()`注册`mmc_bus`。对应sysfs下的`/sys/bus/mmc/`目录。  
+ 利用`class_register()`注册`mmc_host_class`。对应sysfs下的`/sys/class/mmc_host`目录。  


## 3. MMC 驱动（mmc_driver）的注册

`drivers/mmc/core/block.c`中将`mmc_driver`注册到`mmc_bus`对应的总线系统里。主要步骤包括：  
+ 通过`register_blkdev()`向内核注册块设备。  
+ 调用`driver_register()`将`mmc_driver`注册到`mmc_bus`总线系统。和其他驱动注册方式一致。  

`mmc_driver`注册完成之后，会在sysfs中建立目录`/sys/bus/mmc/drivers/mmcblk`。  

```
/*  drivers/mmc/core/block.c  */

module_init(mmc_blk_init);

static struct mmc_driver mmc_driver = {
    .drv        = {
        .name   = "mmcblk",
        .pm     = &mmc_blk_pm_ops,
    },
    .probe      = mmc_blk_probe,
    .remove     = mmc_blk_remove,
    .shutdown   = mmc_blk_shutdown,
};

static int __init mmc_blk_init(void)
{
    int res;

    ...  ...
    res = register_blkdev(MMC_BLOCK_MAJOR, "mmc");
    ...

    res = mmc_register_driver(&mmc_driver);
    ...

    return 0;
    ... ...
}

/**
 *  mmc_register_driver - register a media driver
 *  @drv: MMC media driver
 */
int mmc_register_driver(struct mmc_driver *drv)
{
    drv->drv.bus = &mmc_bus_type;
    return driver_register(&drv->drv);
}
```

## 4. MMC 设备的注册

前文已经简单描述过，MMC设备主要包括主设备`host`和从设备`card`两部分，而主设备`host`将被封装在`platform_device`中注册到驱动模型中。  
为了更加清晰地描述此部分的注册过程，下文将以一个驱动为例分析（此驱动源码只包含关键步骤代码，只为描述MMC驱动的编写基本框架，[demo mmc driver](https://github.com/hughesxu/workspace/blob/master/linux_modules/mmc/klm_mmc.c))。  
```
module_init(xxx_mmc_init);

#define DRIVER_NAME "xxx_mmc"

/* platform driver definition */
static struct platform_driver xxx_mmc_driver = {
    .probe      = xxx_mmc_probe,
    .remove     = xxx_mmc_remove,
    .driver     = {
        .name   = DRIVER_NAME,
    },
};

static struct platform_device *pdev;
static __init int xxx_mmc_init(void)
{
    int err  = 0;

    /*
     * Register platform driver into driver model
     */
    // 将xxx_mmc_driver注册到驱动模型中
    err = platform_driver_register(&xxx_mmc_driver);

    /*
     * Allocate platform device and register into driver model
     * This will call driver->probe()
     */
    // 动态分配platform_device，并将其注册到驱动模型中
    // 此过程会回调driver->probe()函数
    pdev = platform_device_alloc(DRIVER_NAME, 0);
    err = platform_device_add(pdev);

    return err;
}
```
从代码中看到，驱动入口函数中将注册`platform_driver`和`platform_device`，`name`均定义为`xxx_mmc`。根据驱动模型，最终会回调`xxx_mmc_driver`中的`probe()`函数：`xxx_mmc_probe()`。  

### 4.1 xxx_mmc_probe(pdev)
```
// 自定义的mmc_host_ops，用于host做实际操作时回调
static const struct mmc_host_ops xxx_mmc_ops = {
    .request    = xxx_mmc_request,
    .set_ios    = xxx_mmc_set_ios,
};


/* platform driver probe function */
static int xxx_mmc_probe(struct platform_device *pdev)
{
    struct mmc_host *mmc;
    struct xxx_mmc_host *host = NULL;
    int ret = 0;

    /* Step 1: Allocate host structure */
    // 第1步：动态分配mmc_host结构
    mmc = mmc_alloc_host(sizeof(struct xxx_mmc_host), &pdev->dev);
    if (mmc == NULL) {
        pr_err("alloc host failed\n");
        ret = -ENOMEM;
        goto err_alloc_host;
    }

    // pointer initialization
    host = mmc_priv(mmc);
    host->mmc = mmc;

    host->id = pdev->id;

    /* Step 2: Initialize struct mmc_host */
    // 第2步：初始化mmc_host的结构成员
    mmc->ops        = &xxx_mmc_ops;
    mmc->f_min      = 400000;
    mmc->f_max      = 52000000;
    mmc->ocr_avail  = MMC_VDD_32_33;
    mmc->caps       = MMC_CAP_8_BIT_DATA |  MMC_CAP_NONREMOVABLE | MMC_CAP_MMC_HIGHSPEED;
    mmc->caps2      = MMC_CAP2_BOOTPART_NOACC | MMC_CAP_PANIC_WRITE;

    mmc->max_segs       = 1;
    mmc->max_blk_size   = 512;
    mmc->max_req_size   = 65536;                // Maximum number of bytes in one req
    mmc->max_blk_count  = mmc->max_req_size/mmc->max_blk_size;  // Maximum number of blocks in one req
    mmc->max_seg_size   = mmc->max_req_size;    // Segment size in one req, in bytes

    host->dev = &pdev->dev;

    platform_set_drvdata(pdev, host);       // pdev->dev->driver_data = host

    /* Step 3: Register the host with driver model */
    // 第3步：将mmc_host注册到驱动模型中
    mmc_add_host(mmc);

    return 0;

err_alloc_host:
    return ret;
}
```  
`xxx_mmc_probe(pdev)`主要工作如下：  
* 调用`mmc_alloc_host()`分配一个`struct mmc_host`结构。  
* 对`struct mmc_host`结构体成员初始化。  
* 调用`mmc_add_host()将`struct mmc_host`加入到驱动模型中。  


### 4.2 mmc_alloc_host(sizeof(struct xxx_mmc_host), &pdev->dev)
```
/*  drivers/mmc/core/host.c  */

static DEFINE_IDA(mmc_host_ida);

/*  mmc_alloc_host - initialise the per-host structure. */
struct mmc_host *mmc_alloc_host(int extra, struct device *dev)
{
    int err;
    struct mmc_host *host;

    host = kzalloc(sizeof(struct mmc_host) + extra, GFP_KERNEL);
    if (!host)
        return NULL;

    /* scanning will be enabled when we're ready */
    host->rescan_disable = 1;

    // Allocate an unused ID
    err = ida_simple_get(&mmc_host_ida, 0, 0, GFP_KERNEL);
    if (err < 0) {
        kfree(host);
        return NULL;
    }

    host->index = err;

    dev_set_name(&host->class_dev, "mmc%d", host->index);

    // 此处可以稍微注意一下，host->class_dev的类class设置为mmc_host_class
    // 并且host->class_dev的parent指向了pdev->dev (platform_device)
    // 这些许的差异会改变device_add后在sysfs中表现出来的层次结构
    host->parent = dev;                     // host->parent = &pdev->dev
    host->class_dev.parent = dev;           // host->class_dev.parent = &pdev->dev
    host->class_dev.class = &mmc_host_class; 

    // Initialize host->class_dev
    device_initialize(&host->class_dev);
    device_enable_async_suspend(&host->class_dev);

    if (mmc_gpio_alloc(host)) {
        put_device(&host->class_dev);
        return NULL;
    }

    // 初始化自旋锁
    spin_lock_init(&host->lock);
    // 初始化等待队列头
    init_waitqueue_head(&host->wq);
    // 初始化延迟的工作队列`host->detect`和`host->sdio_irq_work`
    INIT_DELAYED_WORK(&host->detect, mmc_rescan);
    INIT_DELAYED_WORK(&host->sdio_irq_work, sdio_irq_work);
    timer_setup(&host->retune_timer, mmc_retune_timer, 0);

    host->max_segs = 1;
    host->max_seg_size = PAGE_SIZE;

    host->max_req_size = PAGE_SIZE;
    host->max_blk_size = 512;
    host->max_blk_count = PAGE_SIZE / 512;

    host->fixed_drv_type = -EINVAL;
    host->ios.power_delay_ms = 10;

    return host;
}
```
主要工作如下：  
+ 动态分配内存给`struct mmc_host`结构体，并对结构体成员初始化。  
+ 调用`device_initialize()`对`host->class_dev`进行初始化，包括`kobject`、`mutex`等。  
+ 初始化自旋锁、等待队列(`waitqueue`)和延迟的工作队列(`Delayed Work`)，其中，用处理函数`mmc_rescan()`来初始化延迟的工作队列`host->detect`，后文会再次提到。  
+ 初始化定时器`host->retune_timer`，处理函数为`mmc_retune_timer()`。  

### 4.3 mmc_add_host(mmc)
在上述对`host`进行初始化后，调用`mmc_add_host()`将`host`注册到驱动模型中。  
```
/*  drivers/mmc/core/host.c  */

/*  mmc_add_host - initialise host hardware  */
int mmc_add_host(struct mmc_host *host)
{
    int err;

    WARN_ON((host->caps & MMC_CAP_SDIO_IRQ) &&
            !host->ops->enable_sdio_irq);

    err = device_add(&host->class_dev);
    if (err)
        return err;

    led_trigger_register_simple(dev_name(&host->class_dev), &host->led);

#ifdef CONFIG_DEBUG_FS
    mmc_add_host_debugfs(host);
#endif

    mmc_start_host(host);
    mmc_register_pm_notifier(host);

    return 0;
}
```
主要的工作包括两个部分：  
+ `device_add()`将`host->class_dev`加入到sysfs中`device`层次结构中。  
+ 调用`mmc_start_host()`启动主设备，也即MMC设备开始正常工作。
 
在介绍`mmc_start_host()`之前，先简单介绍下此处将`host->class_dev`加入到驱动模型后sysfs中表现出来的层次结构。 
4.2一节中介绍`mmc_alloc_host()`时注意到`host->class_dev`的`class`被设置为`mmc_host_class`，`parent`指向`pdev->dev`。`device_add()-->get_device_parent()/device_add_class_symlinks()`调用过程中将在`platform_device`的目录下多建立一个名称和`class name`相同的子文件夹，同时在`class`类目录下也会有指向实际设备的目录项。sysfs此时的结构如下：  
```
/sys/devices/platform/xxx_mmc.0/mmc_host/mmc0$ ll
total 0
... ...  root root    0 Sep 17 11:21 ./
... ...  root root    0 Sep 17 11:21 ../
... ...  root root    0 Sep 17 11:23 device -> ../../../xxx_mmc.0/
... ...  root root    0 Sep 17 11:23 power/
... ...  root root    0 Sep 17 11:23 subsystem -> ../../../../../class/mmc_host/
... ...  root root 4096 Sep 17 11:23 uevent

/sys/class/mmc_host$ ll        /sys/class/mmc_host
total 0
... ...  root root 0 Sep 17 11:21 ./
... ...  root root 0 Sep 17 11:09 ../
... ...  root root 0 Sep 17 11:26 mmc0 -> ../../devices/platform/xxx_mmc.0/mmc_host/mmc0/
```
此时`mmc_host`结构体成员初始化状态简要列举如下（详细可以按照驱动模型中`device`注册步骤推导）：  
```
struct mmc_host mmc {
    .rescan_disable    = 1     // set to 0 in mmc_start_host()

    .index         = id (allocated)
    .class_dev (struct dev)= {
        .p = {
            .device = &mmc.class_dev
                .klist_children = 
                .deferred_probe = 
        }
        . kobj = {
            .name = (“mmc%d”, .index)       // name = “mmc0”
            .kset = devices_kset        
            .ktype = &device_ktype
            INIT_LIST_HEAD(.dma_pools);
            mutex_init(.mutex);
            spin_lock_init(.devres_lock);
            INIT_LIST_HEAD(.devres_head);

            .parent = dir->kobject;  
                /*  /sys/devices/platform/xxx_mmc.0/mmc_host/mmc0  */
                //struct class_dir dir = {
                //    .class = &mmc_host_class
                //        .kobj = {
                //            .ktype = &class_dir_ktype
                //                .kset = & mmc_host_class->p->glue_dirs
                //                .parent = &pdev->dev->kobj      
                //                /*  /sys/devices/platform/xxx_mmc.0/   */

                //                .name = “mmc_host”
                //        }
                //}

        }
        .parent = &pdev->dev
            .class = &mmc_host_class = {
                .name       = "mmc_host",
                .dev_release    = mmc_host_classdev_release,
                .dev_kobj = sysfs_dev_char_kobj
                    .p (struct subsys_private) = {
                        .class = &mmc_host_class
                            . glue_dirs (kset) = {
                                .kobj = 
                                    .list = 
                                    .list_lock = 
                            }
                        . subsys (struct kset) = {
                            .kobj = {
                                .name = “mmc_host”
                                    .parent = & class_kset->kobj
                                    .kset  = class_kset;
                                .ktype = &class_ktype;

                                // create_dir(kobj)
                            }
                        }
                    }
            };
    }
    .parent = &pdev->dev

    spin_lock_init(&.lock);
    init_waitqueue_head(&.wq);
    INIT_DELAYED_WORK(&.detect, mmc_rescan);
    INIT_DELAYED_WORK(&.sdio_irq_work, sdio_irq_work);
    timer_setup(&.retune_timer, mmc_retune_timer, 0);

    .max_segs         = 1
        .max_seg_size  = PAGE_SIZE     //max segment size 8K
        .max_req_size  = PAGE_SIZE
        .max_blk_size  = 512
        .max_blk_count     = PAGE_SIZE / 512
        .fixed_drv_type    = -EINVAL
        .ios = {
            .power_delay_ms = 10
            .power_mode = MMC_POWER_UP      // mmc_start_host()
            .vdd = fls(ocr) – 1             // mmc_power_up(host, host->ocr_avail);
            .clock = .f_init

        }
    . ops = {
        .request    = xxx_mmc_request,
        .set_ios    = xxx_mmc_set_ios,
    }
    .f_min         = 400000
    .f_max         = 52000000
    .ocr_avail     = MMC_VDD_32_33 = 0x00100000
    .caps  = MMC_CAP_8_BIT_DATA |  MMC_CAP_NONREMOVABLE | MMC_CAP_MMC_HIGHSPEED;
    .caps2 = MMC_CAP2_BOOTPART_NOACC | MMC_CAP_PANIC_WRITE;
    .pm_notify.notifier_call = mmc_pm_notify
};
```

### 4.4 mmc_add_host(mmc)
```
/*  drivers/mmc/core/core.c  */

void mmc_start_host(struct mmc_host *host)
{
    host->f_init = max(freqs[0], host->f_min);
    host->rescan_disable = 0;
    host->ios.power_mode = MMC_POWER_UNDEFINED;

    if (!(host->caps2 & MMC_CAP2_NO_PRESCAN_POWERUP)) {
        mmc_claim_host(host);
        mmc_power_up(host, host->ocr_avail);
        mmc_release_host(host);
    }

    mmc_gpiod_request_cd_irq(host);
    _mmc_detect_change(host, 0, false);
}
```
+ `host->rescan_disable = 0`使能主设备的重新检测。  
+ 未使能`MMC_CAP2_NO_PRESCAN_POWERUP`时，将完成`claim_host()`、`power_up()`、`release_host()`等一系列工作。  
+ `mmc_gpiod_request_cd_irq()`用于为`host`申请中断号（和GPIO口对应），并绑定中断服务函数。  
+ `_mmc_detect_change(host, 0, false)`用于检测MMC槽位上的变动。  

`mmc_claim_host(host)`函数用于申请获得`host`（主控制器）的使用权，进程将进入休眠等待状态，直至可以获得主控制器的使用权。该函数结合`mmc_release_host(host)`，利用等待队列实现，原理细节可以参考：[Linux等待队列（Wait Queue）](https://www.cnblogs.com/hueyxu/p/13745029.html)。  


### 4.5 _mmc_detect_change(host, 0, false)
```
static void _mmc_detect_change(struct mmc_host *host, unsigned long delay, bool cd_irq)
{
    if (cd_irq && !(host->caps & MMC_CAP_NEEDS_POLL) &&
            device_can_wakeup(mmc_dev(host)))
        pm_wakeup_event(mmc_dev(host), 5000);

    host->detect_change = 1;
    mmc_schedule_delayed_work(&host->detect, delay);
}

static int mmc_schedule_delayed_work(struct delayed_work *work, unsigned long delay)
{
    return queue_delayed_work(system_freezable_wq, work, delay);
}
```
`_mmc_detect_change()`函数用来检测MMC状态的改变，具体是通过调度工作队列实现，如4.2一节介绍，`mmc_rescan()`作为处理函数被绑定在延迟工作队列`host->detect`上。因此，此处实际上是启动`mmc_rescan()`的执行过程。  

### 4.6 mmc_rescan(&host->detect)  
```
void mmc_rescan(struct work_struct *work)
{
    // 通过host->detect指针得到mmc_host结构体指针
    struct mmc_host *host = container_of(work, struct mmc_host, detect.work);

    // 如果rescan被禁止，函数提前返回
    if (host->rescan_disable)
        return;

    // 对于不可移除(non-removable)的host，如果其正在做rescan工作时，函数提前返回（scan只做一次）
    if (!mmc_card_is_removable(host) && host->rescan_entered)
        return;
    // 基本检查通过，进入rescan流程，标记rescan_entered
    host->rescan_entered = 1;

    ... ...
    // 检查可移除(removable) host是否还存在
    if (host->bus_ops && !host->bus_dead && mmc_card_is_removable(host))
        host->bus_ops->detect(host);

    host->detect_change = 0;
    ... ...

    // 尝试获得host的使用权，实现原理在4.4小节中有提及
    mmc_claim_host(host);
    ... ...

    // rescan流程的关键步骤，依次尝试四个给定频率，直至检测到mmc card的存在
    // static const unsigned freqs[] = { 400000, 300000, 200000, 100000 }
    for (i = 0; i < ARRAY_SIZE(freqs); i++) {
        if (!mmc_rescan_try_freq(host, max(freqs[i], host->f_min)))
            break;
        if (freqs[i] <= host->f_min)
            break;
    }
    mmc_release_host(host);
    ... ...
}
```
```
static int mmc_rescan_try_freq(struct mmc_host *host, unsigned freq)
{
    host->f_init = freq;
    // 完成一系列初始化步骤，保证设备在合适的运行状态，为后面实际探测做准备
    mmc_power_up(host, host->ocr_avail);
    mmc_hw_reset_for_init(host);
    ... ...
    mmc_go_idle(host);

    if (!(host->caps2 & MMC_CAP2_NO_SD))
        mmc_send_if_cond(host, host->ocr_avail);

    /* Order's important: probe SDIO, then SD, then MMC */
    // 依次探测设备：SDIO，SD，MMC
    // 对于MMC设备，尝试调用mmc_attach_mmc(host)
    if (!(host->caps2 & MMC_CAP2_NO_SDIO))
        if (!mmc_attach_sdio(host))
            return 0;

    if (!(host->caps2 & MMC_CAP2_NO_SD))
        if (!mmc_attach_sd(host))
            return 0;

    if (!(host->caps2 & MMC_CAP2_NO_MMC))
        if (!mmc_attach_mmc(host))
            return 0;

    mmc_power_off(host);
    return -EIO;
}
```
`mmc_rescan(&host->detect)`会调用`mmc_rescan_try_freq(host, max(freqs[i], host->f_min))`，进一步调用重要的`mmc_attach_mmc(host)`函数，将MMC设备加入到驱动模型中。  

### 4.7 mmc_attach_mmc(&host)  
```
// mmc_bus相关的一系列operations函数
static const struct mmc_bus_ops mmc_ops = {
    .remove = mmc_remove,
    .detect = mmc_detect,
    .suspend = mmc_suspend,
    .resume = mmc_resume,
    .runtime_suspend = mmc_runtime_suspend,
    .runtime_resume = mmc_runtime_resume,
    .alive = mmc_alive,
    .shutdown = mmc_shutdown,
    .hw_reset = _mmc_hw_reset,
};

// MMC card初始化的入口函数
int mmc_attach_mmc(struct mmc_host *host)
{
    int err;
    u32 ocr, rocr;

    // 首先检查host的使用权是否已经获得
    WARN_ON(!host->claimed);

    /* Set correct bus mode for MMC before attempting attach */
    if (!mmc_host_is_spi(host))
        mmc_set_bus_mode(host, MMC_BUSMODE_OPENDRAIN);
    // OCR register获得，可参考MMC设备操作规范
    err = mmc_send_op_cond(host, 0, &ocr);

    // 将host结构成员bus_ops设置为mmc_ops
    mmc_attach_bus(host, &mmc_ops);
    ... ... 
    // 为host选择合适的工作电压
    rocr = mmc_select_voltage(host, ocr);

    ... ... 
    // 关键步骤1：开始初始化MMC card的流程
    err = mmc_init_card(host, rocr, NULL);
    // MMC card初始化后释放host的使用权，起初在mmc_rescan()函数中获得
    mmc_release_host(host);
    // 关键步骤2：将MMC card注册进设备驱动模型中
    err = mmc_add_card(host->card);

    mmc_claim_host(host);
    return 0;
    ... ... 
}
```
`mmc_attach_mmc(&host)`作为MMC card检测和初始化的关键函数，执行的步骤可概括为：  
+ 获取MMC基本硬件初始化信息，例如OCR register （工作电压相关）  
+ 初始化host->bus_ops成员，`host->bus_ops = &mmc_ops`  
+ `mmc_init_card(host, rocr, NULL)`：MMC card初始化，下文详细介绍  
+ `mmc_add_card(host->card)`：将MMC card加入到设备驱动模型中，下文将详细介绍  


### 4.8 mmc_init_mmc(&host, rocr, NULL)  
```
static struct device_type mmc_type = {
    .groups = mmc_std_groups,
};

static int mmc_init_card(struct mmc_host *host, u32 ocr, struct mmc_card *oldcard)
{
    struct mmc_card *card;
    int err;
    u32 cid[4];
    u32 rocr;

    WARN_ON(!host->claimed);

    ... ...
    mmc_go_idle(host);

    /* The extra bit indicates that we support high capacity */
    err = mmc_send_op_cond(host, ocr | (1 << 30), &rocr);

    ... ... 
    err = mmc_send_cid(host, cid);

    if (oldcard) {
        ...
        card = oldcard;
    } else {
         // 动态分配mmc_card结构
        card = mmc_alloc_card(host, &mmc_type);

        card->ocr = ocr;
        card->type = MMC_TYPE_MMC;
        card->rca = 1;
        memcpy(card->raw_cid, cid, sizeof(card->raw_cid));
    }

    ... ...
    ... ...
}
```
```
struct mmc_card *mmc_alloc_card(struct mmc_host *host, struct device_type *type)
{
    struct mmc_card *card;

    card = kzalloc(sizeof(struct mmc_card), GFP_KERNEL);
    if (!card)
        return ERR_PTR(-ENOMEM);

    card->host = host;

    device_initialize(&card->dev);

    card->dev.parent = mmc_classdev(host);
    card->dev.bus = &mmc_bus_type;
    card->dev.release = mmc_release_card;
    card->dev.type = type;

    return card;
}
```
`mmc_init_mmc(&host, rocr, NULL)`会调用`mmc_alloc_card(host, &mmc_type)`动态分配一个`struct mmc_card`结构，并初始化其内部的`struct device`结构。此外，`mmc_init_mmc()`中还完成了许多初始化工作，这些工作大多是依据MMC操作规范定义的，在此不详细介绍。  
`struct mmc_card`结构成员大致列举如下，以方便分析。  
```
struct mmc_card card = {
    .host = &mmc;

    .dev = {
        .parent     = mmc_classdev(mmc) = &(mmc.class_dev);
        .bus        = &mmc_bus_type = {
            .name       = "mmc",
            .dev_groups = mmc_dev_groups,
            .match      = mmc_bus_match,
            .uevent     = mmc_bus_uevent,
            .probe      = mmc_bus_probe,
            .remove     = mmc_bus_remove,
            .shutdown   = mmc_bus_shutdown,
            .pm         = &mmc_bus_pm_ops,
        };

        .release    = mmc_release_card;
        .type       = &mmc_type = {
            .groups = mmc_std_groups,
        };
    }
    .ocr            = rocr;
    .type           = MMC_TYPE_MMC;
    .rca            = 1;                // relative card address of device
};
```


### 4.9 mmc_add_card(host->card)  
```
int mmc_add_card(struct mmc_card *card)
{
    int ret;

    ... ...
    // #define mmc_hostname(x) (dev_name(&(x)->class_dev))
    // 为card->dev设置名称“mmc0:0001”，实际上设置了card->dev->kobj.name = “mmc0:0001”
    dev_set_name(&card->dev, "%s:%04x", mmc_hostname(card->host), card->rca);

    ... ...
    card->dev.of_node = mmc_of_find_child_device(card->host, 0);

    device_enable_async_suspend(&card->dev);

    // 关键步骤：通过device_add() 将mmc card注册到设备驱动模型中
    ret = device_add(&card->dev);

    // 标记mmc card状态为PRESENT，card->state = MMC_STATE_PRESENT
    mmc_card_set_present(card);

    return 0;
}
```
这里最关键的一步是熟悉的`device_add(&card->dev)`函数，它将`mmc card`添加到驱动模型中。注意到`card.dev`的`parent`被设置为`mmc.class_dev`，所以将在前述`host`的目录层次下建立新的名为`mmc0:0001`的`card`子目录，而在调用`bus_add_device()`时，在`mmc bus`目录下子目录`devices`建立相应的链接，链接到上述`mmc0:0001`的设备目录上。sysfs的整体目录层次表现如下：  
```
/sys/devices/platform/xxx_mmc.0/mmc_host/mmc0/mmc0:0001$ ll
total 0
 ... ...  block/
 ... ...  
 ... ...  driver -> ../../../../../../bus/mmc/drivers/mmcblk/
 ... ...
 ... ...  subsystem -> ../../../../../../bus/mmc/
 ... ...
 ... ...  uevent

/sys/bus/mmc/devices$ ll
 ... ...  mmc0:0001 -> ../../../devices/platform/xxx_mmc.0/mmc_host/mmc0/mmc0:0001/


/sys/bus/mmc/drivers/mmcblk$ ll
 ... ...  mmc0:0001 -> ../../../../devices/platform/xxx_mmc.0/mmc_host/mmc0/mmc0:0001/

/sys/class/mmc_host$ ll
 ... ...  mmc0 -> ../../devices/platform/klm_emmc.0/mmc_host/mmc0/ 
```

按照Linux驱动模型，接下来要完成的是设备和驱动在总线上的匹配工作，最终调用驱动的`probe()`函数。调用的函数依次是：  
```
mmc_bus_probe(&card.dev) --> mmc_blk_probe(&card)
``` 


## 5. 块设备设备驱动
Linux块设备驱动初始化一般包括如下几个方面：  
+ 申请设备号，并将块设备驱动注册到内核。`register_blkdev()` 
+ 初始化请求队列。`blk_init_queue() / blk_alloc_queue()`  
+ 分配gendisk结构，并进行初始化。`alloc_disk()`  
+ 添加gendisk。`add_disk()`  

第3节mmc驱动注册中，函数`mmc_blk_init()`调用`register_blkdev(MMC_BLOCK_MAJOR, "mmc")`完成了设备号的申请，将块设备注册到内核中。下文将从`mmc_blk_probe(card)`入手分析其他几个步骤的实现。  

### 5.1 mmc_blk_probe(card)
```
struct mmc_blk_data {
    struct device   *parent;
    struct gendisk  *disk;
    struct mmc_queue queue;
    struct list_head part;
    struct list_head rpmbs;

    unsigned int    flags;

    unsigned int    usage;
    unsigned int    read_only;
    unsigned int    part_type;
    unsigned int    reset_done;
    ... ...
};
```
```
static int mmc_blk_probe(struct mmc_card *card)
{
    struct mmc_blk_data *md, *part_md;
    char cap_str[10];

    ... ...
    card->complete_wq = alloc_workqueue("mmc_complete",
            WQ_MEM_RECLAIM | WQ_HIGHPRI, 0);

    // 分配和初始化gendisk结构，初始化请求队列
    md = mmc_blk_alloc(card);

    string_get_size((u64)get_capacity(md->disk), 512, STRING_UNITS_2,
            cap_str, sizeof(cap_str));

    if (mmc_blk_alloc_parts(card, md))
        goto out;

    dev_set_drvdata(&card->dev, md);

    // 添加gendisk
    if (mmc_add_disk(md))
        goto out;

    list_for_each_entry(part_md, &md->part, part) {
        if (mmc_add_disk(part_md))
            goto out;
    }

    ...
    return 0;
    ...
}
```
`mmc_blk_probe(card)`为MMC card完成所有块设备驱动有关的工作，主要调用了两个重要的函数：  
+ `mmc_blk_alloc(card)`  
+ `mmc_add_disk(md)`  

### 5.2 mmc_blk_alloc(card)

`mmc_blk_alloc(card)`为MMC card初始化请求队列，函数调用过程为：  
```
mmc_blk_alloc(card) --> mmc_blk_alloc_req() --> mmc_init_queue(&md->queue, card) --> blk_mq_init_queue()
```
```
static struct mmc_blk_data *mmc_blk_alloc(struct mmc_card *card)
{
    sector_t size;

    if (!mmc_card_sd(card) && mmc_card_blockaddr(card)) {
        size = card->ext_csd.sectors;
    } else {
        size = (typeof(sector_t))card->csd.capacity << (card->csd.read_blkbits - 9);
    }

    return mmc_blk_alloc_req(card, &card->dev, size, false, NULL, MMC_BLK_DATA_AREA_MAIN);
}
```
```
static const struct block_device_operations mmc_bdops = {
    .open           = mmc_blk_open,
    .release        = mmc_blk_release,
    .getgeo         = mmc_blk_getgeo,
    .owner          = THIS_MODULE,
    .ioctl          = mmc_blk_ioctl,
#ifdef CONFIG_COMPAT
    .compat_ioctl       = mmc_blk_compat_ioctl,
#endif
};

static struct mmc_blk_data *mmc_blk_alloc_req(struct mmc_card *card,
        struct device *parent,  sector_t size,  bool default_ro,
        const char *subname,  int area_type)
{
    struct mmc_blk_data *md;
    int devidx, ret;

    devidx = ida_simple_get(&mmc_blk_ida, 0, max_devices, GFP_KERNEL);
    ... ...
    md = kzalloc(sizeof(struct mmc_blk_data), GFP_KERNEL);

    md->area_type = area_type;

    md->read_only = mmc_blk_readonly(card);

    // 分配gendisk结构 
    md->disk = alloc_disk(perdev_minors);

    INIT_LIST_HEAD(&md->part);
    INIT_LIST_HEAD(&md->rpmbs);
    md->usage = 1;

    // 初始化请求队列 
    ret = mmc_init_queue(&md->queue, card);

    md->queue.blkdata = md;

    // 初始化gendisk结构成员
    md->disk->major = MMC_BLOCK_MAJOR;
    md->disk->first_minor = devidx * perdev_minors;
    // 为MMC card定义了block_device_operations结构体
    md->disk->fops = &mmc_bdops;
    md->disk->private_data = md;
    md->disk->queue = md->queue.queue;
    md->parent = parent;
    set_disk_ro(md->disk, md->read_only || default_ro);
    md->disk->flags = GENHD_FL_EXT_DEVT;
    if (area_type & (MMC_BLK_DATA_AREA_RPMB | MMC_BLK_DATA_AREA_BOOT))
        md->disk->flags |= GENHD_FL_NO_PART_SCAN | GENHD_FL_SUPPRESS_PARTITION_INFO;

    snprintf(md->disk->disk_name, sizeof(md->disk->disk_name),
            "mmcblk%u%s", card->host->index, subname ? subname : "");

    set_capacity(md->disk, size);

    ... ...
    return md;
    ... ...
}
```
值得注意的是，`mmc_blk_alloc_req()`函数中为将MMC card gendisk结构体的fops赋值为`mmc_bdops`，即为MMC card定义了`open`，`release`，`getgeo`，`ioctl`等操作的回调函数。  

### 5.3 mmc_init_queue(&md->queue, card)
```
static const struct blk_mq_ops mmc_mq_ops = {
    .queue_rq       = mmc_mq_queue_rq,
    .init_request   = mmc_mq_init_request,
    .exit_request   = mmc_mq_exit_request,
    .complete       = mmc_blk_mq_complete,
    .timeout        = mmc_mq_timed_out,
};

int mmc_init_queue(struct mmc_queue *mq, struct mmc_card *card)
{
    struct mmc_host *host = card->host;
    int ret;

    mq->card = card;
    mq->use_cqe = host->cqe_enabled;

    spin_lock_init(&mq->lock);

    memset(&mq->tag_set, 0, sizeof(mq->tag_set));

    // mq->tag_set.ops设置为mmc_mq_ops
    mq->tag_set.ops = &mmc_mq_ops;
    ... ...
    mq->tag_set.driver_data = mq;

    ret = blk_mq_alloc_tag_set(&mq->tag_set);
    if (ret)
        return ret;

    // 初始化请求队列
    mq->queue = blk_mq_init_queue(&mq->tag_set);

    ... ...
    mq->queue->queuedata = mq;
    blk_queue_rq_timeout(mq->queue, 60 * HZ);

    mmc_setup_queue(mq, card);
    return 0;
    ... ...
}
```
```
struct request_queue *blk_mq_init_queue(struct blk_mq_tag_set *set)
{
    struct request_queue *uninit_q, *q;

    uninit_q = blk_alloc_queue_node(GFP_KERNEL, set->numa_node);
    q = blk_mq_init_allocated_queue(set, uninit_q);

    return q;
}
```
```
struct request_queue *blk_mq_init_allocated_queue(struct blk_mq_tag_set *set,
                          struct request_queue *q)
{
    // 用set->ops (mmc_mq_ops)来初始化request_queue的mq_ops成员
    q->mq_ops = set->ops;
    ... ...
    blk_queue_make_request(q, blk_mq_make_request);
    ... ...
}
```
`mmc_init_queue(&md->queue, card)`最终调用`blk_queue_make_request(q, blk_mq_make_request)`初始化了请求队列（制造请求函数）。  
另外，也使用`mmc_mq_ops`初始化了`request_queue`的`mq_ops`成员，下文会提到。  
至此，块设备驱动注册工作基本完成。  

## 6. 请求队列的工作流程梳理
4.1节介绍mmc设备初始化时提到，驱动编写过程中特别地为host编写了`mmc_host_ops`，而至目前仍未介绍到其被使用的地方，本节将从请求队列入手，通过一个简单的情形，分析`mmc_host_ops`操作函数的回调过程。

当对mmc card发起块设备I/O动作时，内核会首先调用到之前初始化的`blk_mq_make_request()`函数（定义在block/blk-mq.c），在特定情况下调用`blk_mq_try_issue_directly()`，最终一步步调用到`__blk_mq_issue_directly()`。  
```
static blk_qc_t blk_mq_make_request(struct request_queue *q, struct bio *bio)
{
    ...
    blk_mq_try_issue_directly(data.hctx, rq, &cookie);
    ..
}
``` 
```
static void blk_mq_try_issue_directly(struct blk_mq_hw_ctx *hctx,
        struct request *rq, blk_qc_t *cookie)
{
    ...
    ret = __blk_mq_try_issue_directly(hctx, rq, cookie, false, true);

    if (ret == BLK_STS_RESOURCE || ret == BLK_STS_DEV_RESOURCE)
        blk_mq_request_bypass_insert(rq, true);
    else if (ret != BLK_STS_OK)
        blk_mq_end_request(rq, ret);
    ...
}
```
```
static blk_status_t __blk_mq_try_issue_directly(struct blk_mq_hw_ctx *hctx,
        struct request *rq, blk_qc_t *cookie, bool bypass_insert, bool last)
{
    ... 
    return __blk_mq_issue_directly(hctx, rq, cookie, last);
    ...
}
```
```
static blk_status_t __blk_mq_issue_directly(struct blk_mq_hw_ctx *hctx,
        struct request *rq, blk_qc_t *cookie, bool last)
{
    struct request_queue *q = rq->q;
    ... 
    ret = q->mq_ops->queue_rq(hctx, &bd);
    ... 
    return ret;
}
```
5.3一节中提到`q->mq_ops`指向`mmc_mq_ops`，因此此处最终会回调函数`mmc_mq_ops->queue_rq()`，即`mmc_mq_queue_rq()`，最终回调至`mmc_host_ops->request(host, mrq)`。  
```
mmc_mq_queue_rq() --> mmc_blk_mq_issue_rq() --> mmc_blk_mq_issue_rw_rq() --> mmc_start_request() 
                  --> __mmc_start_request() --> host->ops->request(host, mrq)
```
```
static blk_status_t mmc_mq_queue_rq(struct blk_mq_hw_ctx *hctx,
                    const struct blk_mq_queue_data *bd)
{
    ...
    issued = mmc_blk_mq_issue_rq(mq, req);
    ...
}
```
```
enum mmc_issued mmc_blk_mq_issue_rq(struct mmc_queue *mq, struct request *req)
{
    ...
    ret = mmc_blk_mq_issue_rw_rq(mq, req);
    ...
}
```
```
static int mmc_blk_mq_issue_rw_rq(struct mmc_queue *mq,
                  struct request *req)
{
    struct mmc_queue_req *mqrq = req_to_mmc_queue_req(req);
    struct mmc_host *host = mq->card->host;
    ... ...

    err = mmc_start_request(host, &mqrq->brq.mrq);

    ... ...
    return err;
}
```
```
int mmc_start_request(struct mmc_host *host, struct mmc_request *mrq)
{
    int err;

    init_completion(&mrq->cmd_completion);

    mmc_retune_hold(host);

    ...
    WARN_ON(!host->claimed);

    err = mmc_mrq_prep(host, mrq);
    ...

    __mmc_start_request(host, mrq);

    return 0;
}
```
```
static void __mmc_start_request(struct mmc_host *host, struct mmc_request *mrq)
{
    int err;
    ...
    err = mmc_retune(host);
    ...
    host->ops->request(host, mrq);
}
```



## 7. 总结

综合上述分析，可以将MMC驱动子系统流程分析概括为下图。从驱动编写的角度，只需关注MMC card设备注册相关代码，主要包含如下几个方面：  
+ 将mmc host封装进一个`platform device`，同时定义一`platform driver`，并将它们注册到驱动模型中  
+ `platform driver`的`probe()`函数中调用`mmc_alloc_host()`和`mmc_add_host()`，将mmc card注册到驱动模型中  

![MMC subsystem Driver Model]({{ "/assets/img/sample/mmc_subsystem_driver_model.svg"| relative_url }})




## 参考资料
[1] Linux设备驱动开发详解（基于最新的Linux4.0内核），宋宝华编著，2016年  
[2] Linux SD/MMC/SDIO驱动分析：[https://www.cnblogs.com/cslunatic/p/3678045.html](https://www.cnblogs.com/cslunatic/p/3678045.html)  
