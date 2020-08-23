---
title:  "Redis源码笔记-服务器日志和函数可变参数处理server.c"
date:   2020-08-21 09:00:00 +0800
categories: [Redis, server]
tags: [Redis]
---

## server.h / server.c

Redis源码中定义了几个和日志相关的函数，用于将不同级别的信息打印到不同的位置（日志文件或标准输出，取决于配置文件的设置），这些函数包括：  
 
```
void serverLog(int level, const char *fmt, ...);
void serverLogRaw(int level, const char *msg);
void serverLogFromHandler(int level, const char *msg);
```

其中，`serverLogRaw()`是`serverLog()`的底层实现，差别在于`serverLog()`进行了上层的简单封装，以支持可视化字符串的打印，而`serverLogRaw()`则只接收完整的字符串进行打印。

Redis Logging定义了四种日志等级，从低到高分别为：调试、详细、注意、警告。对应的宏定义如下：  
```
/* Log levels */
#define LL_DEBUG 0
#define LL_VERBOSE 1
#define LL_NOTICE 2
#define LL_WARNING 3
#define LL_RAW (1<<10) /* Modifier to log without timestamp */
```

### serverLog()

`serverLog()`函数提供了一个`printf-alike`的日志打印支持，能够支持格式化字符串，并接收可变参数。  
```
void serverLog(int level, const char *fmt, ...) {
    va_list ap;
    char msg[LOG_MAX_LEN];

    if ((level&0xff) < server.verbosity) return;

    va_start(ap, fmt);
    vsnprintf(msg, sizeof(msg), fmt, ap);
    va_end(ap);

    serverLogRaw(level,msg);
}
```

函数中对日志打印级别进行了控制，只有给定日志级别不小于服务器设置的级别时，日志才会打印出来，否则，函数提前返回。  
利用了`va_list`、`va_start`、`va_end`等函数对可变参数进行了支持，其原理是利用函数参数在栈中的空间排布，栈空间排布和`va_list`的操作如下图所示：  
![Variadic Macros]({{ "/assets/img/sample/variadic_macros.jpg"| relative_url }})

`int vsnprintf (char * s, size_t n, const char * format, va_list arg)`, 函数利用可变参数列表来格式化字符串，把字符串保存在`s`指向的空间中。  
`Write formatted data from variable argument list to sized buffer`  

### serverLogRaw()

`serverLogRaw()`函数是日志打印的底层实现，主要控制了日志打印的如下几个方面：  
* 日志级别。如果给定级别低于服务器设置的`server.verbosity`级别，日志不输出，函数直接返回。  
* 日志打印位置。如果服务器配置了`server.logfile`，日志会打印到相应的日志文件中。否则，直接输出至标准输出。  
* 日志打印格式。如果在日志级别中设置了原始位，则只打印原始字符串信息。否则，会在字符串首加入进程号、日期等信息。  

```
void serverLogRaw(int level, const char *msg) {
    // Defined in syslog.h
    const int syslogLevelMap[] = { LOG_DEBUG, LOG_INFO, LOG_NOTICE, LOG_WARNING };
    const char *c = ".-*#";
    FILE *fp;
    char buf[64];
    int rawmode = (level & LL_RAW);
    int log_to_stdout = server.logfile[0] == '\0';

    level &= 0xff; /* clear flags */
    if (level < server.verbosity) return;

    fp = log_to_stdout ? stdout : fopen(server.logfile,"a");
    if (!fp) return;

    // log without timestamp
    if (rawmode) {
        fprintf(fp,"%s",msg);
    } else {
        int off;
        struct timeval tv;
        int role_char;
        pid_t pid = getpid();

        gettimeofday(&tv,NULL);
        struct tm tm;
        /*
         * 自定义的localtime函数
         * 标准的 localtime 在多线程下可能出现死锁的情况
         */
        nolocks_localtime(&tm,tv.tv_sec,server.timezone,server.daylight_active);
        off = strftime(buf,sizeof(buf),"%d %b %Y %H:%M:%S.",&tm);
        snprintf(buf+off,sizeof(buf)-off,"%03d",(int)tv.tv_usec/1000);
        if (server.sentinel_mode) {
            role_char = 'X'; /* Sentinel. */
        } else if (pid != server.pid) {
            role_char = 'C'; /* RDB / AOF writing child. */
        } else {
            role_char = (server.masterhost ? 'S':'M'); /* Slave or Master. */
        }
        /*
         * 依次存放：
         * pid, X/C/S/M, time, .-*#, msg
         */
        fprintf(fp,"%d:%c %s %c %s\n",
                (int)getpid(),role_char, buf,c[level],msg);
    }
    fflush(fp);

    if (!log_to_stdout) fclose(fp);
    if (server.syslog_enabled) syslog(syslogLevelMap[level], "%s", msg);
}
```

对于包含时间信息的日志打印会被加入如下信息在字符串首部：  
* 服务器进程号  
* X/C/S/M：体现当前进程的状态，主进程 / 从进程 / Sentinel / RDB/AOF子进程  
* 格式化后的日期  
* 表示不同日志等级的字符 
* 信息字符串

日志打印实例：  
```
1481:C 22 Aug 14:46:31.494 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1481:C 22 Aug 14:46:31.495 # Redis version=4.0.9, bits=64, commit=00000000, modified=0, pid=1481, just started
1481:C 22 Aug 14:46:31.495 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
1481:M 22 Aug 14:46:31.497 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
1481:M 22 Aug 14:46:31.497 # Server can't set maximum open files to 10032 because of OS error: Invalid argument.
1481:M 22 Aug 14:46:31.497 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
                   _._
              _ .-``__ ''-._
         _.- ``    `.  `_.  ''-._           Redis 4.0.9 (00000000/0) 64 bit
     .-`` .-```.  ```\/    _.,_ ''-._
    (    '      ,       .-`  | `,    )     Running in standalone mode
    |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
    |    `-._   `._    /     _.-'    |     PID: 1481
    `-._    `-._  `-./  _.-'    _.-'
    |`-._`-._    `-.__.-'    _.-'_.-'|
    |    `-._`-._        _.-'_.-'    |           http://redis.io
    `-._    `-._`-.__.-'_.-'    _.-'
    |`-._`-._    `-.__.-'    _.-'_.-'|
    |    `-._`-._        _.-'_.-'    |
    `-._    `-._`-.__.-'_.-'    _.-'
        `-._    `-.__.-'    _.-'
            `-._        _.-'
                `-.__.-'

1481:M 22 Aug 14:46:31.501 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1481:M 22 Aug 14:46:31.501 # Server initialized
1481:M 22 Aug 14:46:31.501 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
1481:M 22 Aug 14:46:31.503 * DB loaded from disk: 0.001 seconds
1481:M 22 Aug 14:46:31.503 * Ready to accept connections
```

### nolocks_localtime()

值得一提的是，在对时间进行转换时，redis源码考虑到标准的`localtime`在多线程下可能出现的死锁问题，所以自定义了一个不带锁的函数`nolocks_localtime()`，用于完成时间格式的转换。  
关于`localtime()`函数多线程下可能带来的死锁问题，可以参考：[localtime函数的死锁风险](https://www.codercto.com/a/59045.html)。  
