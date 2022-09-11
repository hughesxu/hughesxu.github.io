---
title:  "学习笔记-Makefile"
date:   2021-05-01 09:00:00 +0800
categories: [Language, Makefile]
tags: [Learning-Note]
---


```
Every Makefile should contain this line: SHELL = /bin/sh
to avoid trouble on systems where the SHELL variable might be inherited from the environment.
(This is never a problem with GNU make.)
```

## 二次展开
GUN make的执行过程分为两个阶段：  
第一阶段：读取所有的makefile文件（包括`MAKEFILES`变量指定的、指示符`include`指定的、以及命令行选项`-f（--file）`指定的makefile文件），内建所有的变量、明确规则和隐含规则，并建立所有目标和依赖之间关系结构链表。  
第二阶段：根据第一阶段已经建立的依赖关系结构链表决定哪些目标需要更新，并使用对应的规则來重建这些目标。

Tips :  
(1)	尽在Makefile文件的目标项冒号后的另起一行的代码才是shell代码。  
(2)	Makefile中的shell，每一行是一个进程，不同行之间变量值不能传递。所以，Makefile中的shell不管多长也要写在一行。  
(3)	Makefile中的变量以`$`开头， 所以，为了避免和shell的变量冲突，shell的变量以`$$`开头  
- shell变量引用：`“${xx}”`或者`“$xx”`
- makefile变量引用：`“$(xx)”`或者`“${xx}”`
- Makefile 用到环境变量时，不能直接使用`$ORACLE_HOME`，而是要使用`$(ORACLE_HOME)`
```
In the VPATH variable, directory names are separated by colons or 
blanks. The order in which directories are listed is the order 
followed by make in its search.
```
```
TOP_DIR=$(shell TOP_DIR=Unable_To_Find_Top_Dir;CUR_DIR=$$(pwd);
while [ "$$CUR_DIR" != "/" ];
do { 
if [ -a $$CUR_DIR/.git ]; then 
TOP_DIR=$$CUR_DIR;
fi;
CUR_DIR=$$(dirname $$CUR_DIR); 			// dirname是Linux command
} 
done;
echo $$TOP_DIR)
```
- `CURDIR`: CURDIR是make的内嵌变量，自动设置为当前目录

**Special Notes**:
Makefile中调用shell 命令，在shell命令中调用变量时，需要使用 `$$()`

## Default goal
```
The order of rules is not significant, except for determining the 
default goal: the target for make to consider, if you do not otherwise 
specify one. The default goal is the target of the first rule in the 
first makefile.
If the first rule has multiple targets, only the first target is taken 
as the default. 
There are two exceptions: a target starting with a period is not a 
default unless it contains one or more slashes, ‘/’, as well; and, a 
target that defines a pattern rule has no effect on the default goal.
```

## Variable definitions
```
Variable definitions are parsed as follows:
immediate = deferred
immediate ?= deferred
immediate := immediate
immediate ::= immediate
immediate += deferred or immediate
immediate != immediate

define immediate
deferred
endef
define immediate =
deferred
endef
define immediate ?=
deferred
endef
define immediate :=
immediate
endef
define immediate ::=
immediate
endef
define immediate +=
deferred or immediate
endef
define immediate !=
immediate
endef
For the append operator, ‘+=’, the right-hand side is considered immediate if the variable was previously set as a simple variable (‘:=’ or ‘::=’), and deferred otherwise.
```
当变量使用追加符（`+=`）时，如果此前这个变量是一个简单变量（使用`:=`定义的）则认为它是立即展开的，其他情况时都被认为是“延后“展开的变量。  
- `=` 是最基本的赋值  
- `:=` 是覆盖之前的值  
- `?=` 是如果没有被赋值过就赋予等号后面的值
- `+=` 是添加等号后面的值


## Two Flavors of Variables
```
•	recursively expanded variables, also known as “deferred” way.  

The value you specify is installed verbatim; if it contains references 
to other variables, these references are expanded whenever this 
variable is substituted.
such as ‘=’, 

•	simply expanded variables.  
The value of a simply expanded variable is scanned once and for all, 
expanding any references to other variables and functions, when the 
variable is defined. The actual value of the simply expanded variable 
is the result of expanding the text that you write. It does not 
contain any references to other variables; it contains their values as 
of the time this variable was defined.
such as ‘: =’, ‘:: =’ 

•	=   :  recursively expanded  
•	:=/::=   :  simply expanded  

•	?=   : variable to be set to a value only if it’s not already set, 
•	!=    : assignment operator ‘!=’ can be used to execute a shell 
script and set a variable to its output.
The resulting string is then placed into the named 
recursively-expanded variable

•	+=     : add more text to the value of a variable already defined 
When the variable in question has not been defined before, ‘+=’ acts 
just like normal ‘=’: it defines a recursively-expanded variable. 

However, when there is a previous definition, exactly what ‘+=’ does 
depends on what flavor of variable you defined originally.

the pattern rule has a pattern of just ‘%’, so it matches any target 
$$@ and $$% evaluate, respectively, to the file name of the target 
and, when the target is an archive member, the target member name. 

The $$< variable evaluates to the first prerequisite in the first rule 
for this target. 
$$^ and $$+ evaluate to the list of all prerequisites of rules that 
have already appeared for the sametarget ($$+ with repetitions and $$^ 
without)
for static pattern rules the $$* variable is set to the pattern stem. 

As with explicit rules, $$? is not available and expands to the empty 
string.
```

