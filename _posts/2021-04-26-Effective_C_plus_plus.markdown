---
title:  "Effective C++之55条建议"
date:   2021-04-26 09:00:00 +0800
categories: [Language, C++]
tags: [C++]
---


本文为Effective C++一书的学习总结，对55条程序设计建议条款进行罗列、简记。

## 1  让自己习惯 C++  


#### No.01  视C++为一个语言联邦  
+ C++实则是由多个次语言（Sublanguage）组成的联邦而非单一语言，主要的次语言包括 C、Object-Oriented C++、Template C++、STL。


#### No.02  尽量以 const, enum, inline 替代#define  
+ 宁可以编译器替换预处理器
+ #define 记号可能无法进入符号表(Symbol table)，无法创建一个class专属常量。  
+ 对于单纯变量，最好以const对象或enums替换#defines。  
+ 对于形似函数的宏 (macros)，最好改用inline函数替换#defines。  


#### No.03  尽可能使用 const  
+ 将某些东西声明为 const 可帮助编译器侦测出错误用法。const 可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体。  
+ 当 const 和 non-const成员函数有着实质等价的实现时，令 non-const 版本调用 const 版本可避免代码重复。  
+ bitwise-constness 和 logical-constness。  


#### No.04  确定对象被使用前已被初始化  
+ 构造函数最好使用成员初始化列表 (member initialization list)，最好不要在构造函数本体使用赋值操作 (assignment)。初值列列出的成员变量，其排列次序应该和它们在class中声明的次序相同。  
+ 为免除“跨编译单元之初始化次序问题”，请以 local static 对象替换 non-local static 对象。  



## 2  构造/析构/赋值运算  


#### No.05  了解C++默默编写并调用哪些函数  
+ 编译器可以暗自为class创建 default构造函数、copy构造函数、copy assignment操作符、以及析构函数。  
+ 如果没有声明任何构造函数，编译器会为你声明一个default构造函数。  


#### No.06  若不想使用编译器自动生成的函数，就该明确拒绝  
+ 为驳回编译器自动（暗自）提供的机能，可将相应的成员函数声明为 private 并且不予实现。使用像Uncopyable这样的base class也是一种做法。  


#### No.07  为多态基类声明 virtual 析构函数  
+ polymorphic（带多态性质的）base classes应该声明一个 virtual 析构函数。如果 class 带有任何 virtual 函数，它就应该拥有一个 virtual 析构函数。  
+ class 的设计目的如果不是作为 base class 使用，或不是为了具备多态性（polymorphically），就不该声明 virtual 析构函数。  


#### No.08  别让异常逃离析构函数  
+ 析构函数中出现的异常，往往会传播并导致不确定行为，而不确定行为比较可怕。  
+ 析构函数绝对不要吐出异常。如果一个析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们（不传播）或结束程序。  


#### No.09  绝不在构造和析构过程中调用 virtual 函数  
+ 在构造和析构期间不要调用 virtual 函数，因为这类调用从不下降至 derived class。  


#### No.10  令 operator= 返回一个 reference to *this  


#### No.11  在 operator= 中处理“自我赋值”  


#### No.12  复制对象时勿忘其每一个成分  
+ Copying函数（copy构造函数和copy assignment操作符）应该确保复制“对象内的所有成员变量”及“所有 base class 成分”。  
不要尝试以某个copying函数实现另一个copying函数。应该将共同机能放进第三个函数中，并由两个copying函数共同调用。  


## 3  资源管理  


#### No.13  以对象管理资源  
+ Resource Acquisition Is Initialization; RAII  


#### No.14  在资源管理类中小心copying行为  
+ 复制 RAII 对象必须一并复制它所管理的资源，所以资源的copying行为决定RAII对象的copying。  
+ 普遍而常见的 RAII class copying行为是：抑制copying、施行引用计数法（reference counting）。  


