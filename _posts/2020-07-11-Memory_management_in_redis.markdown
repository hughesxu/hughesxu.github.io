---
title:  "Redis源码笔记-内存管理zmalloc.c"
date:   2020-07-12 09:00:00 +0800
categories: [Redis, zmalloc]
tags: [Redis]
---

## zmalloc.h / zmalloc.c

Redis源代码中关于内存管理相关的函数定义于`zmalloc.c`文件中，主要函数罗列如下。这里对`malloc/free`等相关函数的实现进行分析。  
参考：[博客园](https://www.cnblogs.com/bush2582/p/8969000.html)

 
```
void *zmalloc(size_t size);
void *zcalloc(size_t size);
void *zrealloc(void *ptr, size_t size);
void zfree(void *ptr);
char *zstrdup(const char *s);
size_t zmalloc_used_memory(void);
void zmalloc_set_oom_handler(void (*oom_handler)(size_t));
size_t zmalloc_get_rss(void);
int zmalloc_get_allocator_info(size_t *allocated, size_t *active, size_t *resident);
void set_jemalloc_bg_thread(int enable);
int jemalloc_purge();
size_t zmalloc_get_private_dirty(long pid);
size_t zmalloc_get_smap_bytes_by_field(char *field, long pid);
size_t zmalloc_get_memory_size(void);
void zlibc_free(void *ptr);
```

### 内存分配器Jemalloc / Tcmalloc / malloc

首先，头文件`zmalloc.h`根据平台和宏定义选择内存管理相关的函数和头文件，涉及到的几种内存分配器，包括：  
* `jemalloc`: 通用的`malloc(3)`实现，最大的优势在于多线程情况下的高性能以及内存碎片的减少。  
* `tcmalloc`: 全称`Thread-Caching Malloc`，即线程缓存的`malloc`，是`google-perftools`的一部分，实现了高效的多线程内存管理。  
* `malloc`: `GNU C`中的库函数。

Linux环境下编译情况下，从`Makefile`可以看出是默认使用`jemalloc`的，编译时加上的宏定义为`-DUSE_JEMALLOC`。  
内存分配器本身通常是能保证分配的内存是字节对齐的，如果需要分配的内存大小不满足内存对齐，则分配分配器再多分配几个填充字节(`Padding Bytes`)保证字节对齐。另外，部分内存分配器在分配内存的时候也会多分配一部分内存用于存储分配空间的大小，此部分空间通常为内存分配器返回地址的前几个字节（4/8 Bytes），空间示意图如下图所示。(借助于此，`free`函数也便能得到需要释放空间大小)  
![malloc memory]({{ "/assets/img/sample/malloc_memory.jpg"| relative_url }})


`jemalloc/tcmalloc/malloc(GNU C)`函数都提供了计算已分配内存大小函数，所以不需要单独分配空间来存储已分配空间大小，而对于其他不具有此类函数的方法，需要额外分配一段空间，长度为`PREFIX_SIZE`。当使用相应的内存分配器时，用对应的内存分配器函数覆盖`malloc/free`函数即可。总结各种情况下的需要包含的头文件以及宏定义，列举如下：  
```
#if defined(USE_TCMALLOC)
    #include <google/tcmalloc.h> 
    #define HAVE_MALLOC_SIZE    1
    #define PREFIX_SIZE         (0)
    #define zmalloc_size(p)     tc_malloc_size(p)
    /***    Override malloc/free etc    ***/
    #define malloc(size)        tc_malloc(size)
    #define calloc(count,size)  tc_calloc(count,size)
    #define realloc(ptr,size)   tc_realloc(ptr,size)
    #define free(ptr)           tc_free(ptr)

#elif defined(USE_JEMALLOC)
    #include <jemalloc/jemalloc.h>
    #define HAVE_MALLOC_SIZE    1
    #define PREFIX_SIZE         (0)
    #define zmalloc_size(p)     je_malloc_usable_size(p)
    #define malloc(size)        je_malloc(size)
    /***    Override malloc/free etc    ***/
    #define calloc(count,size)  je_calloc(count,size)
    #define realloc(ptr,size)   je_realloc(ptr,size)
    #define free(ptr)           je_free(ptr)
    #define mallocx(size,flags) je_mallocx(size,flags)
    #define dallocx(ptr,flags)  je_dallocx(ptr,flags)

#elif defined(__APPLE__)        // IOS
    #include <malloc/malloc.h>
    #define HAVE_MALLOC_SIZE    1
    #define PREFIX_SIZE         (0)
    #define zmalloc_size(p)     malloc_size(p)

#ifdef __GLIBC__
    #include <malloc.h>
    #define HAVE_MALLOC_SIZE    1
    #define zmalloc_size(p)     malloc_usable_size(p)

/*** HAVE_MALLOC_SIZE and PREFIX_SIZE ***/
#ifdef HAVE_MALLOC_SIZE
    #define PREFIX_SIZE         0
#else
    #if defined(__sun) || defined(__sparc) || defined(__sparc__)
        #define PREFIX_SIZE     (sizeof(long long))
    #else
        #define PREFIX_SIZE     (sizeof(size_t))
    #endif
```

### zmalloc()

自定义的`zmalloc`函数根据内存分配器是否提供计算分配空间大小函数，有两个程序分支：  
* `defined(HAVE_MALLOC_SIZE)`:   
使用内存分配器自带的`zmalloc_size`函数(已被覆盖为内存分配器对应的函数)完成分配空间大小的计算。对于Linux系统而言，`zmalloc_size`对应`malloc_usable_size`，此函数返回值不包含首部长度。  
* `not defined(HAVE_MALLOC_SIZE)`:   
多分配`PREFIX_SIZE`(对于64位机器为8字节)的内存用于分配空间的存储。值得注意的是，此时记录的分配空间大小不包含`PREFIX_SIZE`和用于字节对齐的空间。内存分配的示意图如下图所示。  
![zmalloc memory]({{ "/assets/img/sample/zmalloc_memory.jpg"| relative_url }})

`update_zmalloc_stat_alloc()`函数的作用是，获得真正分配的内存空间大小，并更新全局变量`used_memory`。此处，考虑到了`malloc()`中存在的内存对齐行为，所以将内存大小圆整到`sizeof(long)`的值。

```
#define update_zmalloc_stat_alloc(__n) do { \
    size_t _n = (__n); \
    if (_n&(sizeof(long)-1)) _n += sizeof(long)-(_n&(sizeof(long)-1)); \
    atomicIncr(used_memory,__n); \
} while(0)

void *zmalloc(size_t size) {
    void *ptr = malloc(size+PREFIX_SIZE);

    if (!ptr) zmalloc_oom_handler(size);
#ifdef HAVE_MALLOC_SIZE
    update_zmalloc_stat_alloc(zmalloc_size(ptr));
    return ptr;
#else
    *((size_t*)ptr) = size;
    update_zmalloc_stat_alloc(size+PREFIX_SIZE);
    return (char*)ptr+PREFIX_SIZE;
#endif
}
```

### zmalloc_size() and zmalloc_usable()

针对内存分配器不提供计算分配空间大小函数的情况，自定义了`zmalloc_size`函数。注意此处，作为函数参数传入的指针`ptr`指向指向实际分配的可使用内存的起始地址，已经在首部地址上偏移了首部的长度。函数返回值同样考虑到内存对齐，因此返回的内存分配大小为：  
&emsp;`PREFIX_SIZE + size + Padding_bytes_size`。

`zmalloc_usable()`函数返回分配空间中可用的大小，即将首部`PREFIX_SIZE`排除在外。

```
/* Provide zmalloc_size() for systems where this function is not provided by
 * malloc itself, given that in that case we store a header with this
 * information as the first bytes of every allocation. */
#ifndef HAVE_MALLOC_SIZE
size_t zmalloc_size(void *ptr) {
    void *realptr = (char*)ptr-PREFIX_SIZE;
    size_t size   = *((size_t*)realptr);
    /* Assume at least that all the allocations are padded at sizeof(long) by
     * the underlying allocator. */
    if (size&(sizeof(long)-1)) size += sizeof(long)-(size&(sizeof(long)-1));
    return size+PREFIX_SIZE;
}

size_t zmalloc_usable(void *ptr) {
    return zmalloc_size(ptr)-PREFIX_SIZE;
}
#endif
```


### zfree()

`zfree`函数和`zmalloc`函数对应，用于将分配的内存空间释放。同样根据实际选择的内存分配器分为了两个程序分支：  
* `defined(HAVE_MALLOC_SIZE)`:   
&emsp;使用内存分配器自带的`zmalloc_size`函数获得分配的内存空间大小，传入`update_zmalloc_stat_free()`更新全局变量`used_memory`。  
* `not defined(HAVE_MALLOC_SIZE)`:   
&emsp;根据传入的指针`ptr`和已知首部偏移`PREFIX_SIZE`计算出首部的起始地址，也即整个分配空间的起始地址（参考上述分配过程`zmalloc()`）。同样更新了全局变量`used_memory`。
 
```
#define update_zmalloc_stat_free(__n) do { \
    size_t _n = (__n); \
    if (_n&(sizeof(long)-1)) _n += sizeof(long)-(_n&(sizeof(long)-1)); \
    atomicDecr(used_memory,__n); \
} while(0)


void zfree(void *ptr) {
#ifndef HAVE_MALLOC_SIZE
    void *realptr;
    size_t oldsize;
#endif

    if (ptr == NULL) return;
#ifdef HAVE_MALLOC_SIZE
    update_zmalloc_stat_free(zmalloc_size(ptr));
    free(ptr);
#else
    realptr = (char*)ptr-PREFIX_SIZE;
    oldsize = *((size_t*)realptr);
    update_zmalloc_stat_free(oldsize+PREFIX_SIZE);
    free(realptr);
#endif
}
```

### zmalloc_get_rss()

`zmalloc_get_rss()`用来获得redis进程的`RSS`信息，`RSS`信息是指进程实际使用的内存大小。对于linux系统而言，通过文件系统中`/proc/pid/stat`文件获得，此文件也是`ps`命令获取进程信息的来源，该文件中`rss`信息表述为内存页的数量，具体如下： 
```
rss  %ld
    Resident Set Size: number of pages the process has
    in real memory.  This is just the pages which count
    toward text, data, or stack space.  This does not
    include pages which have not been demand-loaded in,
    or which are swapped out.
``` 

`zmalloc_get_rss()`通过读取`proc/pid/stat`文件中的第`24`个字段，与系统运行时内存页大小相乘，获得实际使用的内存大小，单位为`KBytes`。
