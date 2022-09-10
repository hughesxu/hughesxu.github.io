---
title:  "学习笔记-Expect"
date:   2021-05-01 09:00:00 +0800
categories: [Language, Expect]
tags: [Learning-Note]
---



## 1 Expect 脚本语法
### 1.1 Expect 脚本初窥
一个简单的输入密码操作
```
#！/usr/bin/expect –f 
set timeout 100
set password "123456"
spawn sudo rm -rf zzlogic
expect "root123456"
send "$password\n"
interact
```

### 1.2 Expect 命令
#### 1.2.1 spawn
Func: 启动新的进程 
`spawn`命令是Expect的初始命令，它用于启动一个进程，之后所有expect操作都在这个进程中进行，如果没有`spawn`语句，整个expect就无法再进行下去了。  
`spawn  ssh  root@192.168.0.1`

#### 1.2.2 send
Func : 用于向进程发送字符串；  
eg: `［send "ispass\r"］`  
**注意**：命令结尾加 `\t`或者 `\n`，`\t`表示回车，`\n`表示换行。

`send`命令有几个可用的参数：   
- `-i  $spawn_id`：这个参数用来向不同spawn_id的进程发送命令，是进行多程序控制的关键参数。 
- `-s`: s代表`slowly`，也就是控制发送的速度，这个参数使用要与expect中的变量`send_slow`相关联。


#### 1.2.3 expect
Func: 从进程接收字符串，用于等候一个相匹配的输出，一旦匹配就执行后面的动作。  
eg: `expect "password:"`  
使用方法：   
`expect 表达式 动作 表达式 动作 ………………`


eg :   
`expect "hi\n"send "hello there!\n"`  
从标准输入中等到hi和换行键后，向标准输出输出hello there。  
- `$expect_out(buffer)`存储了所有对expect的输入，$expect_out(0,string)存储了匹配到expect参数的输入。
- `expect  *;`  # Receive *anything* returned immediately.

Elements|Significance
:----:|:-------
`expect_out(0,start)`     	|Index of the first character of the string that matched the entire expression
`expect_out(0,end)`    	 | Index of the last character of the string that matched the entire expression
`expect_out(0,string)`    	|String that matched the entire expression
`expect_out(1..9,start)`   	 | Index of the first character of the string that matched the pattern enclosed in the 1st - 9th set of parentheses
`expect_out(1..9,end)`    	|Index of the last character of the string that matched the pattern enclosed in the 1st - 9th set of parentheses
`expect_out(1..9,string)`    |String that matched the pattern enclosed in the 1st - 9th set of parentheses
`expect_out(buffer)`    	|Entire contents of the buffer when a match was found
`expect_out(spawn_id)`    |Spawn id of the process which produced the matching pattern




1. 直接使用：
```
spawn ftp connect
    set spid_ftp $spawn_id
    spawn telnet connect
    set spid_telnet $spawn_id
    exp_send –i $spid_ftp “ftp command”
exp_send –i $spid_telnet “telnet command”
```

2. expect过程调用
```
expect {
-i $spid
-timeout "your timeout"
-re ".+" {append result $expect_out(buffer)}
-re "other exit condition" {}
}
```
- `-i`选项   
`-i`选项一般用于同时控制多个`spawn`进程，通过这个选项向不同spawn_id发送命令就可以同时控制多个进程了

- `-d`选项
几乎所有expect命令都支持`-d`选项，d代表`default`，也就是设置某些命令的默认值，比如：  
```
remove_nulls –d 1  
timeout –d 30
```

- `-re`选项  
`re`代表`regular expression`，也就是正则表达式，用于expect、interact命令族中，这种类型的表达式具有匹配一切字符的能力。


#### 1.2.4 interact
Func: 允许用户交互。`interact`命令是Expect中非常重要的命令之一，它用来在脚本中将控制权交给用户，由用户与`spawn`生成的进程进行交互。
退出`expect`返回终端，可以继续输入，否则将一直在`expect`不能退出到终端。

