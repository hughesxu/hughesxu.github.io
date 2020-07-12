---
title:  "Redis源码笔记-Linux 环境下自定义的宏定义"
date:   2020-07-11 20:00:00 +0800
categories: [Redis, misc]
tags: [Redis]
---

## Linux环境下Redis源代码中的宏定义

Redis源代码中借助自定义宏定义完成了许多程序分支的分离，在开发的Linux环境下列举出当前环境下自定义的宏定义，以供分析时查询。Linux环境下主要用`gcc`编译，因此借助`gcc -dM -E - < /dev/null`命令获得相应的变量，大部分的自定义定义在`config.h`中定义。  
```
#include <linux/version.h>
#include <features.h>

#define redis_fstat fstat
#define redis_stat stat

#define HAVE_PROC_STAT 1
#define HAVE_PROC_MAPS 1
#define HAVE_PROC_SMAPS 1
#define HAVE_PROC_SOMAXCONN 1

#define HAVE_MSG_NOSIGNAL 1
#define HAVE_EPOLL 1

#define redis_fsync fdatasync
#define HAVE_SYNC_FILE_RANGE 1

#define rdb_fsync_range(fd,off,size) sync_file_range(fd,off,size,SYNC_FILE_RANGE_WAIT_BEFORE|SYNC_FILE_RANGE_WRITE)

#include <sys/types.h> /* This will likely define BYTE_ORDER */

# include <endian.h>

#define BYTE_ORDER    LITTLE_ENDIAN

#define GNUC_VERSION (__GNUC__ * 10000 + __GNUC_MINOR__ * 100 + __GNUC_PATCHLEVEL__)

#define redis_set_thread_title(name) pthread_setname_np(pthread_self(), name)

#define USE_SETCPUAFFINITY
void setcpuaffinity(const char *cpulist);
```
