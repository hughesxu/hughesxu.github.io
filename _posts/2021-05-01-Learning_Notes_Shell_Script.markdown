---
title:  "学习笔记-Shell Script"
date:   2021-05-01 09:00:00 +0800
categories: [Language, Shell_Script]
tags: [Learning-Note]
---

## shell脚本初窥
例子：test
```
#!/bin/sh
cd ~
mkdir shell_txt
cd shell_txt
for ((i=0; i<10; i++)); 
do
    touch test_$i.txt
done
```
示例解释：
+ 第1行：指定脚本解释器（声明使用的shell名称），这里是用/bin/sh做解释器的。“#!”是一个约定的标记  
+ 第2行：切换到当前用户的home目录
+ 第3行：创建一个目录shell_txt  
+ 第4行：切换到shell_txt目录  
+ 第5行：循环条件，一共循环10次  
+ 第7行：创建一个test_1…10.txt文件  
+ 第8行：循环体结束  


	
## 1 shell基础知识
### 1.1 shell和shell脚本
- shell：指一种应用程序，提供用户操作系统的接口，通过shell将输入的命令和内核通信。需要调用其他应用程序。  
- shell脚本：shell script，是一种为shell编写的脚本程序。

### 1.2 脚本解释器
可以在/etc/shells文件下查看可使用的shell，常用的有如下几种：
- `sh`：即Bourne shell，POSIX（Portable Operating System Interface）标准的shell解释器，它的二进制文件路径通常是/bin/sh，由Bell Labs开发。
- `bash`：bash是Bourne shell的替代品，属GNU Project，二进制文件路径通常是/bin/bash。业界通常混用bash、sh、和shell
- `ksh`：ksh是Korn shell的缩写，由Eric Gisin编写，共有42条内部命令。兼容bash
- `csh`：csh 是Linux比较大的内核，它由以William Joy为代表的共计47位作者编成，共有52个内部命令。该shell其实是指向/bin/tcsh这样的一个shell，也就是说，csh其实就是tcsh。
- `tcsh`：整合C shell，提供更多的功能
- `zsh`：基于ksh发展出来，功能更强大的shell

### 1.3 shell脚本执行方式
- 作为可执行程序
```
chmod +x test.sh
./test.sh
```
**注意**：一定要写成 `./test.sh`，而不是`test.sh`，运行其它二进制的程序也一样，直接写test.sh，linux系统会去PATH里寻找有没有叫test.sh的，而只有/bin, /sbin, /usr/bin，/usr/sbin等在PATH里，当前目录通常不在PATH里，所以要用 `./test.sh` 告诉系统说，就在当前目录找。

- 作为解释器参数
```
/bin/sh test.sh
/bin/php test.php
```
**注意**：这种方式运行的脚本，不需要在第一行指定解释器信息，写了也没用。
 

## 2 shell基本语法
### 2.1 变量的设置规则
（1）变量与变量内容以一个等号“=”来连结  
- 命令：`myname=VBird`

（2）等号两边不能直接接空格符，如下所示为错误：   
- 命令：`myname = VBird`或`myname=VBird  Tsai`

（3）变量名称只能由英文字母、下划线、数字，但是开头字符不能是数字，如下为错误：   
- 命令：`2myname=VBird`

（4）变量内容若有空格符可使用双引号` ” `或单引号` ’ `将变量内容结合起来，同字符串一节。但是：  
- 双引号内的特殊字符如 ` $ ` 等，可以保有原本的特性，如下所示：  
命令：`var="lang is $LANG"`，则`echo $var`，可得`lang is en_US`

**补充**：	双引号里可以有变量；
双引号里可以出现转义字符。
- 单引号内的特殊字符则仅为一般字符 (纯文本)，如下所示：  
命令：`var='lang is $LANG'`，则`echo $var`，可得`lang is $LANG`

**补充**：	单引号里的任何字符都会原样输出，字符串中的变量是无效的；
单引号字串中不能出现单引号，对单引号使用转义符后也不行。