#### 1.2.5 exit 
`exit`命令功能很简单，就是直接退出脚本，但可以利用这个命令对脚本做一些扫尾工作

#### 1.2.6 set timeout 
设置超时时间，单位是秒 (s)；expect超时等待的时间。默认`timeout`为10s。
```
set timeout  -1  ##设置为永不超时
```

#### 1.2.7 log_user 
```
log_user 1    #是否显示脚本输出信息 1代表显示，0代表不显示
```
主要针对send命令，使得命令执行结果不打印，但是不影响puts输出


#### 1.2.8 match_max
```
match_max 100000  ##设置缓冲区大小，单位：字节。
```

#### 1.2.9 log_file
```
log_file expectout.txt
```

#### 1.2.10 sleep and after

There are three different entities:
The Tclx's sleep
The sleep command from the Tclx package. According to the documentation, it takes a decimal argument, taken to be the number of seconds to sleep. However, the fraction part is truncated. That means sleep 2.5 will sleep for two seconds.
The Expect's sleep
The sleep command from the Expect package. This is similar to its counterpart from the Tclxpackage. However, sleep 2.5 means sleeping for 2.5 seconds, there is no truncation.
After
Finally, the built-in after, which is a totally different beast. The after command takes its first input as the number of milliseconds to sleep off. This is the "synchronous" mode Jerry refers to. After also takes a second argument. In this case, after returns a token right away. After the specified time, the script will be executed. With the token, you can cancel the script.
I hope this helps.
My try at a short explanation:
The Tcl sleep will like the TclX sleep just pause the script.
The after command can pause the script, but it is normally used for event based programming. It can execute a script after the elapsed time (if the event loop is running).
More on this see here at beedub.com.


#### 1.2.11 wait
首先`spawn`会开启一个子进程（spawn_id）去执行命令，`expect eof`用于等待进程结束，另外有一个close命令用于直接结束子进程（根据spawn_id来定位子进程）。问题在于`eof` 和 `close`都只是杀死子进程，但子进程变为僵尸进程依然存在进程列表中，僵尸进程会占用句柄，但是句柄是有限的，大量僵死进程的产生，导致整个脚本进程无法继续运行，报错退出。  
网上的例子大多是流程式的脚本，不是本文这种循环运行的，所以等待`eof`后，主进程退出，将僵尸进程也回收了，因此不会有任何问题。
不退出主进程，还要及时回收僵尸子进程，很多语言都内置了相关的方法，expect脚本也不例外，`wait`就是负责给子进程收尸的。所以文章开头的脚本应该加上`wait`来及时回收`telnet`僵尸进程。

另外`close`, `wait`可以隐含地产生。`expect`和`interact`都能检测到当前程序的退出,并隐含地执行一个`close`，但是`wait`只会在父进程退出时（即脚本退出时）隐含产生。所以对于只有一个子进程的脚本在exit后可以不写wait. 

#### 1.2.12 close a subprocess

(1)	`expect eof`和`interact`都能检测到当前程序的退出，并隐含地执行一个close;   
```
close does not call wait since there is no guarantee that closing a process connection will cause it to exit.
Fortunately, in many scripts it is not necessary to explicitly close the connection because it can occur implicitly. There are two situations when you do not have to use close: 
• when the Expect process ends, or 
• when the expect command reads an eof from the spawned process.
```
`Close`之后`spawned process`会die，结束子进程。  
After closing the connection, a spawned process can finish up and exit.

