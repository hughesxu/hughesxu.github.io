---
title:  "学习笔记-CMake"
date:   2021-05-01 09:00:00 +0800
categories: [Language, CMake]
tags: [Learning-Note]
---


## 帮助文档
1.	cmake官方新手tutorial: [https://cmake.org/cmake/help/latest/guide/tutorial/index.html](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)
2.	cmake 添加头文件目录，链接动态、静态库
3.	官方文档
4.	cmake 语法


## 技巧
1.	cmake命令是不区分大小写的，但是变量区分。
2.	判断编译器类型
```
cmake if ("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang") MESSAGE("Clang") 
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU") MESSAGE("GNU") 
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Intel") MESSAGE("Intel") 
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC") MESSAGE("MSVC") 
endif()
```


## 命令行选项
### `-D`
1.	`-D` 相当于就是定义, `-D` 可以理解为告诉cmake后边我要定义一些参数了, 你每定义一个就在前边加上`-D`就是了
2.	`CMAKE_BUILD_TYPE`这种东西往往是在`CMakeList.txt`中定义的, 这个是你要编译的类型, 一般的选择有`debug`, `release`, 但是不确定
3.	`CMAKE_INSTALL_PREFIX` 这个是安装路径.

例子
```
cmake -DCMAKE_BUILD_TYPE=Debug
```

## 编译选项
在cmake脚本中，设置编译选项可以通过`add_compile_options`命令，也可以通过`set`命令修改`CMAKE_CXX_FLAGS`或`CMAKE_C_FLAGS`。使用这两种方式在有的情况下效果是一样的，但请注意它们还是有区别的：  
1.	`add_compile_options`命令添加的编译选项是针对所有编译器的(包括c和c++编译器)，
2.	而set命令设置`CMAKE_C_FLAGS`或`CMAKE_CXX_FLAG`S变量则是分别只针对c和c++编译器的。

例子
```
#判断编译器类型,如果是gcc编译器,则在编译选项中加入c++11支持
if(CMAKE_COMPILER_IS_GNUCXX)
    set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")
    message(STATUS "optional:-std=c++11")
endif(CMAKE_COMPILER_IS_GNUCXX)
```

### add_compile_options
语法
```
add_compile_options(<option> ...)
```
```
# 例子

add_compile_options(-Wall -Wextra -pedantic -Werror -g)
```

### add_compile_definition
待补充

### option & add_definition
语法
```
# Provides an option for the user to select as ON or OFF. 
If no initial <value> is provided, OFF is used. 
If <variable> is already set as a normal variable then the 
command does nothing 

option(<variable> "<help_text>" [value])

# Add -D define flags to the compilation of source files.
add_definitions(-DFOO -DBAR ...)
```

使用方法
1.	首先在CMakeList.txt中增加选项
```
cmake option(TEST_DEBUG "option for debug" OFF) 
if (TEST_DEBUG) 
add_definitions(-DTEST_DEBUG) 
endif()
```
2.	在cmake构造makefile的时候输入想要的参数
```
bash cmake -DTEST_DEBUG=ON ..
```
3.	源码中使用该定义
```
cmake // test.cpp
include "test.h"
ifdef TEST_DEBUG
...
endif
```

## 语法说明
### 列表和字符串
在CMake中基础的数据形式是字符串。CMake也支持字符串列表。  
列表通过分号分隔。譬如两个声明给变量`VAR`设同样的值：  
```
set(VAR a;b;c)
set(VAR a b c)
```
字符串列表可以通过`foreach`命令迭代或直接操控列表命令。

### 变量
CMake 支持简单的变量可以是字符串也可以是字符串列表。
变量参考使用`${VAR}`语法。多参数可以使用`set`命令组合到一个列表中。所有其他的命令通过空白分隔符传递命令来扩展列表，例如  
```
set(Foo a b c)
# 将 变量 Foo 设为 a b c, 并且如果Foo 传递给另一个命令
command(${Foo})
# 等同于
command(a b c)

# 如果要把参数列表传递给一个命令，且它是一个简单的参数只要加一个双引号就可以。
例如
command("${Foo}")
# 等价于
command("a b c")
```

### 控制流
像大多数语言一样,CMake 提供了控制流结构。  
CMake提供了三中控制流：
1.	条件控制流 if
```
cmake
some_command will be called if the variable's value is not:
empty, 0, N, NO, OFF, FALSE, NOTFOUND, or -NOTFOUND.
if(var) some_command(...) endif(var) 
```

2.	循环结构： foreach 和 while
```
cmake set(VAR a b c)
loop over a, b,c with the variable f
foreach(f ${VAR}) message(${f}) endforeach(f) 
```
3.	过程定义宏和函数（函数在2.6及更高的版本中有效）。函数对变量局部有效，宏是全局有效。
```
cmake
define a macro hello
macro(hello MESSAGE) message(${MESSAGE}) endmacro(hello)
call the macro with the string "hello world"
hello("hello world")
define a function hello
function(hello MESSAGE) message(${MESSAGE}) endfunction(hello) 
```
更多控制流信息参见命令 `if`, `while`, `foreach`, `macro`,`function`文档。

### 引号，字符串和转义
在CMake中原义字符串用双引号括起来。字符串可以是多行字符串，并在其中嵌入新的行。例如
```
set(MY_STRING "this is a string with a

 newline in

it")
```
也可以在一个字符串中转义字符和使用变量
```
set(VAR "

  hello

  world

")

message("\${VAR}= ${VAR}")

# prints out

${VAR}=

hello

world
```
同样支持标准C中的转义
```
message("\n\thello world")

#prints out

hello world
```
如果字符在引号之前是空格则原义字符串只是原义字符串。但是引号必须成对，例如
```
message(hell"o") -> prints hell"o"
message(hell\"o\") -> prints hello"o"
```

### 正则表达式
cmake可以使用正则表达式


## 常用命令
### cmake_minimum_required
cmake project 头文件必须存在这行命令， 例如
```
cmake_minimum_required(VERSION 3.10)
```


### project
设置项目名称project(Tutorial)
```
project(<PROJECT-NAME> [<language-name>...])
project(<PROJECT-NAME>
        [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
        [DESCRIPTION <project-description-string>]
        [HOMEPAGE_URL <url-string>]
        [LANGUAGES <language-name>...])
```
```
project(muduo C CXX)
```

```
Sets the name of the project, and stores it in the 
variable PROJECT_NAME. 
When called from the top-level CMakeLists.txt also stores 
the project name in the variable CMAKE_PROJECT_NAME.


Also sets the variables

    PROJECT_SOURCE_DIR, <PROJECT-NAME>_SOURCE_DIR
PROJECT_BINARY_DIR, <PROJECT-NAME>_BINARY_DIR


PROJECT_SOURCE_DIR:
Top level source directory for the current project.
This is the source directory of the most recent project() command.

Languages are enabled by the project() command. 
Language-specific built-in variables, such as 
CMAKE_CXX_COMPILER, CMAKE_CXX_COMPILER_ID etc are set by 
invoking the project() command. If no project command is 
in the top-level CMakeLists file, one will be implicitly 
generated. By default the enabled languages are C and CXX:

Several variables relate to the language components of a 
toolchain which are enabled. CMAKE_<LANG>_COMPILER is the 
full path to the compiler used for <LANG>. 
CMAKE_<LANG>_COMPILER_ID is the identifier used by CMake 
for the compiler and CMAKE_<LANG>_COMPILER_VERSION is the 
version of the compiler.
```

### set
语法
```
# Set Normal Variable
set(<variable> <value>... [PARENT_SCOPE])

# Set Environment Variable
# 这个环境变量只对当前cmake工程有效，对外界是无效的。
set(ENV{<variable>} [<value>])
```

例子
```
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -O3 -Wall -Wl,-rpath=/tools/lib64 -Wl,--dynamic-linker=/tools/lib/ld-2.17.so")
```

### message
语法
```
message([<mode>] "message to display" ...)
```

### mode关键字
```
The optional <mode> keyword determines the type of message:
|mode | explaination| |--|--:| 
|FATAL_ERROR|CMake Error, stop processing and generation. 
|SEND_ERROR|CMake Error, continue processing, but skip generation. 
|WARNING|CMake Warning, continue processing. 
|AUTHOR_WARNING|CMake Warning (dev), continue processing. 
|DEPRECATION|CMake Deprecation Error or Warning if 
variable CMAKE_ERROR_DEPRECATED or CMAKE_WARN_DEPRECATED 
is enabled, respectively, else no message. 
|(none) or NOTICE|Important message printed to stderr to 
attract user’s attention. 

|STATUS|一般就用这个，Ideally these should be concise, 
no more than a single line, but still informative. 
|VERBOSE|Detailed informational messages intended for 
project users. These messages should provide additional 
details that won’t be of interest in most cases, but which 
may be useful to those building the project when they want 
deeper insight into what’s happening. 

|DEBUG|Detailed informational messages intended for 
developers working on the project itself as opposed to 
users who just want to build it. These messages will not 
typically be of interest to other users building the 
project and will often be closely related to internal 
implementation details. 

|TRACE|Fine-grained messages with very low-level 
implementation details. Messages using this log level 
would normally only be temporary and would expect to be 
removed before releasing the project, packaging up the 
files, etc.
```

### aux_source_directory 查找源文件
```
# 找到所有dir目录下的源文件（不会递归遍历子文件夹），
# 源文件是.c文件（也就是makefile中可以生成.o的文件）
aux_source_directory(<dir> <variable>)
```

### add_library 指定生成链接文件
将指定的源文件（`CPP`文件）生成链接文件，然后添加到工程中去。
语法
```
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [source1] [source2 ...])
```
其中<name>表示库文件的名字，该库文件会根据命令里列出的源文件来创建。而`STATIC`、`SHARED`和`MODULE`的作用是指定生成的库文件的类型。
1.	`STATIC`库是目标文件的归档文件，在链接其它目标的时候使用。
2.	`SHARED`库会被动态链接（动态链接库），在运行时会被加载。
3.	`MODULE`库是一种不会被链接到其它目标中的插件，但是可能会在运行时使用`dlopen`-系列的函数。
4.	默认状态下，库文件将会在于源文件目录树的构建目录树的位置被创建(默认`static`)，该命令也会在这里被调用。

例子  
```
add_library(roland_pb CreateUDiskRequest.pb.cc)
add_executable(echo_client echo_client.cc)
target_link_libraries(echo_client uevent event uevent_base pthread roland_pb protobuf)
```

### add_subdirectory 添加子运行目录
在子文件夹添加了`library`或者`executable`之后，在上层目录添加`subdirectory`, 也可以在同一个`CMakeList.txt`中使用

Add a subdirectory to the build.
```
add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])
Adds a subdirectory to the build. 
The source_dir specifies the directory in which the source CMakeLists.txt and code files are located. 

If it is a relative path it will be evaluated with respect to the current directory (the typical usage), 
but it may also be an absolute path. 
```

### include_directories 添加头文件目录
它相当于g++选项中的-I参数的作用，也相当于环境变量中增加路径到`CPLUS_INCLUDE_PATH`变量的作用。

语法：  
```
include_directories([AFTER|BEFORE] [SYSTEM] dir1 [dir2 ...])
# 例子
include_directories(../../../thirdparty/comm/include)
```

### link_directories 添加需要链接的库文件目录
它相当于g++命令的-L选项的作用，也相当于环境变量中增加`LD_LIBRARY_PATH`的路径的作用。   
语法：
```
link_directories(directory1 directory2 ...)
# 例子
link_directories("/home/server/third/lib")
```

### find_library 查找库所在路径
语法：
```
# A short-hand signature is:
find_library (<VAR> name1 [path1 path2 ...])

# The general signature is:
find_library (
          <VAR>
          name | NAMES name1 [name2 ...] [NAMES_PER_DIR]
          [HINTS path1 [path2 ... ENV var]]
          [PATHS path1 [path2 ... ENV var]]
          [PATH_SUFFIXES suffix1 [suffix2 ...]]
          [DOC "cache documentation string"]
          [NO_DEFAULT_PATH]
          [NO_CMAKE_ENVIRONMENT_PATH]
          [NO_CMAKE_PATH]
          [NO_SYSTEM_ENVIRONMENT_PATH]
          [NO_CMAKE_SYSTEM_PATH]
          [CMAKE_FIND_ROOT_PATH_BOTH |
           ONLY_CMAKE_FIND_ROOT_PATH |
           NO_CMAKE_FIND_ROOT_PATH]
         )

# 例子如下
FIND_LIBRARY(RUNTIME_LIB rt /usr/lib  /usr/local/lib NO_DEFAULT_PATH)
```
cmake会在目录中查找，如果所有目录中都没有，值`RUNTIME_LIB`就会被赋为`NO_DEFAULT_PATH`


### find_program 查找程序所在路径

A short-hand signature is:
```
find_program (<VAR> name1 [path1 path2 ...])
```
The general signature is:
```
find_program (
          <VAR>
          name | NAMES name1 [name2 ...] [NAMES_PER_DIR]
          [HINTS path1 [path2 ... ENV var]]
          [PATHS path1 [path2 ... ENV var]]
          [PATH_SUFFIXES suffix1 [suffix2 ...]]
          [DOC "cache documentation string"]
          [REQUIRED]
          [NO_DEFAULT_PATH]
          [NO_PACKAGE_ROOT_PATH]
          [NO_CMAKE_PATH]
          [NO_CMAKE_ENVIRONMENT_PATH]
          [NO_SYSTEM_ENVIRONMENT_PATH]
          [NO_CMAKE_SYSTEM_PATH]
          [CMAKE_FIND_ROOT_PATH_BOTH |
           ONLY_CMAKE_FIND_ROOT_PATH |
           NO_CMAKE_FIND_ROOT_PATH]
         )

This command is used to find a program. 
A cache entry named by <VAR> is created to store the
result of this command. If the program is found the result 
is stored in the variable and the search will not be 
repeated unless the variable is cleared. If nothing is 
found, the result will be <VAR>-NOTFOUND.
```

### find_package  查找外部包
```
Finds and loads settings from an external project. 
<PackageName>_FOUND will be set to indicate whether the 
package was found.
```
用法：
```
find_package(<PackageName> [version] [EXACT] [QUIET] [MODULE]
             [REQUIRED] [[COMPONENTS] [components...]]
             [OPTIONAL_COMPONENTS components...]
             [NO_POLICY_SCOPE])
```

The `QUIET` option disables informational messages,  including those indicating that the package cannot be found if it is `not REQUIRED`. 

The `REQUIRED` option stops processing with an error message if the package cannot be found.


### find_path  查找包含给定文件的目录
```
find_path (<VAR> name1 [path1 path2 ...])
```
The general signature is:
```
find_path (
          <VAR>
          name | NAMES name1 [name2 ...]
          [HINTS path1 [path2 ... ENV var]]
          [PATHS path1 [path2 ... ENV var]]
          [PATH_SUFFIXES suffix1 [suffix2 ...]]
          [DOC "cache documentation string"]
          [REQUIRED]
          [NO_DEFAULT_PATH]
          [NO_PACKAGE_ROOT_PATH]
          [NO_CMAKE_PATH]
          [NO_CMAKE_ENVIRONMENT_PATH]
          [NO_SYSTEM_ENVIRONMENT_PATH]
          [NO_CMAKE_SYSTEM_PATH]
          [CMAKE_FIND_ROOT_PATH_BOTH |
           ONLY_CMAKE_FIND_ROOT_PATH |
           NO_CMAKE_FIND_ROOT_PATH]
         )

This command is used to find a directory containing the 
named file. A cache entry named by <VAR> is created to 
store the result of this command. If the file in a 
directory is found the result is stored in the variable 
and the search will not be repeated unless the variable is 
cleared. 
If nothing is found, the result will be <VAR>-NOTFOUND.
```

### link_libraries 添加需要链接的库文件路径，指定库文件用于链接
该指令的作用主要是指定要链接的库文件的路径，该指令有时候不一定需要。因为find_package和find_library指令可以得到库文件的绝对路径。不过你自己写的动态库文件放在自己新建的目录下时，可以用该指令指定该目录的路径以便工程能够找到。

语法：
```
link_libraries(library1 <debug | optimized> library2 ...)
# 直接是全路径
link_libraries(“/home/server/third/lib/libcommon.a”)
# 下面的例子，只有库名，cmake会自动去所包含的目录搜索
link_libraries(iconv)

# 传入变量
link_libraries(${RUNTIME_LIB})
# 也可以链接多个
link_libraries("/opt/MATLAB/R2012a/bin/glnxa64/libeng.so"　
"/opt/MATLAB/R2012a/bin/glnxa64/libmx.so")
可以链接一个，也可以多个，中间使用空格分隔.
```

### target_link_libraries 设置要链接的库文件的名称
语法：
```
target_link_libraries(<target> [item1 [item2 [...]]]
                      [[debug|optimized|general] <item>] ...)

# 以下写法都可以：
target_link_libraries(myProject comm)       # 连接libhello.so库，默认优先链接动态库
target_link_libraries(myProject libcomm.a)  # 显示指定链接静态库
target_link_libraries(myProject libcomm.so) # 显示指定链接动态库

# 再如：
target_link_libraries(myProject libcomm.so)　　#这些库名写法都可以。
target_link_libraries(myProject comm)
target_link_libraries(myProject -lcomm)
target_include_directories 设置编译时包含的目录
target_include_directories(<target> [SYSTEM] [BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])

Specifies include directories to use when compiling a 
given target. The named <target> must have been created by 
a command such as add_executable() or add_library() and 
must not be an ALIAS target.
```

### add_executable 为工程生成目标文件
语法：
```
add_executable(<name> [WIN32] [MACOSX_BUNDLE]
               [EXCLUDE_FROM_ALL]
               source1 [source2 ...])
```
简单的例子如下：
```
add_executable(demo main.cpp)
add_executable(echo_client echo_client.cc CreateUDiskRequest.pb.cc)
```

### file 文件操作命令
```
File manipulation command.
file(GLOB <variable>
     [LIST_DIRECTORIES true|false] [RELATIVE <path>] [CONFIGURE_DEPENDS]
     [<globbing-expressions>...])
file(GLOB_RECURSE <variable> [FOLLOW_SYMLINKS]
     [LIST_DIRECTORIES true|false] [RELATIVE <path>] [CONFIGURE_DEPENDS]
     [<globbing-expressions>...])

Generate a list of files that match the 
<globbing-expressions> and store it into the <variable>. 

Globbing expressions are similar to regular expressions, 
but much simpler. 

If RELATIVE flag is specified, the results will be 
returned as relative paths to the given path. 
The results will be ordered lexicographically.
```

### install 运行时规则指定
```
install(TARGETS <target>... [...])
install({FILES | PROGRAMS} <file>... [...])
install(DIRECTORY <dir>... [...])
install(SCRIPT <file> [...])
install(CODE <code> [...])
install(EXPORT <export-name> [...])

install(<FILES|PROGRAMS> files...
        TYPE <type> | DESTINATION <dir>
        [PERMISSIONS permissions...]
        [CONFIGURATIONS [Debug|Release|...]]
        [COMPONENT <component>]
        [RENAME <name>] [OPTIONAL] [EXCLUDE_FROM_ALL])
The FILES form specifies rules for installing files for a project. 
File names given as relative paths are interpreted with 
respect to the current source directory. 

Files installed by this form are by default given 
permissions OWNER_WRITE, OWNER_READ, GROUP_READ, and 
WORLD_READ if no PERMISSIONS argument is given.
```
这个命令只有在`make install`时才产生效果，会完成相应文件在`DESTINATION`上的安装。
