---
title:  "TCP KeepAlive机制理解与实践小结"
date:   2021-12-25 09:00:00 +0800
categories: [Network, TCP]
tags: [TCP]
---

## 0 前言

本文将主要通过抓包并查看报文的方式学习TCP KeepAlive机制，以此加深理解。


## 1 TCP KeepAlive机制简介

TCP长连接下，客户端和服务器若长时间无数据交互情况下，若一方出现异常情况关闭连接，抑或是连接中间路由出于某种机制断开连接，而此时另一方不知道对方状态而一直维护连接，浪费系统资源的同时，也会引起下次数据交互时出错。

为了解决此问题，引入了TCP KeepAlive机制（并非标准规范，但操作系统一旦实现，默认情况下须为关闭，可以被上层应用开启和关闭）。其基本原理是在此机制开启时，当长连接无数据交互一定时间间隔时，连接的一方会向对方发送保活探测包，如连接仍正常，对方将对此确认回应。

关于TCP KeepAlive机制的详细背景可以参考《TCP/IP详解 卷1:协议》一书，在此不详细赘述。



## 2 TCP KeepAlive设置参数和报文格式简介

### 2.1 TCP KeepAlive参数
TCP KeepAlive机制主要涉及3个参数：

- **tcp_keepalive_time** (integer; default: 7200; since Linux 2.2)  
`The number of seconds a connection needs to be idle before TCP begins sending out keep-alive probes. Keep-alives  are  sent only when the SO_KEEPALIVE socket option is enabled. The default value is 7200 seconds (2 hours). An idle connection is terminated after approximately an additional 11 minutes (9 probes an interval of 75 seconds apart) when keep-alive is enabled.`  
在TCP保活打开的情况下，最后一次数据交换到TCP发送第一个保活探测包的间隔，即允许的持续空闲时长，或者说每次正常发送心跳的周期，默认值为7200s（2h）。

- **tcp_keepalive_probes** (integer; default: 9; since Linux 2.2)  
`The  maximum  number of TCP  keep-alive probes to send before giving up and killing the connection if no response is obtained from the other end.`  
在tcp_keepalive_time之后，最大允许发送保活探测包的次数，到达此次数后直接放弃尝试，并关闭连接，默认值为9（次）。

- **tcp_keepalive_intvl** (integer; default: 75; since Linux 2.4)  
  `The number of seconds between TCP keep-alive probes.`  
在tcp_keepalive_time之后，没有接收到对方确认，继续发送保活探测包的发送频率，默认值为75s。

### 2.2 TCP KeepAlive报文格式

TCP KeepAlive探测报文是一种没有任何数据，同时ACK标志被置上的报文，报文中的序列号为上次发生数据交互时TCP报文序列号减1。比如上次本端和对端数据交互的最后时刻，对端回应给本端的ACK报文序列号为 N（即下次本端向对端发送数据，序列号应该为N），则本端向对端发送的保活探测报文序列号应该为 N-1。
在本文第四节，将通过抓包的方式再次介绍TCP KeepAlive报文。



## 3 TCP KeepAlive的配置

### 3.1 系统内核参数配置
Linux内核提供了通过sysctl命令查看和配置TCP KeepAlive参数的方法。
+ 查看当前内核TCP KeepAlive参数  
```
sysctl net.ipv4.tcp_keepalive_time
sysctl net.ipv4.tcp_keepalive_probes
sysctl net.ipv4.tcp_keepalive_intvl
```
+ 修改TCP KeepAlive参数  
```
sysctl net.ipv4.tcp_keepalive_time=3600
```

### 3.2 C语言socket设置

对于Socket而言，可以在程序中通过socket选项开启TCP KeepAlive功能，并配置参数。对应的Socket选项分别为`SO_KEEPALIVE`、`TCP_KEEPIDLE`、`TCP_KEEPCNT`、`TCP_KEEPINTVL`。  
```
int keepalive = 1;          // 开启TCP KeepAlive功能
int keepidle = 7200;        // tcp_keepalive_time
int keepcnt = 9;            // tcp_keepalive_probes
int keepintvl = 75;         // tcp_keepalive_intvl

setsockopt(socketfd, SOL_SOCKET, SO_KEEPALIVE, (void *)&keepalive, sizeof(keepalive));
setsockopt(socketfd, SOL_TCP, TCP_KEEPIDLE, (void *) &keepidle, sizeof (keepidle));
setsockopt(socketfd, SOL_TCP, TCP_KEEPCNT, (void *)&keepcnt, sizeof (keepcnt));
setsockopt(socketfd, SOL_TCP, TCP_KEEPINTVL, (void *)&keepintvl, sizeof (keepintvl));
```
 