（5）可用转义字符` \ ` 将特殊符号(如 `[Enter]` ，`$`， `\` ， 空格符，`！`等)变成一般字符；

（6）在一串命令中，还需要通过其他的命令提供的信息，可以使用反单引号“\`命令\`”或 `$(命令)`。特别注意，那个 \` 是键盘上方的数字键 1 左边那个按键，而不是单引号`！` 例如想要取得核心版本的配置：  
命令：`version=$(uname -r)`，再`echo $version`，可得`2.6.18-128.el5`。

（7）若该变量为增加变量内容时，则可用 `"$变量名称"` 或 `${变量}` 累加内容，如下三种方法均可：  
- 命令：`PATH=$PATH:/home/bin`
- 命令：`PATH="$PATH":/home/bin`
- 命令：`PATH=${PATH}:/home/bin`

（8）若该变量需要在其他子程序运行，则需要以 `export` 来使变量变成环境变量：
- 命令：`export PATH`

（9）通常大写字符为系统默认变量，自行配置变量可以使用小写字符，方便判断 (纯粹依照使用者兴趣与嗜好) ；

（10）取消变量的方法为使用 `unset` ，变量前面不需要加上`“$”`字符  
- 命令：`unset myname`  


**变量作用域**  
`local`一般用于局部变量声明，多在在函数内部使用。  
（1）shell脚本中定义的变量是`global`的，其作用域从被定义的地方开始，到shell结束或被显示删除的地方为止。  
（2）shell函数定义的变量默认是`global`的，其作用域从**函数被调用时执行变量定义的地方**开始，到shell结束或被显示删除处为止。函数定义的变量可以被显示定义成`local`的，其作用域局限于函数内。但请注意，函数的参数是`local`的。  
（3）如果同名，Shell函数定义的`local`变量会屏蔽脚本定义的`global`变量。


### 2.2 环境变量
#### 2.2.1 查看环境变量
命令：`env`  
- `HOME`
代表用户的家目录。利用 cd ~ 就可以直接回到用户家目录了。
- `SHELL`
告知我们，目前这个环境使用的 SHELL 是哪支程序？ Linux 默认使用 /bin/bash 。
- `HISTSIZE`
这个与“历史命令”有关，亦即是曾经下达过的命令可以被系统记录下来，而记录的“笔数”则是由这个值来配置的。
- `MAIL`
当我们使用 mail 这个命令在收信时，系统会去读取的邮件信箱文件 (mailbox)。
- `PATH`
执行文件查找的路径，目录与目录中间以冒号(:)分隔， 由于文件的搜寻是依序由 PATH 的变量内的目录来查询，所以目录的顺序也是重要的。
- `LANG`
语系数据。
- `RANDOM`
目前大多数的 distributions 都会有随机数生成器，那就是 `/dev/random` 这个文件。 我们可以透过这个随机数文件相关的变量 ($RANDOM) 来随机取得随机数值喔。在 BASH 的环境下，这个 RANDOM 变量的内容，介于 0~32767 之间，所以，你只要 `echo $RANDOM` 时，系统就会主动的随机取出一个介于 0~32767 的数值。万一我想要使用 0~9 之间的数值呢？  
命令：`declare -i number=$RANDOM*10/32768；echo $number`

#### 2.2.2 常看所有变量（含环境变量和自定义变量）
命令：`set`

#### 2.2.3 自定义变量转成环境变量
命令：`export variable`

### 2.3 变量的显示
- 使用`echo`，变量前面需要加上`$`字符。输出会自动换行。加上 `-n` 可以不断行，继续在同一行显示。
以下两种均可：  
命令：`echo $PATH`  
命令：`echo ${PATH}`
`echo  >` ：清空原文件内容，填入新内容  
`echo  >>`：在原文件内容的基础上，填入新内容

- 使用`printf`，不会像`echo`那样会自动换行，需要显式添加换行符（`\n`）。

### 2.4 变量键盘读取、声明
#### 2.4.1 变量键盘读取
命令：`read [-pt] variable`
- 作用：读取来自键盘输入的变量
- 参数：  
`-p`：后面可以接提示符  
`-t`：后面可以接等待的“秒数”  
Exp:  
`read -p “Please keyin your name : ” -t 30 named`

#### 2.4.2 变量的声明
命令：`declare [-aixr] variable`
- 作用：声明变量的类型
- 参数： 
`-a`：数组类型（`array`）  
`-i`：整数数字类型（`interger`）  
`-x`：用法与`export`一样，将variable变成环境变量  
`-r`：将变量设置成readonly类型，该变量不能被更改内容，也不能被重设。  
Exp:  
`declare  -i  sum=100+300+50`


### 2.5 数组
bash支持一维数组（不支持多维数组），并且没有限定数组的大小。
#### 2.5.1 定义数组
在Shell中，用括号来表示数组，数组元素用“空格”符号分割开。  
一般形式：  
```
array_name=(value0 value1 value2 value3)
```  
或者  
```
array_name=(
value0
value1
value2
value3
)
```
单独定义数组各个分量：
```
array_name[0]=value0
array_name[1]=value1
array_name[2]=value2
```

#### 2.5.2 读取数组
一般格式： `${array_name[index]}`  
注意：使用 `@` 或 `*` 可以获取数组中的所有元素  
```
${array_name[*]}
${array_name[@]}
```

#### 2.5.3 获取数组的长度
例如：
```
# 取得数组元素的个数
length=${#array_name[@]}
# 或者
length=${#array_name[*]}
# 取得数组单个元素的长度
lengthn=${#array_name[n]}
```

### 2.6 字符串
字符串是shell编程中最常用最有用的数据类型（除了数字和字符串，也没啥其它类型好用了），字符串可以用单引号，也可以用双引号，也可以不用引号。
- **单引号**  
`str='this is a string'`  
单引号字符串的限制：  
（1）单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；  
（2）单引号字串中不能出现单引号（对单引号使用转义符后也不行）。  

- **双引号**  
```
your_name='qinjx'
str="Hello, I know your are \"$your_name\"!"
echo $str
```
结果：`Hello, I know your are “qinjx”! `  
双引号的优点：  
（1）双引号里可以有变量  
（2）双引号里可以出现转义字符

- **拼接字符串**  
```
your_name="qinjx"
greeting="hello, "$your_name"!"
greeting_1="hello, ${your_name}!"
echo $greeting $greeting_1
```
结果：`hello,qinjx! hello,qinjx!`

- **提取子字符串**
```
string="alibaba is a great company"
echo ${string:1:4}
``` 
结果：`liba`

### 2.7 特殊变量

变量|含义
:----:|:-------
`$0`	|当前脚本的文件名
`$n`	|传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。
`$#`	|传递给脚本或函数的参数个数。
`$*`	|传递给脚本或函数的所有参数。
`$@`	|传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同。
`$?`	|上个命令的退出状态，或函数的返回值。若成功执行，回传一个0值；若执行错误，回传一个非0值。
`$$`	|当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。

- `$*` 和 `$@`的区别：  
`$*` 和 `$@` 都表示传递给函数或脚本的所有参数，不被双引号`" "`包含时，都以`"$1" "$2" … "$n"` 的形式输出所有参数。
但是当它们被双引号`" "`包含时，
    - `$*` 会将所有的参数作为一个整体，以`"$1 $2 … $n"`的形式输出所有参数；
    - `$@` 会将各个参数分开，以`"$1" "$2" … "$n"` 的形式输出所有参数。

### 2.8 注释
以`#`开头的行就是注释，会被解释器忽略。

### 2.9 运算符
bash 支持很多运算符，包括算数运算符、关系运算符、布尔运算符、字符串运算符和文件测试运算符。  
原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如`awk`和`expr`，其中`expr`最常用。`expr`是一款表达式计算工具，使用它能完成表达式的求值操作。

#### 2.9.1 算术运算符

运算符|说明|举例
:----:|:-------|:-----
`+`	|加法	|`expr $a + $b` 结果为 30。
`-`	|减法	|`expr $a - $b` 结果为 10。
`*`	|乘法	|`expr $a \* $b` 结果为  200。
`/`	|除法	|`expr $b / $a` 结果为 2。
`%`	|取余	|`expr $b % $a` 结果为 0。
`=`	|赋值	|a=$b 将把变量 b 的值赋给 a。
`==`	|相等。用于比较两个数字，相同则返回 true。|	[ $a == $b ] 返回 false。
`!=`	|不相等。用于比较两个数字，不相同则返回 true。|	[ $a != $b ] 返回 true。  

**注意**：  
（1）乘号 `*` 前边必须加反斜杠 `\` 实现乘法运算  
（2）完整的`expr`表达式要被 \` \` 包含，或者采用 `$(表达式)`的方式

例子：
```
a=10
b=20
val=`expr $a + $b`
echo "a + b : $val"

if [ $a == $b ]
then
echo "a is equal to b"
fi

if [ $a != $b ]
then
echo "a is not equal to b"
fi
```
补充：数值计算的几种方法
- `$[]`  
例如：`x=1 	echo $[x += 1]`
- `expr`
- `(())`  
例如：
```
echo $ (( 13 % 3 ))`
echo $((x+=1))
```

#### 2.9.2 关系运算符
关系运算符只支持数字，不支持字符串，除非字符串的值是数字。

运算符|说明|举例
:----:|:-------|:-----
`-eq`	|检测两个数是否相等，相等返回 true。	|[ $a -eq $b ] 返回 true。
`-ne`	|检测两个数是否相等，不相等返回 true。	|[ $a -ne $b ] 返回 true。
`-gt`	|检测左边的数是否大于右边的，如果是，则返回 true。	|[ $a -gt $b ] 返回 false。
`-lt`	|检测左边的数是否小于右边的，如果是，则返回 true。	|[ $a -lt $b ] 返回 true。
`-ge`	|检测左边的数是否大于等于右边的，如果是，则返回 true。	|[ $a -ge $b ] 返回 false。
`-le`	|检测左边的数是否小于等于右边的，如果是，则返回 true。	|[ $a -le $b ] 返回 true。


例子：
```
a=10
b=20
if [ $a -lt $b ]
then
echo "$a -lt $b: a is less than b"
else
echo "$a -lt $b: a is not less than b"
fi

if [ $a -ge $b ]
then
echo "$a -ge $b: a is greater or equal to b"
else
echo "$a -ge $b: a is not greater or equal to b"
fi
```

#### 2.9.3 布尔运算符

运算符|说明|举例
:----:|:-------|:-----
`!`	|非运算，表达式为 true 则返回 false，否则返回 true。	|[ ! false ] 返回 true。
`-o`	|或运算，有一个表达式为 true 则返回 true。	|[ $a -lt 20 -o $b -gt 100 ] 返回 true。
`-a`	|与运算，两个表达式都为 true 才返回 true。	|[ $a -lt 20 -a $b -gt 100 ] 返回 false。

例子：
```
a=10
b=20
if [ $a -lt 5 -o $b -gt 100 ]
then
echo "$a -lt 100 -o $b -gt 100 : returns true"
else
echo "$a -lt 100 -o $b -gt 100 : returns false"
fi
```

#### 2.9.4 字符串运算符

运算符|说明|举例
:----:|:-------|:-----
`=`	|检测两个字符串是否相等，相等返回 true。	|[ $a = $b ] 返回 false。
`!=`	|检测两个字符串是否相等，不相等返回 true。	|[ $a != $b ] 返回 true。
`-z`	|检测字符串长度是否为0，为0返回 true。	|[ -z $a ] 返回 false。
`-n`	|检测字符串长度是否为0，不为0返回 true。	|[ -z $a ] 返回 true。
`str`	|检测字符串是否为空，不为空返回 true。	|[ $a ] 返回 true。

例子：
```
a="abc"
b="efg"
if [ -z $a ]
then
echo "-z $a : string length is zero"
else
echo "-z $a : string length is not zero"
fi

if [ -n $a ]
then
echo "-n $a : string length is not zero"
else
echo "-n $a : string length is zero"
fi

if [ $a ]
then
echo "$a : string is not empty"
else
echo "$a : string is empty"
fi
```

#### 2.9.5 文件测试运算符
文件测试运算符用于检测 Unix 文件的各种属性。
此处假设：变量 file 表示文件`“/var/www/tutorialspoint/unix/test.sh”`，它的大小为100字节，具有 rwx 权限。


操作符|说明|举例
:------:|:-------|:-----
`-b file`	|检测文件是否是块设备文件，如果是，则返回 true。	|[ -b $file ] 返回 false。
`-c file`	|检测文件是否是字符设备文件，如果是，则返回 true。	|[ -b $file ] 返回 false。
`-d file`	|检测文件是否是目录，如果是，则返回 true。	|[ -d $file ] 返回 false。
`-f file`	|检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。	|[ -f $file ] 返回 true。
`-g file`	|检测文件是否设置了 SGID 位，如果是，则返回 true。	|[ -g $file ] 返回 false。
`-k file`	|检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。	|[ -k $file ] 返回 false。
`-p file`	|检测文件是否是具名管道，如果是，则返回 true。	|[ -p $file ] 返回 false。
`-u file`	|检测文件是否设置了 SUID 位，如果是，则返回 true。	|[ -u $file ] 返回 false。
`-r file`	|检测文件是否可读，如果是，则返回 true。	|[ -r $file ] 返回 true。
`-w file`	|检测文件是否可写，如果是，则返回 true。	|[ -w $file ] 返回 true。
`-x file`	|检测文件是否可执行，如果是，则返回 true。	|[ -x $file ] 返回 true。
`-s file`	|检测文件是否为空（文件大小是否大于0），不为空返回 true。	|[ -s $file ] 返回 true。
`-e file`	|检测文件（包括目录）是否存在，如果是，则返回 true。	|[ -e $file ] 返回 true。

### 2.10 判断式
#### 2.10.1 test命令

**注意**：此处操作符基本和“运算符”一节所提到的相同，只是之前采用判断符号进行判断，而此节采用test命令代替。

- **数值测试**
功能：关于两个证书之间的判定，例如`test $n1 -eq $n2` 

参数|说明|举例
:------:|:-------|:-----
`-eq`	|等于则为真	|test ${num1} -eq ${num1}
`-ne`	|不等于则为真	
`-gt`	|大于则为真	
`-ge`	|大于等于则为真	
`-lt`	|小于则为真	
`-le`	|小于等于则为真	

- **字符串测试**  

参数|说明
:------:|:-------
`test str1 = str2`	|判定str1是否等于str2，等于则为true
`test str1 != str2`	|不相等则为true
`test -z string`	|判定字符串长度是否为0，若为空则为true
`test -n string`	|判定字符串长度是否为0，若为空字符串，则为false

**注意**：-n 选项可省略

- **文件类型测试**  
功能：关于某个文件名的“文件类型”判断，如`test -e filename`表示存在否。

参数|说明
:------:|:-------
`test -b filename`	|检测文件是否存在且是块设备文件（block device），如果是，则返回 true。
`test -c filename`	|检测文件是否存在且是字符设备文件（character device），如果是，则返回 true。
`test -d filename`	|检测文件是否存在且是目录（directory），如果是，则返回 true。
`test -f filename`	|检测文件是否存在且是文件（file），如果是，则返回 true。
`test -p filename`	|检测文件是否存在且是FIFO（pipe），如果是，则返回 true。
`test -e filename`	|检测文件（包括目录）是否存在，如果是，则返回 true。（常用）
`test -S filename`	|检测文件是否存在且是Socket文件，如果是，则返回true
`test -L filename`	|检测文件是否存在且是一个连接文件，如果是，则返回true

- **文件权限测试**  
功能：关于文件权限检测，如`test -r filename`表示文件是否可读

参数|说明
:------:|:-------
`test -u file`	|检测文件是否存在且设置了 SUID 位，如果是，则返回 true。
`test -r file`	|检测文件是否存在且可读，如果是，则返回 true。
`test -w file`	|检测文件是否存在且可写，如果是，则返回 true。
`test -x file`	|检测文件是否存在且可执行，如果是，则返回 true。
`test -s file`	|检测文件是否存在且为“非空白文件”，若是则返回 true。
`test -k file`	|检测文件是否存在且具有“Sticky Bit”属性，如果是，则返回 true。
`test -u file`	|检测文件是否存在且具有“SUID”属性，如果是，则返回 true。

- **文件比较**  
功能：两个文件之间的比较，如 `test file1 -nt file2`

参数|说明
:------:|:-------
`test file1 -nt file2`	|（newer than）判断file1是否比file2新，如果是，返回true
`test file1 -ot file2`	|（older than）判断file1是否比file2旧，如果是，返回true
`test file1 -ef file2`	|判断file1与file2是否为同一文件，可用于在判断hard link的判定上。主要意义在于判定两个文件是否均指向同一个inode。如果是，返回true

- **多重条件判定**
功能：多重条件判断

运算符|说明
:------:|:-------
`!`	|非运算，表达式为 true 则返回 false，否则返回 true。
`-o`	|或运算，有一个表达式为 true 则返回 true。
`-a`	|与运算，两个表达式都为 true 才返回 true。


**注意**：test命令执行结果并不会显示任何信息，可以通过 $? 或&&及||来显示整个结果。  
例如：
```
test -e /home && echo “exist” || echo “Not exist”
test -e /home && echo “exist” && exit 0
```

#### 2.10.2 判断符号[ ]
例如：判断变量`$HOME`是否为空，命令：`[ -z $HOME ]`

**注意事项**：  
（1）中括号[]内的每个组件都需要有空格键来分割；  
`[□"$HOME"□==□"$MAIL"□]` ，此处`□`均为空格  
（2）中括号内的变量，最好都以双引号括号起来；  
（3）中括号内的常量，最好都以单或双引号括起来。

a. `[ ]` 两个符号左右都要有空格分隔  
b. 内部操作符与操作变量之间要有空格：如 `[“a” = “b” ]` 
c. 字符串比较中，`> <` 需要写成`> \<` 进行转义  
d. `[ ]` 中字符串或者`${}`变量尽量使用`””` 双引号扩住，以避免值未定义引用而出错  
e. `[ ]` 中可以使用 `–a –o` 进行逻辑运算  
f. `[ ]` 是bash 内置命令：`[` is a shell builtin

#### 2.10.3 判断符号 [[  ]] 双方括号

基本要素：  
- `[[ ]]` 两个符号左右都要有空格分隔
- 内部操作符与操作变量之间要有空格：如  `[[  “a” =  “b”  ]]`
- 字符串比较中，可以直接使用 `>` `<` 无需转义
- `[[ ]]` 中字符串或者`${}`变量尽量`””` 双引号扩住，如未使用 `""` 双引号扩住的话，会进行模式和元字符匹配
```
[root@localhostkuohao]# [[ "ab"=a* ]] && echo "ok"
  ok
```
- `[[]]` 内部可以使用 &&  || 进行逻辑运算
- `[[ ]]` 是bash  keyword：`[[` is a shell keyword  
**[[ ]] 其他用法都和[ ] 一样** 

- `[[ ]]` 和 `[ ]` 都可以和 `!` 配合使用  

```
优先级      !  >  && > || 
逻辑运算符  < 关系运算符

逻辑运算符  ： !  &&  || -a  -o
关系运算符  ： <  >  \> \<  ==  = !=  – eq –ne  -gt -ge  –lt  -le
```

- `[[  ]]` 比`[ ]` 具备的优势

    - `[[`是 bash 程序语言的关键字。并不是一个命令，`[[ ]]` 结构比`[ ]`结构更加通用。在`[[`和`]]`之间所有的字符都不会发生文件名扩展或者单词分割，但是会发生参数扩展和命令替换。
    - 支持字符串的模式匹配，使用`=~`操作符时甚至支持shell的正则表达式。字符串比较时可以把右边的作为一个模式，而不仅仅是一个字符串，比如`[[ hello == hell? ]]`，结果为真。`[[ ]]` 中匹配字符串或通配符，不需要引号。
    - 使用`[[ ... ]]`条件判断结构，而不是`[...]`，能够防止脚本中的许多逻辑错误。比如，`&&`、`||`、`<`和`>` 操作符能够正常存在于`[[ ]]`条件判断结构中，但是如果出现在`[ ]`结构中的话，会报错。
    - bash把双中括号中的表达式看作一个单独的元素，并返回一个退出状态码。


### 2.11 条件判断式
#### 2.11.1 利用if…then判断
- **单层、简单条件判断式**
语句形式：
```
if [ 条件判断式 ]; then
语句执行
fi					#将if反过来写，表示结束
```

**注意**：可以使用 `&&` 代替`AND`，使用 `||` 代替`or`
如：`[ “$yn” == “Y” -o “$yn” == “y” ]`可以变为`[ “$yn” == “Y” ] || [ “$yn” == “y” ]`

- **多层、复杂条件条件判断式**  
语句形式：  
```
if [ 条件判断式 ]; then
语句执行1
else
语句执行2
fi
或
if [ 条件判断式1 ]; then
语句执行1
elif [ 条件判断式2 ]; then
语句执行2
else
语句执行3
fi
```

**注意**：每一个`if`或`elif`后面均要接一个`then`来处理

#### 2.11.2 使用case…esac判断
语句形式：  
```
case $变量名称 in		     #关键词为case
“第一个变量内容”）			  #每个变量内容建议用双引号括起来，关键字为）
程序段
;;							#每个类型结尾使用两个连续的分号处理
“第二个变量内容”）
程序段
;;
*）							#最后一个变量内容都会有 * 来代表所有值
程序段
;;
esac 						#case反过来写，表示结尾
```

#### 2.11.3 利用 function 功能
语句形式：
```
function fname ( ) {				#fname为自定义的执行命令名称
程序段
}
```
- 函数返回值，可以显式增加`return`语句；如果不加会将最后一条命令运行结果作为返回值。
- 函数中用`return`返回的值，在函数执行后用 `$?` 获得。
- Shell 函数返回值只能是整数，一般用来表示函数执行成功与否，0表示成功，其他值表示失败。

**注意**：  
（1）调用函数只需要给出函数名，不需要加括号。  
（2）function也是拥有内置变量的。内置变量与shell script类似，函数名称代表`$0`，后续接的变量用`$1`, `$2`等代替。其值由引用函数时所跟随的参数相关。

### 2.12 循环 (loop)
#### 2.12.1 while do done, until do done （不定循环）
- `while do done`  
语句形式：
```
while [ condition ]				#中括号内为判断式
do								#done内为循环的开始
程序段落
done							#done内为循环的结束
```
功能：当condition条件成立时，就进行循环，直到condition的条件不成立才停止。

- `until do done`  
语句形式：
```
until [ condition ]				#中括号内为判断式
do								#done内为循环的开始
程序段落
done							#done内为循环的结束
```
功能：当condition条件成立时，就停止循环，否则继续执行循环的程序段。

#### 2.12.2 for ... do ... done（固定循环）
语句形式：  
```
for var in con1 con2 con3
do 
程序段
done
``` 
功能：第一次执行时，var为con1；第二次为con2......  
注意：  
（1）seq 语句可表示连续数字。如：
```
for sitenu in $( seq 1 100 )
do 
程序段
done 
```
（2）顺序输出字符串。如：
```
for str in ‘This is a string’
do
echo $str
done
```

#### 2.12.3 for ... do ... done的数值处理
语句形式：
```
for (( 初始值; 限制值; 执行步长))
do 
程序段
done 
```
 
## 3 其他特殊命令
#### 3.1 文本颜色
```
echo -e "\033[字背景颜色;文字颜色m字符串\033[0m"
```

eg :
`echo -e "\033[41;36m something here \033[0m"`  
其中41的位置代表底色，36的位置代表字的颜色

对于`printf`同样生效：`printf "\e[颜色值m 文本\n" #这里加上换行符`  
`'\e['` 和 `‘\033[’` 是一个控制码，紧跟在在这个控制码后面的字符是由特殊含义的。

颜色|前景色|背景色
:------:|:-------|:-----
黑色 (Black)		|30			|40
红色 (Red)			|31			|41
绿色 (Green)		|32			|42
黄色 (Yellow)		|33			|43
蓝色 (Blue)			|34			|44
紫红色 (Magenta)	|35			|45
青色 (Cyan)		    |36			|46
白色 (White)		|37			|47

ANSI 码|含义
:------:|:-------
0		|常规文本
1		|粗体文本
4		|含下划线文本
5		|闪烁文本
7		|反色(补色)文本

- **控制选项说明**  

选项|含义
:------:|:-------
`\33[0m` 	|关闭所有属性 
`\33[1m` 	|设置高亮度 
`\33[4m` 	|下划线 
`\33[5m` 	|闪烁 
`\33[7m` 	|反显 
`\33[8m` 	|消隐 
`\33[30m — \33[37m` |设置前景色 
`\33[40m — \33[47m` |设置背景色 
`\33[nA` 	|光标上移n行 
`\33[nB` 	|光标下移n行 
`\33[nC` 	|光标右移n行 
`\33[nD` 	|光标左移n行 
`\33[y;xH`	|设置光标位置 
`\33[2J` 	|清屏 
`\33[K` 	|清除从光标到行尾的内容 
`\33[s` 	|保存光标位置 
`\33[u` 	|恢复光标位置 
`\33[?25l` 	|隐藏光标 
`\33[?25h` 	|显示光标

set –e		// bash如果任何语句的执行结果不是true则应该退出


### 3.2 kill / killall  
```
killall 	(选项)	(参数)  
```
**选项**
- `-e`：对长名称进行精确匹配；
- `-l`：忽略大小写的不同；
- `-p`：杀死进程所属的进程组；
- `-i`：交互式杀死进程，杀死进程前需要进行确认；
- `-l`：打印所有已知信号列表；
- `-q`：如果没有进程被杀死。则不输出任何信息；
- `-r`：使用正规表达式匹配要杀死的进程名称；
- `-s`：用指定的进程号代替默认信号`SIGTERM`；
- `-u`：杀死指定用户的进程。

**参数**：进程名称（指定要杀死的进程名称）。

```
kill	(选项)	(参数)
```
**选项**
- `-a`：当处理当前进程时，不限制命令名和进程号的对应关系；
- `-l <信息编号>`：若不加<信息编号>选项，则-l参数会列出全部的信息名称；
- `-p`：指定kill 命令只打印相关进程的进程号，而不发送任何信号；
- `-s <信息名称或编号>`：指定要送出的信息；
- `-u`：指定用户。

**参数**:	进程或作业识别号（指定要删除的进程或作业）。


### 3.3 perl
Perl 用作命令行操作的快速而又难看的脚本是很有用的；通过命令行，Perl仅用一行就可以实现大多数其它语言需要数页代码才能完成的任务，这个小东东的功能可是非常强大的。

**参数**：
- `-w`     打开警告。
- `-i`     在原文件中编辑（就地编辑）。
- `-i.bak` 就地编辑，但是会备份原文件，并且以`.bak`为后缀，这个`.bak`可以修改成自己想要的任何符号。
- `-n`    使用`<>`将所有`@ARGV`参数当作文件来逐行运行,会将读入的内容隐式的逐一按行来遍历文件，每一行将缺省保存在 `$_`；意即会把输入的文件逐行的读取并保存在`$_`这个变量中，我们修改`$_`相当于间接影响文件中的内容，这个工作其实是perl封装好了的，直接使用就好了；这个参数不会自动打印`$_`。
- `-p`     这个和`-n`类似，但是会打印`$_`。
- `-e`     指定字符串用作脚本执行；通常后跟单引号，把需要执行的语句封装在其中。
```
替换A为B
      perl  -i  -pe ‘s/old_str/new_str/g’  files
替换A为B并备份
      perl  -i.bak -pe  ‘s/old_str/new_str/g’ files
修改并输出到屏幕
      perl  -ne ‘s/old_str/new_str/g;print;’  files    // 此处修改后输出到屏幕，但并不会改变原文件。
搜索满足条件的行
      perl  -i  -ne ‘print  if /condition/’  files
在文件中插入行号
      perl  -i  -pe ‘$_ = sprintf “d %s”, $. , $_’  files
在匹配的某行行首添加字串
      perl  -i  -pe ‘print  “string” if  /condition/’  files
在匹配的某行行尾添加字串
      perl  -i  -pe ‘chomp; $_ = $_ . “string\n”  if /condition/’  files
在匹配的某行前增加一行
      perl  -i  -pe ‘print  “string\n” if  /condition/’  files
在匹配的某行后增加一行
      perl  -i  -pe ‘$_ = $_ . “string\n”  if /condition/’  files
```


### 3.4 正则表达式
```
regex="([0-9]+) [kKMG]?B\)$"
if [[ $tmp =~ $regex ]]; then
```
加了双中括号`[[ ]]`以后，在`[[`和`]]`之间所有的字符都不会发生文件名扩展或者单词分割，但是会发生参数扩展和命令替换，bash把双中括号中的表达式看作一个单独的元素，并返回一个退出状态码。  
在`[[`和`]]`之间支持字符串的模式匹配，使用`=~`操作符时甚至支持shell的正则表达式。  
简单来说，=~ 就是匹配正则表达式用的。

### 3.5 换行
**\ 再加ENTER**


### 3.6 log输入与屏幕输出
```
https://unix.stackexchange.com/questions/145651/using-exec-and-tee-to-redirect-logs-to-stdout-and-a-log-file-in-the-same-time
Use process substitution with & redirection and exec:
exec &> >(tee -a "$log_file")
echo This will be logged to the file and to the screen
$log_file will contain the output of the script and any subprocesses, and the output will also be printed to the screen.
>(...) starts the process ... and returns a file representing its standard input. exec &> ...redirects both standard output and standard error into ... for the remainder of the script (use just exec > ... for stdout only). tee -a appends its standard input to the file, and also prints it to the screen.
```

