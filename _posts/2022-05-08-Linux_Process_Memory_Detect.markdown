---
title:  "Linux进程内存问题排查"
date:   2022-05-08 09:00:00 +0800
categories: [Linux, common]
tags: [Linux]
---

## 0 前言

本文主要记录Linux环境进程内存问题的排查方法。  


## 1 内存泄漏排查

Linux环境进程内存泄漏通常使用Valgrind工具进行排查，使用的命令如下：  
```
valgrind --trace-children=yes --show-reachable=yes --leak-check=full
--show-leak-kinds=all --trake-origins=yes your_prog
The correct way to log is to use the option, "--log-file=filename"
```


Memcheck is the memory error detector tool in Valgrind, to use it, you "may" specify the option "--tool=memcheck", but you don't need to since Memcheck is the default tool of Valgrind.  

https://prajankya.me/valgrind-on-linux/  
http://senlinzhan.github.io/2017/12/31/valgrind/  
valgrind 官方网站：https://valgrind.org/docs/manual/index.html  



https://valgrind.org/docs/manual/faq.html  
```
Is it possible to attach Valgrind to a program that is already running?
No. The environment that Valgrind provides for running programs is 
significantly different to that for normal programs, e.g. due to 
different layout of memory. Therefore Valgrind has to have full 
control from the very start.  

It is possible to achieve something like this by running 
your program without any instrumentation (which involves a 
slow-down of about 5x, less than that of most tools), 
and then adding instrumentation once you get to a point 
of interest. Support for this must be provided by the tool, 
 however, and Callgrind is the only tool that currently has such support. 
 See the instructions on the callgrind_control program for details.
```



## 2 进程运行时内存检查

https://unix.stackexchange.com/questions/36450/how-can-i-find-a-memory-leak-of-a-running-process

- Find out the PID of the process which causing memory leak.

```
ps -aux
```

- capture the `/proc/PID/smaps` and save into some file.

- wait till memory gets increased.
- capture again `/proc/PID/smaps` and save it  
find the difference between first smaps and 2nd smaps

- note down the address range where memory got increased

- use GDB to dump memory on running process or get the coredump using gcore -o process
```
gdb -p PID
dump memory ./dump_outputfile.dump 0x2b3289290000 0x2b3289343000
```
- now, use `strings` command or `hexdump -C` to print the `dump_outputfile.dump`
```
strings outputfile.dump
```
- You get readable form where you can locate those strings into your source code. Analyze your source to find the leak.   


### RES内存

https://stackoverflow.com/questions/23077525/resident-memory-increase-while-valgrind-not-showing-any-leaks
```
The heap is part of your "res" memory, so if you have something
that allocates x MB of heap memory, then releases it, unless
the OS actually need that memory for other purposes, it will
remain as part of your applications memory. 
(Actually, it's quite a bit more complex than that, 
but for this discussion, this picture is valid).
```


## 3 进程内核调用检查strace

https://wangchujiang.com/linux-command/c/strace.html