#### No.15  在资源管理类中提供对原始资源的访问  
+ APIs往往要求访问原始资源（raw resources），所以每一个RAII class应该提供一个“取得其管理之资源”的方法。  
+ 对原始资源的访问可能经由显示转换或隐式转换。一般而言显示转换比较安全，但隐式转换对客户比较方便。  
+ 显示转换指get()函数，隐式转换比如隐式的类型转换运算符。  


#### No.16  成对使用 new 和 delete 时要采取相同形式  
+ 如果你在 new 表达式中使用 []，必须在相应的 delete 表达式中也使用 []。 
+ 如果你在 new 表达式中不使用 []，一定不要在相应的 delete 表达式中使用 []。  
+ 最好尽量不要对数组形式做 typedefs 动作。  


#### No.17  以独立语句将 newed 对象置入智能指针  
+ 以独立语句将 newed 对象存储于（置入）智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄漏。  


## 4  设计与声明  


#### No.18  让接口容易被正确使用，不易被误用  
+ 好的接口很容易被正确使用，不容易被误用。  
+ “促进正确使用”的办法包括接口的一致性，以及内置类型的行为兼容。  
+ “阻止误用”的办法包括建立新类型（引入新类型）、限制类型上的操作，束缚对象值（接口一致性），以及消除客户的资源管理责任。  
+ shared_ptr 支持定制型删除器 (custom deleter)。这可防范 DLL（动态链接程序库）问题，可被用来自动解除互斥锁等等。  


#### No.19  设计 class 犹如设计 type  
+ class 的设计就是 type的设计。  


#### No.20  宁以 pass-by-reference-to-const 替换 pass-by-value  
+ 尽量以 pass-by-reference-to-const 替换 pass-by-value。前者通常比较高效，并可避免切割问题（Slicing problem）（基类和派生类）。  
+ 以上规则并不适用于内置类型，以及 STL 的迭代器和函数对象。对它们而言，pass-by-value往往比较适当。  


#### No.21  必须返回对象时，别妄想返回其reference  
+ 绝不要返回 pointer 或 reference 指向一个 local stack 对象，或返回 reference 指向一个 heap-allocated 对象，或返回 pointer或 reference 指向一个 local static 对象而有可能同时需要多个这样的对象。  


#### No.22  将成员变量声明为private  
+ 切记将成员变量声明为 private。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供 class 作者以充分的实现弹性。  
+ protected 并不比 public 更具封装性。  


#### No.23  宁以 non-member、non-friend替换member函数  
+ 宁可拿 non-member non-friend函数替换member函数。这样做可以增加封装性、包裹弹性（packaging flexibility）和机能扩充性。  


#### No.24   若所有参数皆需类型转换，请为此采用non-member函数  
+ 如果需要为某个函数的所有参数（包括被this指针所指的那个隐喻参数）进行类型转换，那么这个函数必须是个non-member。  


#### No.25   考虑写出一个不抛异常的swap函数
+ 当std::swap对你的类型效率不高时，提供一个swap成员函数，并确定这个函数不抛出异常。  
+ 如果你提供一个member swap，也该提供一个non-member swap用来调用前者。对于classes（而非templates），也请特化std::swap。  
+ 调用swap时应针对std::swap使用using声明式，然后调用swap并且不带任何“命名空间资格修饰”。  
+ 为“用户定义类型”进行std templates全特化是好的，但千万不要尝试在std内加入某些对std而言全新的东西。  


## 5  实现  


#### No.26   尽可能延后变量定义式的出现时间  
+ 尽可能延后变量定义式的出现，直到能够给它初值实参为止。  


#### No.27   尽量少做转型动作  
+ 如果可以，尽量避免转型，特别是在注重效率的代码中避免 dynamic_cast。如果这个设计需要转型动作，试着发展无需转型的替代设计。（vitual函数继承体系）。  
+ 如果转型是必要的，试着将它隐藏在某个函数背后。客户随后可以调用该函数，而不需将转型放进他们自己的代码内。  
+ 宁可使用C++ style（新式）转型，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分门别类的职掌。  


