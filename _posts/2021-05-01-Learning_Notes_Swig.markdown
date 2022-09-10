---
title:  "学习笔记-Swig"
date:   2021-05-01 09:00:00 +0800
categories: [Language, Swig]
tags: [Learning-Note]
---



## 1.	Introduction
`SWIG (Simplified Wrapper and Interface Generator)`
```
SWIG is a software development tool that connects programs written in C and C++ with a variety of high-level programming languages. SWIG is used with different types of target languages including common scripting languages such as Javascript, Perl, PHP, Python, Tcl and Ruby.  
SWIG is most commonly used to create high-level interpreted or compiled programming environments, user interfaces, and as a tool for testing and prototyping C/C++ software. SWIG is typically used to parse C/C++ interfaces and generate the 'glue code' required for the above target languages to call into the C/C++ code. SWIG can also export its parse tree in the form of XML and Lisp s-expressions.   
SWIG is free software and the code that SWIG generates is compatible with both commercial and non-commercial projects.
It works by taking the declarations found in C/C++ header files and using them to generate the wrapper code that scripting languages need to access the underlying C/C++ code.
```

## 2.	Getting Start
let's say you have them in a file `example.c`
```
 /* File : example.c */
 #include <time.h>
 double My_variable = 3.0;
 int fact(int n) {
     if (n <= 1) return 1;
     else return n*fact(n-1);
 }
 int my_mod(int x, int y) {
     return (x%y);
 }	
 char *get_time()
 {
     time_t ltime;
     time(&ltime);
     return ctime(&ltime);
 }
```

- **Interface file**

```
/* example.i */
 %module example
 %{
 /* Put header files here or function declarations like below */
 extern double My_variable;
 extern int fact(int n);
 extern int my_mod(int x, int y);
 extern char *get_time();
 %}
 
 extern double My_variable;
 extern int fact(int n);
 extern int my_mod(int x, int y);
 extern char *get_time();
```

- **Building a Tcl module**

```
 swig -tcl example.i
 gcc -fpic -c example.c example_wrap.c \
     -I/usr/local/include 
 gcc -shared example.o example_wrap.o -o example.so
```

The swig command produces a file `example_wrap.c` that should be compiled and linked with the rest of the program.  
Everything in the `%{ ... %}` block is simply copied verbatim to the resulting wrapper file created by SWIG, 类似声明，包括头文件和声明。

On most machines, C++ extension modules should be linked using the C++ compiler. For example:

```
% swig -c++ -tcl example.i
% g++ -fPIC -c example.cxx
% g++ -fPIC -c example_wrap.cxx -I/usr/local/include
% g++ -shared example.o example_wrap.o -o example.so
```

How do I create shared libraries for Linux?   
- **For C**:
```
$ cc -fpic -c $(SRCS)
$ ld -shared $(OBJS) -o module.so
```

- **For C++**:
```
$ c++ -fpic -c $(SRCS)
$ c++ -shared $(OBJS) -o module.so
```

If you are using GCC 4.x under Ubuntu and using python 2.6 try the following   
```
$ swig -python module.i
$ gcc -fpic -I/usr/include/python2.6 -c module_wrap.c
$ gcc -shared module_wrap.o -o module.so
```

- **Noun interpretation**   
   - `.i`  --  SWIG interface file(a special .i or .swg suffix)
   - `.wrap.cpp` – wrapper function, write a special "wrapper" function that serves as the glue between the interpreter and the underlying C function

By default, an input file with the name `file.i` is transformed into a file `file_wrap.c` or `file_wrap.cxx` (depending on whether or not the `-c++` option has been used).


**Note**:   
(1) shared object file的名字要和SWIG中 `%module` 中指定的名字相符合，object file名字中可以加lib开头，不影响。如：`libtclxgphy.so`  

key point:
	文件名要和`#define SWIG_init    Tclbcmxgphy_Init`对应。  

(2) 如果module名字中带有number，需要注意采用如下方式：  
`load ./md5.so md5`  
(3) `–I`: 将所需头文件的地址指定，如果不指定，需要在`.cpp`文件中给出绝对地址