(2)	`Wait`用来清理子进程， 
```
The -nowait flag causes the wait to return immediately with the indication of a successful wait. When the process exits (later), it will automatically disappear without the need for an explicit wait.
Because a process will not disappear from the system until you give the wait command, it is common to speak of waiting for or waiting on a process. Some people also like to use the term reap as in "reaping a process".

Unlike close, however, wait implicitly happens in only one case-when an Expect process (i.e., script) exits. On exit, all the spawned processes are waited for.
This means that Expect scripts that only spawn a single process and then exit, need not call wai t since it will be done automatically
```
只有一个子进程的程序不需要显式调用`close/wait`，Expect主进程结束时会隐式调用`close`和`wait`.
```
wai t returns a list describing a process that was waited upon. 
The list contains the process id, spawn id, and a 0 or -1. A 0 indicates that the process was waited upon successfully. In this case, the next value is the status.
```

(3)	Kill  
Once killed, the process connection should be recycled by calling close and wait.  
(4)	


```
#!/usr/bin/expect -f

set spawn_id 0

set expectStr ""

spawn /bin/bash
expect "# $"
match_max 100000

# Now, start switchd with the new skip flags.
log_user 0
set timeout 120
send -i $spawn_id ""
expect {
    -i $spawn_id $expectStr { after 10 }
    timeout { return -code error "FATAL ERROR" }
}

expect -timeout 1 -i $spawn_id $prompt

send -i $spawn_id "\r"
expect -timeout 1 -i $spawn_id $prompt

log_user 1
set pid [exp_pid -i $spawn_id]  # exp_pid命令用来获取当前spawn的进程号，支持-i选项，用来指定具体的spawn进程
exec kill -9 $pid
puts "\n"
```

### 1.3 命令行参数
The script name is not included in the argument list. 
The arguments are only those of the script, not of Expect. This is convenient in many scripts because the argument list can be directly used without stripping out the command name.   
The script name is stored in the variable `argv0`. Adding the command "puts "argv0 : $argv0 "" as the first line of the example script causes it to print an additionalline:
``` 
argvO: eeho.exp 
arg 0: faa 
arg 1: bar 
arg 2: 17 and a half
```
Very old systems do not follow the `#!` convention at all, and instead use `/bin/` sh to execute all scripts. Inserting the following lines at the beginning of your script allows it to be portable between such systems and modern ones that do invoke the correct `interpreter.t` 
```
#!/bin/sh 
set kludge { ${1+"$@"} 
shift 
shift 
exec expect -f $0 ${1+"$@"} 
} # rest of script follows
```

### 1.4 parameters
Flags that Expect knows about are: 

Flags|Description
:----:|:-------
`-b` 		|read the script a line at a time (i.e., unbuffered) 
`-c  cmd`  	|execute this command before any in the script 
`-f  file`	|read commands from this file 
`-d`		|print internal (diagnostic) information , `exp_internal 1`
`-D` 		|enable the debugger 
`-i` 		|run interactively
`-n`		|do not source - / . expect. Rc
`-N`		|do not source $expect_library / expect. Rc
`-`		|read commands from the standard input
`--`		|do not interpret remaining arguments, The `--` is a flag to Expect. It says not to interpret any of the script arguments but just to pass them on to the script.

The `-f` flag prefaces a file from which to read commands from.   
The flag itself is optional as it is only useful when using the `#!` notation (see above), so that other arguments may be supplied on the command line.  
在脚本第一行解释器处加上这个参数，可以使得所有的`-c`选项能够被识别为参数。  
Just as with `--`, when a script starts out with this -f line and is invoked just by its name (without "expect"), it behaves as you had entered the following command: 
```
% expect -f script args   
```
Now you can use Expect flags such as `-c` and they will be correctly handled. Since the `-f script` looks like a flag, Expect continues looking and finds the `-c` and interprets this as a flag, too.

If you give the `-c` before the script name, it will not be included in the argv variable when the script ultimately gets control.

### 1.5 debug


### 1.6 send slow
```
LIB/common_lib.tcl
proc console_login { ts_IP_sup1 ts_port_sup1 {sup_pw 0}} {

set send_slow {2 .1}
send -s -i $sessionID "$bootString\r"
```