## 4 抓包实践分析

### 4.1 实践环境设置

将利用C语言分别编写服务器侧的代码和客户端的代码，在服务器侧开启TCP KeepAlive功能，服务器的端口号设置为6699，客户端的端口号为内核自动分配，并且每次客户端重启后内核分配的端口号可能不同，因此只需记住服务器端口号即可，另外一个相对的就是客户端。

服务器侧TCP KeepAlive参数具体设置如下：
```
tcp_keepalive_time = 55s
tcp_keepalive_probes = 2
tcp_keepalive_intvl = 6s
```

客户端运行于个人MAC电脑上，服务器运行在另外一台Linux系统上，后续通过wireshark软件进行抓包也是在MAC电脑上进行抓包。

### 4.2 wireshark抓包分析

将服务器开启，并通过客户端与之建立连接，过程中客户端和服务器无任何数据交互。通过wireshark抓包分析客户端和服务器间的数据交互，如下图：  
![tcp_keepalive_1]({{ "/assets/img/sample/tcp_keepalive_1.png"| relative_url }}){:height="50px" width="2000px"}  

可以看到，客户端和服务器建立连接后，服务器每隔55s发送了一次TCP KEEPALIVE的探测报文，也验证了上面服务器`tcp_keepalive_time`的设置，客户端收到探活报文后，会作出回应。点击具体报文可以查看报文详情，如下图所示：  
![tcp_keepalive_2]({{ "/assets/img/sample/tcp_keepalive_2.png"| relative_url }}){:height="320px" width="600px"}  

![tcp_keepalive_3]({{ "/assets/img/sample/tcp_keepalive_3.png"| relative_url }}){:height="320px" width="600px"}  

本文2.2节中介绍过TCP KEEPALIVE探活报文的序列号为上次发生数据交互时TCP报文序列号减1，实践中客户端和服务器在建立连接三次握手之后，没有发生任何数据交换，因此服务器收到客户端发送的最后报文的ACK值为1（注意wireshark抓包的报文序列号为相对序列号，Relative Sequence Number），所以服务器发送保活探测报文的序列号应该为0，客户端收到服务器探活报文后回应的确认报文序列号为1，和wireshark实际抓包相符合。


### 4.3 进一步测试

上面的测试中验证了`tcp_keepalive_time`参数，接下来验证TCP KeepAlive机制中另外两个参数。为了模拟由于连接中间路由等异常导致探活报文收不到回应的场景，借助Linux iptables命令设置防火墙，阻断TCP探活报文的传输。

这里将来自客户端的报文丢弃，即模拟服务器发送TCP KEEPALIVE探活报文后迟迟收不到回应报文的场景。使用的命令为：`sudo iptables -A INPUT -p tcp --sport $YOUR_CLIENT_PORT -j DROP`，其中`sport`为系统为客户端分配的端口号。

注意服务器是运行在Linux系统上的，客户端和wireshark运行在本地MAC电脑上，因此利用上述`iptables`设置防火墙规则后，wireshark仍能抓到所有确认报文，只是在Linux系统这一侧通过软件防火墙将报文丢弃了。

设置报文过滤规则后，再次通过wireshark抓包，如下图：  
![tcp_keepalive_4]({{ "/assets/img/sample/tcp_keepalive_4.png"| relative_url }}){:height="50px" width="800px"} 

从捕捉到的报文可以看到，3024.35s时间戳开始，服务器发送TCP KEEPALIVE探活报文后，一直收不到确认报文的返回，便隔6秒重新发送一个探活报文，即上文提到的服务器`tcp_keepalive_intvl`参数的设置。再等待2个`tcp_keepalive_intvl`时间间隔后，服务器仍未收到确认报文后，服务器发送了RST报文，以释放网络连接资源，这里的2次即`tcp_keepalive_probes`设置的。

注意：在完成上述测试后，借助`sudo iptables -F`命令清除所有的防火墙规则。


## 4 总结

本文借助wireshark软件，对TCP KEEPALIVE报文进行抓包分析，分析了TCP保活机制中`tcp_keepalive_time`、`tcp_keepalive_probes`、`tcp_keepalive_intvl`三个参数的实际效果，加深了对TCP KEEPALIVE机制的理解。



## 参考资料

[1] About TCP keepalive: [(http://baotiao.github.io/tech/2015/09/25/tcp-keepalive/)](http://baotiao.github.io/tech/2015/09/25/tcp-keepalive/)  
[2] https://www.codenong.com/cs105424711/