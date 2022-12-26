---
title:  "Linux升级GLIBC的血泪教训"
date:   2022-12-25 09:00:00 +0800
categories: [Linux, common]
tags: [Linux]
---

## 简述

开发环境的机器使用了Debian 9 GLIBC 2.24，而生产环境使用的是Debian 10 GLIBC 2.28，遂产生想法将开发环境也升级到GLIBC2.28，殊不知踏入大坑。  

起初幼稚地认为按照网上教程编译安装GLIBC库即可，便找了个网上教程：[https://www.cnblogs.com/beckyyyy/p/16911058.html](https://www.cnblogs.com/beckyyyy/p/16911058.html)，进行至`make install`一步时报错，然后机器上所有的命令执行时，都报`segmentation fault`。

参考了多篇博客的教程，仍为解决，最终只能无奈 重装系统，也积攒了血泪教训。  

GLIBC是Linux系统中最底层的API，最主要的功能是对系统调用进行了封装，几乎其他任何的运行库都要依赖glibc。因此，切勿擅自通过编译的方式升级，容易将系统搞坏。

升级glibc主要是对/lib库中的libc.so.6，libm.so.6， libpthread.so.0和librt.so.1这四个文件的修改。


## 参考资料

[https://blog.csdn.net/qq_42721097/article/details/120916145](https://blog.csdn.net/qq_42721097/article/details/120916145)
[记GLIBC升级失败后的恢复](https://blog.koko.vc/a/33/%E8%AE%B0GLIBC%E5%8D%87%E7%BA%A7%E5%A4%B1%E8%B4%A5%E5%90%8E%E7%9A%84%E6%81%A2%E5%A4%8D)