## 通配符
- `‘*’, ‘?’ and ‘[...]’`：  
Wildcard expansion is performed by make automatically in targets and in prerequisites.
In recipes, the shell is responsible for wildcard expansion. 
In other contexts, wildcard expansion happens only if you request it explicitly with the 
wildcard function.

Wildcard expansion does not happen when you define a variable.

One use of the wildcard function is to get a list of all the C source files in a directory, 
like this:
`$(wildcard *.c)`  
We can change the list of C source files into a list of object files by replacing the `‘.c’` suffix with `‘.o’` in the result, like this:
`$(patsubst %.c,%.o,$(wildcard *.c))`  
(Here we have used another function, patsubst.


## `PHONY`
A phony target is one that is not really the name of a file; rather it is just a name for a 
recipe to be executed when you make an explicit request.
two reasons to use a phony target: 
to avoid a conflict with a file of the same name, and to improve performance

```
.PHONY: clean  
clean:  
rm *.o temp  
```
In this example, the clean target will not work properly if a file named clean is ever 
created in this directory. Since it has no prerequisites, clean would always be considered 
up to date and its recipe would not be executed


`$(MAKE) -C $(KERNEL_SRC) M=$(PWD) modules`  
当make的目标为all时，`-C $(KDIR)` 指明跳转到内核源码目录下读取那里的Makefile；`M=$(PWD)` 表明然后返回到当前目录继续读入、执行当前的Makefile。  
`$(MAKE)` 指向当前使用的Make工具. 


## Variables

Common variables used as names of programs:
Used in implicit rules

Variables|Description
:----:|:-------
`AR`	|Archive-maintaining program; default ‘ar’.
`AS`	|Program for compiling assembly files; default ‘as’.
`CC`	|Program for compiling C programs; default ‘cc’.
`CXX`	|Program for compiling C++ programs; default ‘g++’.
`CPP`	|Program for running the C preprocessor, with results to standard output; default`‘$(CC) -E’`.
`RM` 	|Command to remove a file; default ‘rm -f’.

	

Variables whose values are additional arguments for the programs above.
The default values for all of these is the empty string, unless otherwise noted.  

Used in implicit rules  

Flags|Description
:----:|:-------
`ARFLAGS`	| Flags to give the archive-maintaining program; default ‘rv’.  
`ASFLAGS`	| Extra flags to give to the assembler (when explicitly invoked on a ‘.s’ or ‘.S’file).  
`CFLAGS`	| Extra flags to give to the C compiler
`CXXFLAGS`	| Extra flags to give to the C++ compiler.
`CPPFLAGS`	| Extra flags to give to the C preprocessor and programs that use it (the C andFortran compilers).
`LDFLAGS`	| Extra flags to give to compilers when they are supposed to invoke the linker, ‘ld’, such as -L. Libraries (-lfoo) should be added to the LDLIBS variable instead.
`LDLIBS`	| Library flags or names given to compilers when they are supposed to invoke the linker, ‘ld’. LOADLIBES is a deprecated (but still supported) alternative to LDLIBS. Non-library linker flags, such as -L, should go in the LDFLAGS variable.
	
Automatic variables:


Variable|Description
:----:|:-------
`$@`	|The file name of the target of the rule. If the target is an archive member, then `$@` is the name of the archive file. In a pattern rule that has multiple targets, `$@` is the name of whichever target caused the rule’s recipe to be run.
`$%`	|The target member name, when the target is an archive member.For example, if the target is foo.a(bar.o) then `$%` is bar.o and `$@` is foo.a. `$%` is empty when the target is not an archive member.
`$<` | The name of the first prerequisite.If the target got its recipe from an implicit rule, this will be the first prerequisite added by the implicit rule
`$?`	| The names of all the prerequisites that are newer than the target, with spaces between them. For prerequisites which are archive members, only the named member is used
`$^`	| The names of all the prerequisites, with spaces between them. A target has only one prerequisite on each other file it depends on, no matter how many times each file is listed as a prerequisite. So if you list a prerequisite more than once for a target, the value of `$^` contains just one copy of the name.
`$+`	| This is like `$^`, but prerequisites listed more than once are duplicated in the order they were listed in the makefile.
`$\|`	| The names of all the order-only prerequisites, with spaces between them.
$*	|The names of all the order-only prerequisites, with spaces between them.
`$(@D)`	| `$@`的目录部分（不以斜杠作为结尾），如果`$@`中没有包含斜杠，其值为`“.”（当前目录）`
`$(@F)`	| `$@`的文件部分，相当于函数`$(notdir $@)`
	
	

 
## Gcc 编译选项
### 后缀文件
- `.o` : 标准目标文件，链接文件object，.o 就相当于windows里的obj文件 ，一个.c或.cpp文件对应一个.o文件  
- `.lo`: `libtool`生成的文件，被`libtool`用来生成共享库的

```
The ‘.lo’ file is a library object, which may be built into a shared library, and the ‘.o’ 
file is a standard object file 
The .lo file is the libtool object, which Libtool uses to determine what object file may be 
built into a shared library
```

```
Shared libraries, however, may only be built from position-independent code (PIC). So, 
special flags must be passed to the compiler to tell it to generate PIC rather than the 
standard position-dependent code.   
Since this is a library implementation detail, libtool hides the complexity of PIC compiler 
flags by using separate library object files (which end in ‘.lo’ instead of ‘.o’). On 
systems without shared libraries (or without special PIC compiler flags), these library 
object files are identical to “standard” object files.
```

- `.a` : 静态库文件archive，`.a` 是好多个`.o`合在一起,用于静态连接 ，即`STATIC mode`，多个`.a`可以链接生成一个exe的可执行文件  
- `.so` : 动态链接文件shared object。用于动态连接的,和windows的dll差不多，使用时才载入。



### 静态库和动态库
- 静态库：在链接阶段，会将汇编生成的目标文件.o与引用到的库一起链接打包到可执行文件中  
（1） 静态库对函数库的链接是放在编译时期完成的。  
（2） 程序在运行时与函数库再无瓜葛，移植方便。  
（3） 浪费空间和资源，因为所有相关的目标文件与牵涉到的函数库被链接合成一个可执行文件。  
（4） 静态库对程序的更新、部署和发布页会带来麻烦。如果静态库liba.lib更新了，所以使用它的应用程序都需要重新编译、发布给用户（对于玩家来说，可能是一个很小的改动，却导致整个程序重新下载，全量更新）
```
g++ -c StaticMath.cpp
ar -crv libstaticmath.a StaticMath.o
```
- 动态库：程序编译时并不会被连接到目标代码中而是在程序运行是才被载入。**不同的应用程序如果调用相同的库，那么在内存里只需要有一份该共享库的实例**，规避了空间浪费问题。动态库在程序运行是才被载入，也解决了静态库对程序的更新、部署和发布页会带来麻烦。用户只需要更新动态库即可，**增量更新**。  
    - 动态库把对一些库函数的链接载入推迟到程序运行的时期。  
    - 可以实现进程之间的资源共享。（因此动态库也称为共享库）
    - 将一些程序升级变得简单。
    - 甚至可以真正做到链接载入完全由程序员在程序代码中控制（显示调用）。  
```
g++ -fPIC -c DynamicMath.cpp
```
`-fPIC` 创建与地址无关的编译程序（pic，position independent code），是为了能够在多个应用程序间共享。
```
g++ -shared -o libdynmath.so DynamicMath.o
g++ -fPIC -shared -o libdynmath.so DynamicMath.cpp
```


### 查看动态库和静态库中的符号——nm
Func : 列出.o .a .so中的符号信息，包括诸如符号的值，符号类型及符号名称等。所谓符号，通常指定义出的函数，全局变量等等。
```
Usage : nm [option(s)] [file(s)]
```
有用的options:  
- `-A` 在每个符号信息的前面打印所在对象文件名称；
- `-C` 输出demangle过了的符号名称；
- `-D` 打印动态符号；
- `-l` 使用对象文件中的调试信息打印出所在源文件及行号；
- `-n` 按照地址 / 符号值来排序；
- `-u` 打印出那些未定义的符号；

常见的符号类型:  
- `A` 该符号的值在今后的链接中将不再改变；
- `B` 该符号放在BSS段中，通常是那些未初始化的全局变量；
- `D` 该符号放在普通的数据段中，通常是那些已经初始化的全局变量；
- `T` 该符号放在代码段中，通常是那些全局非静态函数；
- `U` 该符号未定义过，需要自其他对象文件中链接进来；
- `W` 未明确指定的弱链接符号；同链接的其他对象文件中有它的定义就用上，否则就用一个系统特别指定的默认值。  

**ldd**  
作用：用来查看程式运行所需的共享库,常用来解决程式因缺少某个库文件而不能运行的一些问题。
```
/opt/app/todeav1/test$ldd test 
libstdc++.so.6 => /usr/lib64/libstdc++.so.6 (0x00000039a7e00000) 
libm.so.6 => /lib64/libm.so.6 (0x0000003996400000) 
libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00000039a5600000) 
libc.so.6 => /lib64/libc.so.6 (0x0000003995800000) 
/lib64/ld-linux-x86-64.so.2 (0x0000003995400000)
```
第一列：程序需要依赖什么库 
第二列: 系统提供的与程序需要的库所对应的库 
第三列：库加载的开始地址


### gcc命令编译过程
 
（1）	预处理 Preprocessing  
预处理用于将所有的#include头文件以及宏定义替换成其真正的内容. 生成 .i
```
gcc -E -I./inc test.c -o test.i
```
`-E`是让编译器在预处理之后就退出，不进行后续编译过程；`-I`指定头文件目录，这里指定的是我们自定义的头文件目录；`-o`指定输出文件名。

（2）	编译 Compilation  
将经过预处理之后的程序转换成特定汇编代码(assembly code)的过程。
```
gcc -S -I./inc test.c -o test.s
```
`-S`让编译器在编译之后停止，不进行后续过程。

（3）	汇编 Assemble  
汇编过程将上一步的汇编代码转换成机器码(machine code)，这一步产生的文件叫做目标文件
```
gcc -c test.s -o test.o
```
（4）	链接 Linking  
链接过程将多个目标文以及所需的库文件(`.so`等)链接成最终的可执行文件(executable file)

 
### gcc常用选项

选项|含义
:----:|:-------
`--help` `--target-help`	| 显示 gcc 帮助说明。‘target-help’是显示目标机器特定的命令行选项。
`--version` |	显示 gcc 版本号和版权信息 。
`-o  outfile`	| 输出到指定的文件。
`-x  language`	|指明使用的编程语言。允许的语言包括：c c++ assembler none 。 ‘none’意味着恢复默认行为，即根据文件的扩展名猜测源文件的语言。
`-v`	| 打印较多信息，显示编译器调用的程序。
`-###`	| 与 -v 类似，但选项被引号括住，并且不执行命令。
`-E`	| 仅作预处理，不进行编译、汇编和链接。
`-S`	| 仅编译到汇编语言，不进行汇编和链接。
`-c`	| 编译、汇编到目标代码，不进行链接。
`-pipe`	| 使用管道代替临时文件。
`-combine`	| 将多个源文件一次性传递给汇编器。


命令| 描述
:----:|:-------
`-l name` `-lname`	| 进行链接时搜索名为library的库。libname.so / libname.a,  VPATH search path  例子： `$ gcc test.c -lm -o test`. `.so/.a`都存在时，根据编译方式选择(`-static / -shared`)
`-Idir`	| 把dir加入到搜索头文件的路径列表中。例子： `$ gcc test.c -I../inc -o test`
`-Ldir`	| 把dir加入到搜索库文件的路径列表中。例子： `$ gcc -I/home/foo -L/home/foo -ltest test.c -o test`
`-Dname` | 预定义一个名为name的宏，值为1。例子： `$ gcc -DTEST_CONFIG test.c -o test`
`-Dname=definition`	| 预定义名为name，值为definition的宏。
`-ggdb` `-ggdblevel`|为调试器 gdb 生成调试信息。level可以为1，2，3，默认值为2。
`-g` `-glevel`	| 生成操作系统本地格式的调试信息。-g 和 -ggdb 并不太相同， -g 会生成 gdb 之外的信息。level取值同上。
`-s`	| 去除可执行文件中的符号表和重定位信息。用于减小可执行文件的大小。
`-shared`	|（1）可以生成动态库文件 （2）进行动态编译，尽可能链接动态库，只有没有动态库时才会链接同名静态库
`-fPIC (-fpic)`	| 生成使用相对地址的位置无关的目标文件（Position Independent Code）。然后通常使用gcc的-static选项 从该PIC目标文件生成动态库文件
`-M`	| 告诉预处理器输出一个适合make的规则，用于描述各目标文件的依赖关系。对于每个源文件，预处理器输出 一个make规则，该规则的目标项(target)是源文件对应的目标文件名，依赖项(dependency)是源文件中 `#include引用的所有文件。生成的规则可以是单行，但如果太长，就用`\'-换行符续成多行。规则显示在标准输出，不产生预处理过的C程序。
`-C`	| 告诉预处理器不要丢弃注释。配合`-E'选项使用。
`-P`	| 告诉预处理器不要产生`#line'命令。配合`-E'选项使用。
`-static`	| 在支持动态链接的系统上，阻止连接共享库。该选项在其它系统上无效。
`-nostdlib`	| 不连接系统标准启动文件和标准库文件，只把指定的文件传递给连接器。

**注意**: `-fPIC`是生成.o时使用，`-shared`是用来生成动态链接库的
- `libz`      压缩库（Z）
- `librt`     实时库（real time）：shm_open系列
- `libm`     数学库（math）
- `libc`     标准C库（C lib）

`--whole-archive` 和 `--no-whole-archive` 是ld专有的命令行参数，gcc 并不认识，要通gcc传递到 ld，需要在他们前面加`-Wl`，字串。  
`--whole-archive` 可以把 在其后面出现的静态库包含的函数和变量输出到动态库，`--no-whole-archive` 则关掉这个特性。
`-Wl`选项告诉编译器将后面的参数传递给链接器。    
`-soname`则指定了动态库的soname(简单共享名，Short for shared object name)  
soname的关键功能是它提供了兼容性的标准：
当要升级系统中的一个库时，并且新库的soname和老库的soname一样，用旧库链接生成的程序使用新库依然能正常运行。这个特性使得在Linux下，升级使得共享库的程序和定位错误变得十分容易。


### Warnings

选项| 描述
:----:|:-------
`-Wall`	| 会打开一些很有用的警告选项，建议编译时加此选项。
`-W` `-Wextra`	| 打印一些额外的警告信息。
`-w`	| 禁止显示所有警告信息。
`-Wshadow`	| 当一个局部变量遮盖住了另一个局部变量，或者全局变量时，给出警告。很有用的选项，建议打开。 -Wall 并不会打开此项。
`-Wpointer-arith`	| 对函数指针或者void *类型的指针进行算术操作时给出警告。也很有用。 `-Wall` 并不会打开此项。
`-Wcast-qual`	| 当强制转化丢掉了类型修饰符时给出警告。 -Wall 并不会打开此项。
`-Waggregate-return` | 如果定义或调用了返回结构体或联合体的函数，编译器就发出警告。
`-Winline`	|无论是声明为 inline 或者是指定了`-finline-functions` 选项，如果某函数不能内联，编译器都将发出警告。如果你的代码含有很多 inline 函数的话，这是很有用的选项。
`-Werror`	|把警告当作错误。出现任何警告就放弃编译。
`-Wunreachable-code`	|如果编译器探测到永远不会执行到的代码，就给出警告。也是比较有用的选项。
`-Wcast-align`	| 一旦某个指针类型强制转换导致目标所需的地址对齐增加时，编译器就发出警告。
`-Wundef`	| 当一个没有定义的符号出现在 `#if` 中时，给出警告。
`-Wredundant-decls`	| 如果在同一个可见域内某定义多次声明，编译器就发出警告，即使这些重复声明有效并且毫无差别。


### Optimization

选项| 描述
:----:|:-------
`-O0`	| 禁止编译器进行优化。默认为此项。
`-O`  `-O1`	| 尝试优化编译时间和可执行文件大小。
`-O2`	| 更多的优化，会尝试几乎全部的优化功能，但不会进行“空间换时间”的优化方法。
`-O3`	| 在 -O2 的基础上再打开一些优化选项：`-finline-functions`， `-funswitch-loops` 和 `-fgcse-after-reload`。
`-Os`	| 对生成文件大小进行优化。它会打开 -O2 开的全部选项，除了会那些增加文件大小的。
`-finline-functions`	|把所有简单的函数内联进调用者。编译器会探索式地决定哪些函数足够简单，值得做这种内联。
`-fstrict-aliasing`	|施加最强的别名规则（aliasing rules）。


### Standard

选项| 描述
:----:|:-------
`-ansi`	|支持符合ANSI标准的C程序。这样就会关闭GNU C中某些不兼容ANSI C的特性。
`-std=c89` `-iso9899:1990`	| 指明使用标准 ISO C90 作为标准来编译程序。
`-std=c99` `-std=iso9899:1999`	| 指明使用标准 ISO C99 作为标准来编译程序。
`-std=c++98`	| 指明使用标准 C++98 作为标准来编译程序。
`-std=gnu9x` `-std=gnu99`	| 使用 ISO C99 再加上 GNU 的一些扩展。
`-fno-asm`	| 不把asm, inline或typeof当作关键字，因此这些词可以用做标识符。用 __asm__， __inline__和__typeof__能够替代它们。 `-ansi' 隐含声明了`-fno-asm'。
`-fgnu89-inline`	|告诉编译器在 C99 模式下看到 inline 函数时使用传统的 GNU 句法。


### C options	

options|描述
:----:|:------- 
`-fsigned-char`  `-funsigned-char`	| 把char定义为有/无符号类型，如同signed char/unsigned char。
`-traditional`	| 尝试支持传统C编译器的某些方面。详见GNU C手册。
`-fno-builtin` `-fno-builtin-function`	|不接受没有 __builtin_ 前缀的函数作为内建函数。
`-trigraphs`	| 支持ANSI C的三联符（ trigraphs）。`-ansi'选项隐含声明了此选项。
`-fsigned-bitfields` `-funsigned-bitfields`	| 如果没有明确声明`signed'或`unsigned'修饰符，这些选项用来定义有符号位域或无符号位域。缺省情况下，位域是有符号的，因为它们继承的基本整数类型，如int，是有符号数。
`-Wstrict-prototypes`	| 如果函数的声明或定义没有指出参数类型，编译器就发出警告。很有用的警告。
`-Wmissing-prototypes`	| 如果没有预先声明就定义了全局函数，编译器就发出警告。即使函数定义自身提供了函数原形也会产生这个警告。这个选项 的目的是检查没有在头文件中声明的全局函数。
`-Wnested-externs`	| 如果某extern声明出现在函数内部，编译器就发出警告。
`-Wwrite-strings`	| deprecated conversion from string constant to ‘char*’
	

### C++ options

options|描述
:----:|:-------
`-ffor-scope`	| 从头开始执行程序，也允许进行重定向。
`-fno-rtti`	| 关闭对 dynamic_cast 和 typeid 的支持。如果你不需要这些功能，关闭它会节省一些空间。
`-Wctor-dtor-privacy`	| 当一个类没有用时给出警告。因为构造函数和析构函数会被当作私有的。
`-Wnon-virtual-dtor`	| 当一个类有多态性，而又没有虚析构函数时，发出警告。-Wall会开启这个选项。
`-Wreorder`	|如果代码中的成员变量的初始化顺序和它们实际执行时初始化顺序不一致，给出警告。
`-Wno-deprecated`	| 使用过时的特性时不要给出警告。
`-Woverloaded-virtual`	|如果函数的声明隐藏住了基类的虚函数，就给出警告。


### Machine Dependent Options (Intel)

options|描述
:----:|:-------
`-mtune=cpu-type`	| 为指定类型的 CPU 生成代码。cpu-type可以是：i386，i486，i586，pentium，i686，pentium4 等等。
`-msse` `-msse2` `-mmmx` `-mno-sse` `-mno-sse2` `-mno-mmx`	|使用或者不使用MMX，SSE，SSE2指令。
`-m32` `-m64`	|生成32位/64位机器上的代码。
`-mpush-args` `-mno-push-args`	|（不）使用 push 指令来进行存储参数。默认是使用。
`-mregparm=num`	|当传递整数参数时，控制所使用寄存器的个数。

### ld options

•	`-( archives -)`  
`--start-group archives --end-group`
```
The archives should be a list of archive files. They may be either explicit file names, or 
-l options.
The specified archives are searched repeatedly until no new undefined references are 
created. Normally, an archive is searched only once in the order that it is specified on 
the command line. If a symbol in that archive is needed to resolve an undefined symbol 
referred to by an object in an archive that appears later on the command line, the linker 
would not be able to resolve that reference. By grouping the archives, they all be searched 
repeatedly until all possible references are resolved.
Using this option has a significant performance cost. It is best to use it only when there 
are unavoidable circular references between two or more archives.
```
将反复查找group的archive文件，直到没有未定义的引用出现，以此来解决相互依赖。

### Vpath

Makefile文件中的特殊变量“VPATH”就是完成这个功能的，如果没有指明这个变量，make只会在当前的目录中去找寻依赖文件和目标文件。如果定义了这个变量，那么，make就会在当当前目录找不到的情况下，到所指定的目录中去找寻文件了。  
  	  ```VPATH = src:../headers```  
上面的的定义指定两个目录，`src`和`../headers`，make会按照这个顺序进行搜索。目录由**冒号**分隔。（当然，当前目录永远是最高优先搜索的地方）

另一个设置文件搜索路径的方法是使用make的`vpath`关键字（注意，它是全小写的）， 这不是变量，这是一个make的关键字，这和上面提到的那个`VPATH`变量很类似，但是它更为灵活。它可以指定不同的文件在不同的搜索目录中。这是一个很 灵活的功能。它的使用方法有三种：  
- `vpath <pattern> <directories>`
    为符合模式<pattern>的文件指定搜索目录<directories>。
- `vpath <pattern>`
    清除符合模式<pattern>的文件的搜索目录。
- `vpath`
    清除所有已被设置好了的文件搜索目录。  
vapth使用方法中的`<pattern>`需要包含`%`字符。`%`的意思是匹 配零或若干字符，例如，`%.h`表示所有以`.h`结尾的文件。`<pattern>`指定了要搜索的文件集， 而`<directories>`则指定了`<pattern>`的文件集的搜索的目录。

 
## GCC

### __attribute__

GCC specific syntaxes 
- `__attribute__((constructor))` syntax : This particular GCC syntax, when used with a 
function, executes the same function at the startup of the program, i.e before main() 
function.
- `__attribute__((destructor))` syntax : This particular GCC syntax, when used with a 
function, executes the same function just before the program terminates through _exit, i.e 
after main() function.

Explanation:  
The way constructors and destructors work is that the shared object file contains special 
sections (.ctors and .dtors on ELF) which contain references to the functions marked with 
the constructor and destructor attributes, respectively. When the library is loaded/unloaded, 
the dynamic loader program checks whether such sections exist, and if so, calls the 
functions referenced therein.

Few points regarding these are worth noting :
- `__attribute__((constructor))` runs when a shared library is loaded, typically during 
program startup.
- `__attribute__((destructor))` runs when the shared library is unloaded, typically at 
program exit.
- The two parentheses are presumably to distinguish them from function calls.
- `__attribute__` is a GCC specific syntax;not a function or a macro.

```
constructor
destructor
constructor (priority)
destructor (priority)
```
The `constructor` attribute causes the function to be called automatically before execution 
enters `main()`. Similarly, the destructor attribute causes the function to be called 
automatically after `main()` has completed or `exit()` has been called. Functions with these 
attributes are useful for initializing data that will be used implicitly during the execution of 
the program.  
You may provide an optional integer priority to control the order in which constructor and 
destructor functions are run. **A constructor with a smaller priority number runs before a constructor with a larger priority number; the opposite relationship holds for destructors**. 
So, if you have a constructor that allocates a resource and a destructor that deallocates the 
same resource, both functions typically have the same priority. The priorities for constructor 
and destructor functions are the same as those specified for namespace-scope C++ objects (see C++ 
Attributes).    

These attributes are not currently implemented for Objective-C. 
The priority values you give **must be greater than 100** as the compiler reserves priority 
values between 0-100 for implementation. Aconstructor or destructor specified with priority 
executes before a constructoror destructor specified without priority.   

(1)	Static 不能影响constructor的顺序  
(2)	Static 变量看起来也在constructor后面才执行 (system dependent)
```
class Foo{
    public:
        Foo() { printf("static foo\n");}
};
static Foo foo;
__attribute__ ((constructor)) static void cons_foo_null(void){
    printf("constructor cons_foo_NULL \n");
}
__attribute__ ((constructor (102))) void cons_foo_102(void){
    printf("constructor cons_foo_102 \n");
}
__attribute__ ((constructor (101))) void cons_foo_101(void){
    printf("constructor cons_foo_101 \n");
}
```
Results:
```
constructor cons_foo_101
constructor cons_foo_102
static foo
constructor cons_foo_NULL
This is main function!
```

https://stackoverflow.com/questions/2053029/how-exactly-does-attribute-constructor-work  

It seems pretty clear that it is supposed to set things up.
1.	When exactly does it run?
2.	Why are there two parentheses?
3.	Is __attribute__ a function? A macro? Syntax?
4.	Does this work in C? C++?
5.	Does the function it works with need to be static?
6.	When does __attribute__((destructor)) run?

Example in Objective-C:

```
__attribute__((constructor))
static void initialize_navigationBarImages() {
  navigationBarImages = [[NSMutableDictionary alloc] init];
}

__attribute__((destructor))
static void destroy_navigationBarImages() {
  [navigationBarImages release];
}
```

```
1.	It's run when a shared library is loaded, typically during program startup.
2.	That's how all GCC attributes are; presumably to distinguish them from function calls. 

3.	GCC-specific syntax.
4.	Yes, this works in C and C++.
5.	No, the function does not need to be static.
6.	The destructor is run when the shared library is unloaded, typically at program exit.
```

```
So, the way the constructors and destructors work is that the shared object file contains 
special sections (`.ctors` and `.dtors` on ELF) which contain references to the functions 
marked with the constructor and destructor attributes, respectively. 
When the library is loaded/unloaded the dynamic loader program (ld.so or somesuch) checks 
whether such sections exist, and if so, calls the functions referenced therein. 

Come to think of it, there is probably some similar magic in the normal static linker, 
so that the same code is run on startup/shutdown regardless if the user chooses static or 
dynamic linking.  
`.init/.fini` isn't deprecated. It's still part of the the ELF standard and I'd dare say 
it will be forever. Code in `.init/.fini` is run by the loader/runtime-linker when code is 
loaded/unloaded. I.e. on each ELF load (for example a shared library) code in `.init` will be run. 
It's still possible to use that mechanism to achieve about the same thing as with 
`__attribute__((constructor))/((destructor))`. It's old-school but it has some benefits.  

`.ctors/.dtors` mechanism for example require support by system-rtl/loader/linker-script. 
This is far from certain to be available on all systems, for example deeply embedded systems 
where code executes on bare metal. I.e. even if __attribute__((constructor))/((destructor)) is 
supported by GCC, it's not certain it will run as it's up to the linker to organize it and to 
the loader (or in some cases, boot-code) to run it. To use `.init/.fini` instead, 
the easiest way is to use linker flags: `-init` & `-fini` (i.e. from GCC command line, 
syntax would be `-Wl -init my_init -fini my_fini`).

`.ctors/.dtors` mechanism for example require support by system-rtl/loader/linker-script. 
This is far from certain to be available on all systems, for example deeply embedded systems 
where code executes on bare metal. I.e. even if `__attribute__((constructor))/((destructor))` 
is supported by GCC, it's not certain it will run as it's up to the linker to organize it and 
to the loader (or in some cases, boot-code) to run it. To use .init/.fini instead, the easiest 
way is to use linker flags: `-init` & `-fini` (i.e. from GCC command line, syntax would be 
`-Wl -init my_init -fini my_fini`).
```

```
#define SECTION( S ) __attribute__ ((section ( S )))
void test(void) {
   printf("Hello\n");
}
void (*funcptr)(void) SECTION(".ctors") =test;
void (*funcptr2)(void) SECTION(".ctors") =test;
void (*funcptr3)(void) SECTION(".dtors") =test;
```


### Shared linked library

Debugging Cheat Sheet
If you ever get this error when running an executable:

```
$ ./main
./main: error while loading shared libraries: librandom.so: cannot open shared object file: No such file or directory 
```

You can try doing the following:
- Find out what dependencies are missing with `ldd <executable>`.  
- If you don’t identify them, you can check if they are direct dependencies by running 
`readelf -d <executable> | grep NEEDED`.  
- Make sure the dependencies actually exist. Maybe you forgot to compile them or move them 
to a libs directory?  
- Find out where dependencies are searched by using `LD_DEBUG=libs ldd <executable>`.  
- If you need to add a directory to the search:
    - Ad-hoc: add the directory to the LD_LIBRARY_PATH environment variable.
    - Baked in the file: add the directory to the executable or shared library’s rpath or 
    runpath by passing `-Wl,-rpath,<dir>` (for `rpath`) or `-Wl,--enable-new-dtags`,`-rpath,
    <dir>`(for `runpath`). Use `$ORIGIN` for paths relative to the executable. 

- If `ldd` shows that no dependencies are missing, see if your application has elevated privileges. If so, `ldd` might lie. See security concerns above.

### LD_DEBUG
**shared (dynamically) loaded libraries debug tool**.


`LD_DEBUG=help cat`
Valid options for the LD_DEBUG environment variable are:

- `libs`        display library search paths
- `reloc`       display relocation processing
- `files`       display progress for input file
- `symbols`     display symbol table processing
- `bindings`    display information about symbol binding
- `versions`    display version dependencies
- `all`         all previous options combined
- `statistics`  display relocation statistics
- `unused`      determined unused DSOs
- `help`        display this help message and exit

To direct the debugging output into a file instead of standard output a filename can be 
specified using the `LD_DEBUG_OUTPUT` environment variable.

### ld.so
http://man7.org/linux/man-pages/man8/ld.so.8.html
If a shared object dependency does not contain a slash, then it is searched for in the 
following order:

- Using the directories specified in the DT_RPATH dynamic section attribute of the binary if 
present and DT_RUNPATH attribute does not exist.  Use of DT_RPATH is deprecated.

- Using the environment variable `LD_LIBRARY_PATH`, unless the executable is being run in 
secure-execution mode (see below), in which case this variable is ignored.

- Using the directories specified in the `DT_RUNPATH` dynamic section  attribute of the binary 
if present.  Such directories are searched  only to find those objects required by `DT_NEEDED` (direct dependencies) entries and do not apply to those objects' children, which must themselves 
have their own DT_RUNPATH entries.  This is  unlike DT_RPATH, which is applied to searches for 
all children in
the dependency tree.

- From the cache file `/etc/ld.so.cache`, which contains a compiled list of candidate shared 
objects previously found in the augmented library path.  If, however, the binary was linked with 
the -z nodeflib linker option, shared objects in the default paths are skipped.  Shared objects 
installed in hardware capability directories (see below) are preferred to other shared objects.

- In the default path `/lib`, and then `/usr/lib`.  (On some 64-bit  architectures, the default 
paths for 64-bit shared objects are  `/lib64`, and then `/usr/lib64`.)  If the binary was linked 
with the -z nodeflib linker option, this step is skipped.



## 模式规则

### 模式规则
模式规则类似于普通规则。只是在模式规则中，目标名中需要包含有模式字符`%`（一个），包含有模式字符`%`的目标被用来匹配一个文件名，`%`可以匹配任何非空字符串。规则的依赖文件中同样可以使用`%`，依赖文件中模式字符`%`的取值情况由目标中的`%`来决定。例如：对于模式规则`%.o : %.c`，它表示的含义是：所有的`.o`文件依赖于对应的.c文件。我们可以使用模式规则来定义隐含规则。  
**要注意的是**：模式字符`%`的匹配和替换发生在规则中所有变量和函数引用展开之后，变量和函数的展开一般发生在make读取Makefile时（变量和函数的展开可参考第五 章 使用变量 和 第七章 make的函数），而模式规则中的`%`的匹配和替换则发生在make执行时。

### 模式规则介绍
在模式规则中，目标文件是一个带有模式字符`%`的文件，使用模式来匹配目标文件。文件名中的模式字符`%`可以匹配任何非空字符串，除模式字符以外的部分要求一致。例如：`%.c`匹配所有以“.c”结尾的文件（匹配的文件名长度最少为3个字母），`s%.c`匹配所有第一个字母为`s`，而且必须以`.c`结尾的文件，文件名长度最小为5个字符（模式字符`%`至少匹配一个字符）。在目标文件名中`%`匹配的部分称为“茎”（前面已经提到过，参考4.12 静态模式一节）。使用模式规则时，目标文件匹配之后得到“茎”，依赖根据“茎”产生对应的依赖文件，这个依赖文件必须是存在的或者可被创建的。  
因此，一个模式规则的格式为： 
```
%.o : %.c ; COMMAND... 
```
这个模式规则指定了如何由文件`N.c`来创建文件`N.o`，文件`N.c`应该是已存在的或者可被创建的。  
模式规则中依赖文件也可以不包含模式字符`%`。当依赖文件名中不包含模式字符`%`时，其含义是所有符合目标模式的目标文件都依赖于一个指定的文件（例如：`%.o : debug.h`，表示所有的`.o`文件都依赖于头文件`debug.h`）。这样的模式规则在很多场合是非常有用的。  
同样一个模式规则可以存在多个目标。多目标的模式规则和普通多目标规则有些不同，普通多目标规则的处理是将每一个目标作为一个独立的规则来处理，所以多个目标就就对应多个独立的规则（这些规则各自有自己的命令行，各个规则的命令行可能相同）。但对于多目标模式规则来说，所有规则的目标共同拥有依赖文件和规则的命令行，当文件符合多个目标模式中的任何一个时，规则定义的命令就有可能将会执行；因为多个目标共同拥有规则的命令行，因此一次命令执行之后，规则不会再去检查是否需要重建符合其它模式的目标。看一个例子： 
```
#sample Makefile 
Objects = foo.o bar.o 
CFLAGS := -Wall 
%x : CFLAGS += -g 
%.o : CFLAGS += -O2 
%.o %.x : %.c 
$(CC) $(CFLAGS) $< -o $@ 
```

当在命令行中执行`make foo.o foo.x`时，会看到只有一个文件`foo.o`被创建了，同时make会提示`foo.x`文件是最新的（其实`foo.x`并没有被创建）。此过程表明了多目标的模式规则在make处理时是被作为一个整体来处理的。这是多目标模式规则和多目标的普通规则的区别之处。大家不妨将上边的例子改为普通多目标规则试试看将会得到什么样的结果。  
最后需要说明的是： 
1. 模式规则在Makefile中的顺序需要注意，当一个目标文件同时符合多个目标模式时，make将会把第一个目标匹配的模式规则作为重建它的规则。
2. Makefile中明确指定的模式规则会覆盖隐含模式规则。就是说如果在Makefile中出现了一个对目标文件合适可用的模式规则，那么make就不会再为这个目标文件寻找其它隐含规则，而直接使用在Makefile中出现的这个规则。在使用时，明确规则永远优先于隐含规则。
3. 另外，依赖文件存在或者被提及的规则，优先于那些需要使用隐含规则来创建其依赖文件的规则。

