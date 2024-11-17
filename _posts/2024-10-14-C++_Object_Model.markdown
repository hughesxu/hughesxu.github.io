---
title:  "C++对象模型实践探索"
date:   2024-10-14 09:00:00 +0800
categories: [Language, C++]
tags: [C++]
---

## 前言

C++对象模型是个常见、且复杂的话题，本文基于[Itanium C++ ABI](https://itanium-cxx-abi.github.io/cxx-abi/abi.html)通过程序实践介绍了几种 简单C++继承 场景下对象模型，尤其是存在虚函数的场景，并通过图的方式直观表达内存布局。  
本文展示的程序构建环境为Ubuntu，glibc 2.24，gcc 6.3.0。由于clang和gcc编译器都是基于Itanium C++ ABI（详细信息参考[gcc ABI policy](https://gcc.gnu.org/onlinedocs/libstdc++/manual/abi.html)），因此本文介绍的对象模型对clang编译的程序也基本适用。  

## 虚函数表简介  
### 虚函数表布局
含有虚函数的类，编译器会为其添加一个虚函数表（vptr）。  
用如下程序验证含有虚函数的类的内存布局，该程序很简单，只定义了构造函数，虚析构函数，和一个int成员变量。
```
// Derive.h
class Base_C
{
public:
    Base_C();
    virtual ~Base_C();

private:
    int baseC;
};

// Derive.cc
Base_C::Base_C()
{
}

Base_C::~Base_C()
{
}
```
gcc编译器可通过`-fdump-class-hierarchy`参数，查看类的内存布局。可得到如下信息：
```
// g++ -O0 -std=c++11 -fdump-class-hierarchy Derive.h
Vtable for Base_C
Base_C::_ZTV6Base_C: 4u entries
0     (int (*)(...))0
8     (int (*)(...))(& _ZTI6Base_C)
16    (int (*)(...))Base_C::~Base_C
24    (int (*)(...))Base_C::~Base_C

Class Base_C
   size=16 align=8
   base size=12 base align=8
Base_C (0x0x7fb8e9185660) 0
    vptr=((& Base_C::_ZTV6Base_C) + 16u)
```
从类`Base_C`的定义来看，类占用的空间包括一个**虚函数表指针vptr**和一个**整型变量**。由于内存对齐的原因，类占用16字节。  
接下来看虚函数表，表中一共有4个entry，每个entry都是函数指针，指向具体的虚函数，因此每个entry在测试的机器上编译占8字节（指针大小）。注意看到表中虚析构函数有两个，这实际上是Itanium C++ ABI规定的：  
```
The entries for virtual destructors are actually pairs of entries. 
The first destructor, called the complete object destructor, performs the destruction without calling delete() on the object. 
The second destructor, called the deleting destructor, calls delete() after destroying the object. 
Both destroy any virtual bases; a separate, non-virtual function, called the base object destructor, performs destruction of the object but not its virtual base subobjects, and does not call delete().
```
虚析构函数在虚函数表中占用两条目，分别是`complete object destructor`和`deleting destructor`。  
除了析构函数，虚函数表还有两个条目，紧靠析构函数的是`typeinfo`指针，指向类型信息对象（typeinfo object），用于运行时类型识别（RTTI)。第一个条目看起来可能比较陌生，是`offset`，该偏移存储了从当前虚表指针（vtable pointer）位置到对象顶部的位移。在ABI文档中这两个条目均有详细的介绍：
```
// typeinfo指针
The typeinfo pointer points to the typeinfo object used for RTTI. It is always present. 
All entries in each of the virtual tables for a given class must point to the same typeinfo object.
A correct implementation of typeinfo equality is to check pointer equality, except for pointers (directly or indirectly) to incomplete types. 
The typeinfo pointer is a valid pointer for polymorphic classes, i.e. those with virtual functions, and is zero for non-polymorphic classes.
```
```
The offset to top holds the displacement to the top of the object from the location within the object of the virtual table pointer that addresses this virtual table, as a ptrdiff_t. It is always present. 
The offset provides a way to find the top of the object from any base subobject with a virtual table pointer. This is necessary for dynamic_cast<void*> in particular. 
In a complete object virtual table, and therefore in all of its primary base virtual tables, the value of this offset will be zero. 
For the secondary virtual tables of other non-virtual bases, and of many virtual bases, it will be negative. Only in some construction virtual tables will some virtual base virtual tables have positive offsets, due to a different ordering of the virtual bases in the full object than in the subobject's standalone layout.
```
另外需要注意的是：`vptr=((& Base_C::_ZTV6Base_C) + 16u)`，虽然虚函数表中有四个条目，但是vptr的指针实际上并不是指向表的起始位置，而是指向第一个虚函数的位置。  
 
`Base_C`的内存布局如下图所示：  
![Base_C Object Layout]({{ "/assets/img/sample/C++_Model_BaseC_vtable_layout.png"| relative_url }})

### 虚函数表展示  
从上述类内存布局可以看出，虚函数表指针位于类内存空间的起始头部，类对象的起始地址，实际上也是虚函数表指针的地址。  
顺着这个思路，类对象起始地址，也即`this`指针，实际也是虚函数表指针的地址。  
对于虚函数表，可在代码中表示为：
```
typedef void* FUNC_PTR;     // 虚函数表中每个元素为函数指针，函数指针定义为FUNC_PTR
typedef FUNC_PTR* VTABLE_PTR; // 虚函数表指针
```
为了能够将虚函数表中每个虚函数符号展示出来，可以借助库函数[`dladdr`](https://man7.org/linux/man-pages/man3/dladdr.3.html)，其能通过地址获得对应的符号信息。该函数只能针对动态链接库，因此在测试时可将类的定义和实现代码编译成动态链接库，借助此函数，可通过如下函数打印出虚函数表的符号信息：  
```
void print_vtable(VTABLE_PTR start, size_t idx)
{
    void *func = start[idx];
    Dl_info func_info;
    dladdr(func, &func_info);
    printf("[idx:%zu] dli_sname = %s, dli_fname = %s, dli_saddr = %p, dli_fbase = %p\n", idx, func_info.dli_sname, func_info.dli_fname, func_info.dli_saddr, func_info.dli_fbase);
}
```
对于上述类`Base_C`，可以通过如下测试代码，打印出虚函数表的符号：  
```
Base_C *pBaseC = new Base_C();
VTABLE_PTR vtable = *reinterpret_cast<const VTABLE_PTR*>(pBaseC);
for (size_t i = 0; i < 2; i++)  // 虚函数表指针指向的位置起始，虚函数表中一共只有2项
{
    print_vtable(vtable, i);
}
```
测试结果如下，看到能正确打印出两个虚析构函数的符号：  
```
[idx:0] dli_sname = _ZN6Base_CD1Ev
[idx:1] dli_sname = _ZN6Base_CD0Ev
```


## 继承下的C++对象模型  
### 单继承下C++对象模型  
首先，看一个单继承场景的例子：  
```
// 此处省略类的实现部分
class Base_C
{
public:
    Base_C();
    virtual ~Base_C();

private:
    int baseC;
};

class Base_D : public Base_C
{
public:
    Base_D(int i);
    virtual ~Base_D();
    virtual void add(void) { cout << "Base_D::add()..." << endl; }
    virtual void print(void);

private:
    int baseD;
};

class Derive_single : public Base_D
{
public:
    Derive_single(int d);
    void print(void) override;
    virtual void Derive_single_print();

private:
    int Derive_singleValue;
};
```
单继承场景下，派生类有且只有一个虚表（将基类的虚表复制），同时派生类中override的虚函数，会在虚函数表中对原函数进行覆盖，派生类新增的虚函数也将追加到虚函数表的尾部。  
从整体内存布局上来看，派生类中新增的非静态成员变量，也会追加到基类的成员变量之后。  
打印类内存布局如下：  
```
Vtable for Derive_single
Derive_single::_ZTV13Derive_single: 7u entries
0     (int (*)(...))0
8     (int (*)(...))(& _ZTI13Derive_single)
16    (int (*)(...))Derive_single::~Derive_single
24    (int (*)(...))Derive_single::~Derive_single
32    (int (*)(...))Base_D::add
40    (int (*)(...))Derive_single::print
48    (int (*)(...))Derive_single::Derive_single_print

Class Derive_single
   size=24 align=8
   base size=20 base align=8
Derive_single (0x0x7fb8e93fe8f0) 0
    vptr=((& Derive_single::_ZTV13Derive_single) + 16u)
  Base_D (0x0x7fb8e93fe958) 0
      primary-for Derive_single (0x0x7fb8e93fe8f0)
    Base_C (0x0x7fb8e91857e0) 0
        primary-for Base_D (0x0x7fb8e93fe958)
```
内存布局如下图所示，内存布局和上述描述一致：  
![Single Derive Object Layout]({{ "/assets/img/sample/C++_Model_Single_Derive_vtable.png"| relative_url }})



### 多继承下C++对象模型（非菱形） 
接下来考虑非菱形多继承场景，此时对于派生类，会将其每个基类的虚函数表“拷贝”一份，最终组成虚函数表组，虚函数表排列顺序，由基类在类定义中的声明顺序决定。  
派生类的虚函数被放在声明的第一个基类的虚函数表中，派生类对基类函数override时，会覆盖所有基类中对应的函数。  

```
// 此处省略类的实现部分
class Base_A
{
public:
    Base_A(int i);
    virtual ~Base_A();
    int getValue();
    static void countA();
    virtual void print(void);

private:
    int baseA;
    static int baseAS;
};

class Base_B
{
public:
    Base_B(int i);
    virtual ~Base_B();
    int getValue();
    virtual void add(void);

    static void countB();
    virtual void print(void);

private:
    int baseB;
    static int baseBS;
};

class Derive_multiBase : public Base_A, public Base_B
{
public:
    Derive_multiBase(int d);
    void add(void) override;
    void print(void) override;
    virtual void Derive_multiBase_print();

private:
    int Derive_multiBaseValue;
};
```
打印类内存布局如下：  
```
Vtable for Derive_multiBase
Derive_multiBase::_ZTV16Derive_multiBase: 13u entries
0     (int (*)(...))0
8     (int (*)(...))(& _ZTI16Derive_multiBase)
16    (int (*)(...))Derive_multiBase::~Derive_multiBase
24    (int (*)(...))Derive_multiBase::~Derive_multiBase
32    (int (*)(...))Derive_multiBase::print
40    (int (*)(...))Derive_multiBase::add
48    (int (*)(...))Derive_multiBase::Derive_multiBase_print
56    (int (*)(...))-16
64    (int (*)(...))(& _ZTI16Derive_multiBase)
72    (int (*)(...))Derive_multiBase::_ZThn16_N16Derive_multiBaseD1Ev
80    (int (*)(...))Derive_multiBase::_ZThn16_N16Derive_multiBaseD0Ev
88    (int (*)(...))Derive_multiBase::_ZThn16_N16Derive_multiBase3addEv
96    (int (*)(...))Derive_multiBase::_ZThn16_N16Derive_multiBase5printEv

Class Derive_multiBase
   size=32 align=8
   base size=32 base align=8
Derive_multiBase (0x0x7fb8e910cd20) 0
    vptr=((& Derive_multiBase::_ZTV16Derive_multiBase) + 16u)
  Base_A (0x0x7fb8e91855a0) 0
      primary-for Derive_multiBase (0x0x7fb8e910cd20)
  Base_B (0x0x7fb8e9185600) 16
      vptr=((& Derive_multiBase::_ZTV16Derive_multiBase) + 72u)
```
从内存布局中可看到存在两个vptr（分别指向两个虚函数表），对应`Derive_multiBase`从两个基类`Base_A`、`Base_B`拷贝得到的虚函数表。  
派生类`Derive_multiBase`中所有虚函数都拓展在主虚函数表（primary virtual table），也即从`Base_A`拷贝得到的虚函数表。  
从`Base_B`拷贝得到的虚函数表也称为辅助虚函数表（secondary virtual tables），从内存布局中看到其`offset`为***-16***，因为此虚函数表指针距对象内存的初始位置16个字节。同时注意到此虚函数表中虚函数符号为`non-virtual thunk to...`，这个和函数跳转的机制有关，通过thunk对调用不同父类的函数的地址进行修正，可以参考[深入探索 C++多态②-继承关系](https://wenfh2020.com/2023/08/22/cpp-inheritance/)、[C++对象模型](https://miaoerduo.com/2023/01/19/cpp-object-model/)中的介绍。  
```
thunk
A segment of code associated (in this ABI) with a target function, which is called instead of the target function for the purpose of modifying parameters (e.g. this) or other parts of the environment before transferring control to the target function, and possibly making further modifications after its return. 
A thunk may contain as little as an instruction to be executed prior to falling through to an immediately following target function, or it may be a full function with its own stack frame that does a full call to the target function.
```
内存布局如下图所示：  
![Multi Derive Object Layout]({{ "/assets/img/sample/C++_Model_Multi_Derive_vtable.png"| relative_url }})


### 讨论：enable_shared_from_this特性如何影响内存布局  
[enable_shared_from_this](https://en.cppreference.com/w/cpp/memory/enable_shared_from_this)文档中有如下描述：  
```
A common implementation for enable_shared_from_this is to hold a weak reference (such as std::weak_ptr) to *this. 
For the purpose of exposition, the weak reference is called weak-this and considered as a mutable std::weak_ptr member.
```
enable_shared_from_this的通常实现是让实例拥有一个“弱引用”，可表现为实例有个std::weak_ptr的成员变量。  
可在单继承场景的测试代码上进行验证，对`Derive_single`类增加继承自`std::enable_shared_from_this<Derive_single>`，其他不变：  
```
class Derive_single : public Base_D, public std::enable_shared_from_this<Derive_single>
{
public:
    Derive_single(int d);
    void print(void) override;
    virtual void Derive_single_print();

private:
    int Derive_singleValue;
};
```
首先打印类内存布局如下：  
```
Vtable for Derive_single
Derive_single::_ZTV13Derive_single: 7u entries
0     (int (*)(...))0
8     (int (*)(...))(& _ZTI13Derive_single)
16    (int (*)(...))Derive_single::~Derive_single
24    (int (*)(...))Derive_single::~Derive_single
32    (int (*)(...))Base_D::add
40    (int (*)(...))Derive_single::print
48    (int (*)(...))Derive_single::Derive_single_print

Class Derive_single
   size=40 align=8
   base size=36 base align=8
Derive_single (0x0x7fd5c76431c0) 0
    vptr=((& Derive_single::_ZTV13Derive_single) + 16u)
  Base_D (0x0x7fd5c7639750) 0
      primary-for Derive_single (0x0x7fd5c76431c0)
    Base_C (0x0x7fd5c7632780) 0
        primary-for Base_D (0x0x7fd5c7639750)
  std::enable_shared_from_this<Derive_single> (0x0x7fd5c76327e0) 16
```
对比前文，可发现`Derive_single`内存占用由24字节增大到40字节，原因是`std::enable_shared_from_this<Derive_single>`的继承多占用了16字节。从[std::weak_ptr](https://en.cppreference.com/w/cpp/memory/weak_ptr)的文档中可知`std::weak_ptr`的典型实现实际上是存储了两个指针，和这里的16字节内存增长一致。  
```
Like std::shared_ptr, a typical implementation of weak_ptr stores two pointers:
-- a pointer to the control block; and
-- the stored pointer of the shared_ptr it was constructed from.
```
另外，特别注意此时`Derive_single`类虚函数表和前文没有差异，因此enable_shared_from_this特性不影响虚函数表的内容。  



## 参考资料
[Itanium C++ ABI](https://itanium-cxx-abi.github.io/cxx-abi/abi.html)  
[图说C++对象模型：对象内存布局详解](https://www.cnblogs.com/QG-whz/p/4909359.html)  
[C++对象模型](https://miaoerduo.com/2023/01/19/cpp-object-model/)  
[C++深入探索C++多态②-继承关系](https://wenfh2020.com/2023/08/22/cpp-inheritance/)  
[虚函数继承-thunk技术初探](https://juejin.cn/post/7058433021094920206)  
[C++：虚函数内存布局解析（以clang编译器为例）](https://www.less-bug.com/posts/cpp-vtable-and-object-memory-layout-clang-example/)