#### No.28   避免返回handles指向对象内部成分  
+ 避免返回handles（包括reference、指针、迭代器）指向对象内部。遵守这个条款可增加封装性，帮助const成员函数的行为像个const，并将发生“虚调号码牌” (dangling handles)的可能性降至最低。  
+ handles（号码牌，用来取得某个对象）：References、指针和迭代器。  
+ 内部成分包括：成员变量和不被公开的成员函数。  


#### No.29   为“异常安全”而努力是值得的  
+ 异常安全函数（Exception-safe functions）即使发生异常也不会 泄漏资源 或 允许任何数据结构破坏。这样的函数分为三种可能的保证：基本承诺、强烈保证、不抛掷保证。  
+ “强烈保证”往往能够以copy-and-swap实现出来，但“强烈保证”并非对所有的函数都可实现或具备现实意义。  
+ 函数提供的“异常安全保证”通常最高只等于其所调用之各个函数的“异常安全保证”中的最弱者。  


#### No.30   透彻了解inlining的里里外外  
+ inline只是对编译器的一个申请，不是强制命令。  
+ 将大多数inlining限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级（Binary upgradability）更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化。  
+ inline函数通常一定被置于头文件内。  
+ 对vitual函数的调用也都会使inlining落空 （virtual是运行时决定调用）。  
+ 构造函数和析构函数往往是inlining的糟糕候选人（可能引起在整个继承链路上的代码插入inline）。  
+ inline函数无法随着程序库的升级而升级，需要重新编译。  
+ 不要只因为function templates出现在头文件，就将他们声明为inline。  


#### No.31   将文件间的编译依存关系降至最低  
+ 支持“编译依存性最小化”的一般构想是：相依于声明式，不要相依于定义式。基于此构想的两个手段是 Handle classes 和 Interface classes。  
Handle classes: 使用pimpl idiom (pointer to implementation)。  
Interface classes: vitual, factory （工厂）函数。  
+ 程序库头文件应该以“完全且仅有声明式”（full and declaration-only forms）的形式存在。这种做法不论是否涉及templates都适用。  
+ 接口与实现分离：以 声明的依存性 替换 定义的依存性。  
+ 如果使用 object references 或 object pointers 可以完成任务，就不要使用objects。  
+ 如果能够，尽量以 class声明式 替换 class 定义式。  
+ 为声明式和定义式提供不同的头文件。  


## 6  继承与面向对象设计  


#### No.32   确定你的 public 继承塑模出 is-a 关系  
+ “public继承”意味is-a。适用于base classes身上的每一件事情一定也适用于derived classes身上，因为每一个derived class对象也都有一个base class对象。  


#### No.33   避免遮掩继承而来的名称  
+ derived classes内的名称会遮掩base classes内的名称。在public继承下从来没有人希望如此。  
+ 为了让被遮掩的名称再见天日，可使用 using 声明式或转交函数（forwarding functions）。  
base class定义的virtual函数，如果不想在derived class中隐藏，derived class中要对每个函数都进行声明。  
using声明式会令继承而来的某给定名称之所有同名函数在 derived class中都可见。  


#### No.34   区分接口继承和实现继承  
+ public继承由两部分组成：函数接口（function interfaces）继承 和 函数实现（function implementation）继承。  
在public继承之下，derived class总是继承base class的接口。  
+ pure virtual 只继承接口。  
+ impure virtual(简朴的非纯) 让derived class继承接口和缺省实现。  
+ non-virtual 为了令 derived class继承函数的接口及一份强制性实现。（不变性凌驾其特异性）  


#### No.35   考虑virtual函数以外的其他选择  
+ virtual函数的替代方案包括NVI手法（non-virtual interface）及Strategy设计模式的多种形式。NVI手法自身是一个特殊的Template Method设计模式。  
+ 将机能从成员函数移到class外部函数，带来的一个缺点是，非成员函数无法访问class的non-public成员。  
+ tr1::function对象的行为就像一般函数指针。这样的对象可接纳“与给定之目标签名式兼容”的所有可调用物。  


### 参考资料
Effective C++ : 改善程序与设计的55个具体做法     