---
title:  "学习笔记-Python"
date:   2021-05-01 09:00:00 +0800
categories: [Language, Python]
tags: [Learning-Note]
---


## Python 脚本初窥   
```
#!/usr/bin/python		
// 告诉操作系统执行这个脚本的时候，调用 /usr/bin 下的 python 解释器
print "Hello, Python!";
#!/usr/bin/env python
// 这种用法是为了防止操作系统用户没有将 python 装在默认的 /usr/bin 路径 
// 里。当系统看到这一行的时候，首先会到 env 设置里查找 python 的安装路径，
// 再调用对应路径下的解释器程序完成操作。
```


## 1 Python基本语法
### 2.1 标识符
所有标识符可以包括英文、数字以及下划线(`_`)，但不能以数字开头。区分大小写。
以单下划线开头 `_foo` 的代表不能直接访问的类属性，需通过类提供的接口进行访问，不能用 `from xxx import *` 而导入；  
以双下划线开头的 `__foo` 代表类的私有成员；以双下划线开头和结尾的 `__foo__` 代表 Python 里特殊方法专用的标识，如 `__init__()` 代表类的构造函数。

### 2.2 保留字（关键字）
```
and         exec        not
assert      finally     or
break       for         pass
class       from        print
continue    global      raise
def         if          return
del         import      try
elif        in          while
else        is          with
except      lambda      yield
```

### 2.3 引号
Python 可以使用引号( `'` )、双引号( `"` )、三引号( `'''` 或 `"""` ) 来表示字符串，引号的开始与结束必须的相同类型的。  
其中**三引号可以由多行组成**，编写多行文本的快捷语法，常用于文档字符串，在文件的特定地点，被当做注释。

### 2.4 输入和输出
+ `raw_input()`: 等待用户输入，按回车键后就会退出  
+ `print`: 默认输出是换行的，如果要实现不换行需要在变量末尾加上逗号 `,`
 
## 2 Python变量类型
+ Python 中的变量赋值不需要类型声明。
+ 每个变量在内存中创建，都包括变量的标识，名称和数据这些信息。
+ 每个变量在使用前都必须赋值，变量赋值以后该变量才会被创建。
+ 等号（`=`）用来给变量赋值。
+ 等号（`=`）运算符左边是一个变量名，等号（`=`）运算符右边是存储在变量中的值。

Python有五个标准的数据类型：
+ **Numbers（数字）**
+ **String（字符串）**
+ **List（列表）**
+ **Tuple（元组）**
+ **Dictionary（字典）**


### 3.1 数字
#### 3.1.1 基本定义
数字用于存储数值，是不可改变的数据类型，改变数字数据类型会分配一个新的对象。  
当指定一个值时，Number对象就会被创建：
```
var1 = 1
var2 = 10
```
可以使用del语句删除单个或多个对象的引用。del语句的语法是：
```
del  var1[,var2[,var3[....,varN]]]]	
del  var_a, var_b
```
**del也可以用来删除其他类型的对象，如列表、元组、字典等**

Python支持四种不同的数字类型：  
+ `int（有符号整型）`  
`eg  : -0x260`

+ `long（长整型[也可以代表八进制和十六进制]）`：长整型用L/l结尾
`eg : -4721885298529L`
+ `float（浮点型）`
+ `complex（复数）`：实数部分和复数部分，可以用`a + bj`，或者 `complex(a,b)` 表示， 复数的实部 a 和虚部 b 都是浮点型。
`eg : 4.53e - 7j`

#### 3.1.2 math/ cmath 模块
+ Python math 模块提供了许多对浮点数的数学运算函数。
+ Python cmath 模块包含了一些用于复数运算的函数。
```
import math
dir(math)
['__doc__', '__file__', '__loader__', '__name__', '__package__', 
'__spec__', 'acos', 'acosh', 'asin', 'asinh', 'atan', 'atan2', 
'atanh', 'ceil', 'copysign', 'cos', 'cosh', 'degrees', 'e', 'erf', 
'erfc', 'exp', 'expm1', 'fabs', 'factorial', 'floor', 'fmod', 
'frexp', 'fsum', 'gamma', 'gcd', 'hypot', 'inf', 'isclose', 
'isfinite', 'isinf', 'isnan', 'ldexp', 'lgamma', 'log', 'log10', 
'log1p', 'log2', 'modf', 'nan', 'pi', 'pow', 'radians', 'sin', 
'sinh', 'sqrt', 'tan', 'tanh', 'tau', 'trunc']
```

```
import cmath  
dir(cmath)  
['__doc__', '__file__', '__loader__', '__name__', '__package__', 
'__spec__', 'acos', 'acosh', 'asin', 'asinh', 'atan', 'atanh', 'cos', 
'cosh', 'e', 'exp', 'inf', 'infj', 'isclose', 'isfinite', 'isinf', 
'isnan', 'log', 'log10', 'nan', 'nanj', 'phase', 'pi', 'polar', 
'rect', 'sin', 'sinh', 'sqrt', 'tan', 'tanh', 'tau']
```


#### 3.1.3 数学函数
```
import math
```
+ `abs()` 是一个内置函数，而`fabs()`在`math`模块中定义的。
+ `fabs()`函数 只适用于`float`和`integer`类型，而 `abs()` 也适用于复数。

函数 |描述|所需模块
:-----:|:-------|:------
`abs(x)`|返回数字的绝对值，如abs(-10)返回10|内置
`ceil(x)`|返回数字的上入整数，如math.ceil(4.1) 返回5	|math
`cmp(x, y)`|如果 x < y 返回-1，如果 x == y 返回0，如果 x > y 返回 1|内置
`exp(x)`|返回e的x次幂(ex)，如math.exp(1) 返回2.718281828459045|math
`fabs(x)`|返回数字的绝对值，如math.fabs(-10) 返回10.0|math
`floor(x)`|返回数字的下舍整数，如math.floor(4.9)返回4|math
`log(x)`|如math.log(math.e)返回1.0，math.log(100,10)返回2.0|math
`log10(x)`|返回以10为基数的x的对数，如math.log10(100)返回2.0|math
`max(x1, x2,...)`|返回给定参数的最大值，参数可以为序列|内置
`min(x1, x2,...)`|返回给定参数的最小值，参数可以为序列|内置
`modf(x)`|返回x的整数部分与小数部分，两部分的数值符号与x相同，整数部分以浮点型表示|math
`pow(x, y)`|x**y 运算后的值|math
`round(x [,n])`|返回浮点数x的四舍五入值，如给出n值，则代表舍入到小数点后的位数|内置
`sqrt(x)`|返回数字x的平方根|math


#### 3.1.4 随机数函数
```
import random
```

函数|描述|所需模块
:----:|:-------|:------
`choice(seq)`|从序列的元素中随机挑选一个元素，比如random.choice(range(10))，从0到9中随机挑选一个整数|random
`randrange ([start,] stop [,step])`|从指定范围内，按指定基数递增的集合中获取一个随机数，基数缺省值为1	|random
`random()`|随机生成下一个实数，它在[0,1)范围内|random
`seed([x])`|改变随机数生成器的种子seed。如果你不了解其原理，你不必特别去设定seed，Python会帮你选择seed。	|random
`shuffle(list)`|将序列的所有元素随机排序|random
`uniform(x, y)`|随机生成下一个实数，它在[x,y]范围内|random


#### 3.1.5 三角函数

函数|描述|所需模块
:----:|:-------|:------
`acos(x)`|返回x的反余弦弧度值
`asin(x)`|返回x的反正弦弧度值
`atan(x)`|返回x的反正切弧度值
`atan2(y, x)`|返回给定的 X 及 Y 坐标值的反正切值|math
`cos(x)`|返回x的弧度的余弦值
`hypot(x, y)`|返回欧几里德范数 sqrt(x*x + y*y)。	
`sin(x)`|返回的x弧度的正弦值。	
`tan(x)`|返回x弧度的正切值。	
`degrees(x)`|将弧度转换为角度,如degrees(math.pi/2) ，返回90.0	|math
`radians(x)`|将角度转换为弧度	|math

 
### 3.2 字符串
#### 3.2.1 基本定义
字符串或串(String)是由**数字**、**字母**、**下划线**组成的一串字符。  
一般记为：`s="a1a2···an"(n>=0)`，它是编程语言中表示文本的数据类型。  
python的字串列表有2种取值顺序:  
+ 从左到右索引默认0开始的，最大范围是字符串长度少1
+ 从右到左索引默认-1开始的，最大范围是字符串开头

从字符串中获取一段子字符串，可以使用**变量 [头下标 : 尾下标]**，就可以截取相应的字符串。
+ 下标是从 0 开始算起，可以是正数或负数，下标可以为空表示取到头或尾。
+ 取到的数据不包含尾下标。
+ 加号（+）是字符串连接运算符，星号（*）是重复操作。
+ 当使用以冒号分隔的字符串，python返回一个新的对象，结果包含了以这对偏移标识的连续的内容，左边的开始是包含了下边界。  
比如:
`s = 'ilovepython'		// s[1:5]的结果是love。`

#### 3.2.2 转义字符

转义字符|描述
:----:|:-------
\ (在行尾时)	|续行符
\\	|反斜杠符号
\’	|单引号
\’’	|双引号
\a	|响铃
\b	|退格（backspace）
\e	|转义
\000|空
\n	|换行
\v	|纵向制表符
\t	|横向制表符
\r	|回车
\f	|换页
\oyy	|八进制数，yy代表的字符，例如：\o12代表换行
\xyy	|十六进制数，yy代表的字符，例如：\x0a代表换行
\other	|其它的字符以普通格式输出


#### 3.2.3 字符串运算符
假设`a = “hello”`, `b = “Python”`  

操作符|描述|实例
:----:|:---|:---
`+`     |字符串连接
`*`	    |重复输出字符串
`[]`    |通过索引获取字符串中字符	
`[:]`   |截取字符串中的一部分	
`in`    |成员运算符 - 如果字符串中包含给定的字符返回 True	
`not in`|成员运算符 - 如果字符串中不包含给定的字符返回 True	
`r/R原始字符串`|**原始字符串**：所有的字符串都是直接按照字面的意思来使用，没有转义特殊或不能打印的字符。 原始字符串除在字符串的第一个引号前加上字母"r"（可以大小写）以外，与普通字符串有着几乎完全相同的语法
`%`     |格式字符串	

#### 3.2.4 字符串格式化
```
#!/usr/bin/python
print "My name is %s and weight is %d kg!" % ('Zara', 21)
```
##### (1) python字符串格式化符号  

符号|描述
:----:|:-------
%c	|格式化字符及其ASCII码
%s	|格式化字符串
%d	|格式化整数
%u	|格式化无符号整型
%o	|格式化无符号八进制数
%x	|格式化无符号十六进制数
%X	|格式化无符号十六进制数（大写）
%f	|格式化浮点数字，可指定小数点后的精度
%e	|用科学计数法格式化浮点数
%E	|作用同%e，用科学计数法格式化浮点数
%g	|%f和%e的简写
%G	|%f 和 %E 的简写
%p	|用十六进制数格式化变量的地址

##### (2) 格式化操作符辅助指令

符号|功能
:----:|:-------
`*`	    |定义宽度或者小数点精度
`-`	    |用做左对齐
`+`	    |在正数前面显示加号( + )
`<sp>`	|在正数前面显示空格
`#`	    |在八进制数前面显示零(`'0'`)，在十六进制前面显示`'0x'`或者`'0X'`(取决于用的是`'x'`还是`'X'`)
`0`	    |显示的数字前面填充`'0'`而不是默认的空格
`%`	    |`'%%'`输出一个单一的`'%'`
`(var)`	|映射变量(字典参数)
`m.n.`	|m 是显示的最小总宽度,n 是小数点后的位数(如果可用的话)

##### (3) format 格式化函数
基本语法是通过 `{}` 和 `:` 来代替以前的 `%` 。
```
>>>"{} {}".format("hello", "world")    	# 不设置指定位置，按默认顺序
'hello world'
>>> "{0} {1}".format("hello", "world")  # 设置指定位置
'hello world'
>>> "{1} {0} {1}".format("hello", "world")  # 设置指定位置
'world hello world'

print("网站名：{name}, 地址 {url}".format(name="菜鸟教程", url="www.runoob.com"))

# 必须是”=”连接，和字典区分开
# 通过字典设置参数
site = {"name": "菜鸟教程", "url": "www.runoob.com"}
print("网站名：{name}, 地址 {url}".format(**site))		# **是必须要加的
# 通过列表索引设置参数
my_list = ['菜鸟教程', 'www.runoob.com']
print("网站名：{0[0]}, 地址 {0[1]}".format(my_list)) 	 # "0" 是必须的，指定第一个列表
```

+ 数字格式化：
可以使用大括号 `{ }` 来转义大括号
```
#!/usr/bin/python
print ("{} 对应的位置是 {{0}}".format("runoob"))
```
  
数字|格式|输出|描述
:----:|:-------|:------|:-----
3.1415926	|`{:.2f}`	|3.14	|保留小数点后两位
3.1415926	|`{:+.2f}`	|+3.14	|带符号保留小数点后两位
-1          |`{:+.2f}`	|-1.00	|带符号保留小数点后两位
2.71828	    |`{:.0f}`	|3	    |不带小数（四舍五入）
5	        |`{:0>2d}`	|05	    |数字补零 (填充左边, 宽度为2)
5	        |`{:x<4d}`	|5xxx	|数字补x (填充右边, 宽度为4)
10	        |`{:x<4d}`	|10xx	|数字补x (填充右边, 宽度为4)
1000000	    |`{:,}`	    |1,000,000	|以逗号分隔的数字格式
0.25	    |`{:.2%}`	|25.00%		|百分比格式
100000000	|`{:.2e}`	|1.00e+8	|指数记法
13	        |`{:10d}`	|13	    |右对齐 (默认, 宽度为10)
13	        |`{:<10d}`	|13	    |左对齐 (宽度为10)
13	        |`{:^10d}`	|13	    |中间对齐 (宽度为10)


#### 3.2.5 Unicode字符串
```
>>> u'Hello World !'
u'Hello World !'
```
引号前小写的`"u"`表示这里创建的是一个 Unicode 字符串。

#### 3.2.6 字符串内建函数

方法|描述
:----:|:-------
`string.count(str, beg=0, end=len(string))`|返回 str 在 string 里面出现的次数，若beg 或 end 指定则返回指定范围内 str 出现的次数
`string.startswith(obj, beg=0, end=len(string))`|检查字符串是否以 obj 结束，如果beg 或者 end 指定则检查指定的范围内是否以 obj 开始，如果是，返回 True,否则返回 False.
`string.endswith(obj, beg=0, end=len(string))`|检查字符串是否以 obj 结束，如果beg 或者 end 指定则检查指定的范围内是否以 obj 结束，如果是，返回 True,否则返回 False.
`string.find(str, beg=0, end=len(string))`|检测 str 是否包含在 string 中，如果 beg 和 end 指定范围，则检查是否包含在指定范围内，如果是返回开始的索引值，否则返回-1
`string.format()`	|格式化字符串
`string.index(str, beg=0, end=len(string))`|跟find()方法一样，只不过如果str不在 string中会报一个异常.
`string.join(seq)`|以 string 作为分隔符，将 seq 中所有的元素(的字符串表示)合并为一个新的字符串
`string.partition(str)`|有点像 find()和 split()的结合体,从 str 出现的第一个位置起,把 字 符 串 string 分 成 一 个 3 元 素 的 元 组 (string_pre_str,str,string_post_str),如果 string 中不包含str 则 string_pre_str == string.
`string.replace(str1, str2,  num=string.count(str1))`|把 string 中的 str1 替换成 str2,如果 num 指定，则替换不超过 num 次.
`string.split(str="", num=string.count(str))`	|以 str 为分隔符切片 string，如果 num有指定值，则仅分隔 num 个子字符串
`string.strip([obj])`	|在string上执行 lstrip()和 rstrip()，特定字符obj删除string首部和尾部的空格（默认为空格或换行符）
`string.translate(str, del="")`|根据 str 给出的表(包含 256 个字符)转换 string 的字符，要过滤掉的字符放到 del 参数中


### 3.3 列表
#### 3.3.1 基本定义
**List（列表）**用 `[ ]` 标识，是 python 最通用的复合数据类型。  
列表可以完成大多数集合类的数据结构实现。它支持字符，数字，字符串甚至可以包含列表（即嵌套）。
+ 列表中值的切割也可以用到变量 `[头下标:尾下标]` ，就可以截取相应的列表，从左到右索引默认 0 开始，从右到左索引默认 -1 开始，下标可以为空表示取到头或尾。
+ 加号`+`是列表连接运算符，星号`*`是重复操作。
```
#!/usr/bin/python
list = [ 'runoob', 786 , 2.23, 'john', 70.2 ]
tinylist = [123, 'john']
print list                 # 输出完整列表	['runoob', 786, 2.23, 'john', 70.2]
print list[0]            # 输出列表的第一个元素
print list[1:3]          # 输出第二个至第三个元素 
print list[2:]           # 输出从第三个开始至列表末尾的所有元素 [2.23, 'john', 70.2]
print tinylist * 2       # 输出列表两次 [123, 'john', 123, 'john']
print list + tinylist    # 打印组合的列表
```

#### 3.3.2 列表截取
```
L = ['Google', 'Runoob', 'Taobao']
```
Python表达式|结果|描述
:----:|:-------|:------
L[2]	|'Taobao'	            |读取列表中第三个元素
L[-2]	|'Runoob'	            |读取列表中倒数第二个元素
L[1:]	|['Runoob', 'Taobao']	|从第二个元素开始截取列表

#### 3.3.3 列表函数
   
函数名|描述
:----:|:-------
cmp(list1, list2)	|比较两个列表的元素
len(list)	        |列表元素个数
max(list)	        |返回列表元素最大值
min(list)   	    |返回列表元素最小值
list(seq)	        |将元组转换为列表

#### 3.3.4 列表方法
   
方法名|描述
:----:|:-------
list.append(obj)	|在列表末尾添加新的对象
list.count(obj)	    |统计某个元素在列表中出现的次数
list.extend(seq)	|在列表末尾一次性追加另一个序列中的多个值（用新列表扩展原来的列表）
list.index(obj)	    |从列表中找出某个值第一个匹配项的索引位置
list.insert(index, obj)	|将对象插入列表
list.pop(obj=list[-1])	|移除列表中的一个元素（默认最后一个元素），并且返回该元素的值
list.remove(obj)	|移除列表中某个值的第一个匹配项
list.reverse()	    |反向列表中元素
list.sort([func])	|对原列表进行排序

### 3.4 元组
#### 3.4.1 基本定义
元组是另一个数据类型，类似于List（列表）。
元组用 `"( )"`标识。内部元素用逗号隔开。但是元组不能二次赋值，相当于只读列表。  
**元组中只包含一个元素时，需要在元素后面添加逗号 （括号被解释为优先级表达式）。**
元组中的元素值是不允许删除的，但我们可以使用del语句来删除整个元组。
`tup1 = (50,);`
```
#!/usr/bin/python
tuple = ( 'runoob', 786 , 2.23, 'john', 70.2 )
tinytuple = (123, 'john')
print  tuple               # 输出完整元组
print  tuple[0]            # 输出元组的第一个元素
print  tuple[1:3]          # 输出第二个至第三个的元素 
print  tuple[2:]           # 输出从第三个开始至列表末尾的所有元素
print  tinytuple * 2       # 输出元组两次
print  tuple + tinytuple   # 打印组合的元组
```

#### 3.4.2 元组函数
   
函数名|描述
:----:|:-------
`cmp(tuple1, tuple2)`       |比较两个元组元素
`len(tuple)`	            |计算元组元素个数
`max(tuple)`	            |返回元组中元素最大值
`min(tuple)`	            |返回元组中元素最小值
`tuple(seq)`                |将列表转换为元组


### 3.5 字典
#### 3.5.1 基本定义
字典(dictionary)是除列表以外python之中最灵活的内置数据结构类型。列表是有序的对象集合，字典是无序的对象集合。  
两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。
字典用`"{ }"`标识。字典由索引(key)和它对应的值value组成。
键必须不可变，所以可以用**数字，字x符串或元组**充当，所以用列表就不行。

能删单一的元素也能清空字典，清空只需一项操作。
显示删除一个字典用del命令
```
del dict['Name']; # 删除键是'Name'的条目
dict.clear();     # 清空词典所有条目
del dict ;        # 删除词典
```
```
#!/usr/bin/python
dict = {}
dict['one'] = "This is one"
dict[2] = "This is two"
 
tinydict = {'name': 'john','code':6734, 'dept': 'sales'}
 
print  dict['one']          # 输出键为'one' 的值
print  dict[2]              # 输出键为 2 的值
print  tinydict             # 输出完整的字典
print  tinydict.keys()      # 输出所有键
print  tinydict.values()    # 输出所有值
```
#### 3.5.2 字典函数

函数|描述
:----:|:-------
cmp(dict1, dict2)	|比较两个字典元素。
len(dict)	        |计算字典元素个数，即键的总数。
str(dict)	        |输出字典可打印的字符串表示。
type(variable)	    |返回输入的变量类型，如果变量是字典就返回字典类型。

#### 3.5.3 字典方法

函数|描述
:----:|:-------
`dict.clear()`	    |删除字典内所有元素
`dict.copy()`	        |返回一个字典的浅复制
`dict.deepcopy()`	    |返回一个字典的复制
`dict.fromkeys(seq[, val])`|创建一个新字典，以序列 seq 中元素做字典的键，val 为字典所有键对应的初始值
`dict.get(key, default=None)`|返回指定键的值，如果值不在字典中返回default值
`dict.has_key(key)`	|如果键在字典dict里返回true，否则返回false
`dict.items()`	    |以列表返回可遍历的(键, 值) 元组数组
`dict.keys()`	        |以列表返回一个字典所有的键
`dict.setdefault(key, default=None)`|和get()类似, 但如果键不存在于字典中，将会添加键并将值设为default
`dict.update(dict2)`	|把字典dict2的键/值对更新到dict里
`dict.values()`	    |以列表返回字典中的所有值
`pop(key[,default])`	|删除字典给定键 key 所对应的值，返回值为被删除的值。key值必须给出。 否则，返回default值。
`popitem()`	        |随机返回并删除字典中的一对键和值。


### 3.6 数据类型转换
函数返回一个新的对象，表示转换的值。

函数|描述
:----:|:-------
`int(x [,base])`	|将x转换为一个整数
`long(x [,base] )`	|将x转换为一个长整数
`float(x)`	        |将x转换到一个浮点数
`complex(real [,imag])`	|创建一个复数
`str(x)`	        |将对象 x 转换为字符串
`repr(x)`	        |将对象 x 转换为表达式字符串
`eval(str)`	        |用来计算在字符串中的有效Python表达式,并返回一个对象
`tuple(s)`	        |将序列 s 转换为一个元组
`list(s)`	        |将序列 s 转换为一个列表
`set(s)`	        |转换为可变集合
`dict(d)`	        |创建一个字典。d 必须是一个序列 (key,value)元组。
`frozenset(s)`	    |转换为不可变集合
`chr(x)`	        |将一个整数转换为一个字符
`unichr(x)`	        |将一个整数转换为Unicode字符
`ord(x)`	        |将一个字符转换为它的整数值
`hex(x)`	        |将一个整数转换为一个十六进制字符串
`oct(x)`	        |将一个整数转换为一个八进制字符串


### 3.7 综合应用
```
ch = ‘China’
print ch			// China
print ch [0:]		// China
print ch [:]			// China
print ch [ :-1]		// Chin	-1代表最后一个字符，但不包含高位（即最后一个字符）
print ch[1:-1]		// hin
print ch[1:0]		// 空行
print ch[:0]		// 空行
```
Note: 
+ (1) 在[]内和ch 、[]之间的空格，均不影响输出结果，
+ (2) -1只有在高位才能表示字符串尾部，否则将失去此种意义。

	数字类型中，复数用print输出时加括号。用科学计数法表示时，e紧随的数字是指数项
```
x = 66+1122j
print x			// (66+1122j)
x = 1e+2+244j
print x			// (100+244j)
```
**列表类型变量的值可以为列表类型变量，元组也是如此，和C/C++中数组类似**
```
list = [2018, 101.11, 'China', 66.6]
tinylist = ["who are you?", 232, list]

print  tinylist[2]			// [2018, 101.11, 'China', 66.6]
print  tinylist[2][0]		// 2018
print  tinylist[2][0][1]		// u

tuple = (2018, 101.11, 'China', 66.6)
tinytuple = ("who are you?", 232, tuple)
print  tinytuple			// ('who are you?', 232, (2018, 101.11, 'China', 66.6))
print  tinytuple[2]			// (2018, 101.11, 'China', 66.6)
print  tinytuple[2][0]		// 2018
print  tinytuple[2][0][1]		// u
```

**数字、字符串、列表、元组等定义的变量均可以被另外赋值，并且赋值前后的数据类型可以不一致。特别注意对于列表和元组均适用。**  
```
list = [2018, 101.11, 'China', 66.6]
print list				// [2018, 101.11, 'China', 66.6]
list = 123
print list				// 123
tuple = (2018, 101.11, 'China', 66.6)
print tuple				// (2018, 101.11, 'China', 66.6)
tuple = 2345			
print tuple				// 2345
```
**已定义的列表和字典均可二次赋值，但是已定义的元组不能二次赋值，即不能更新。**
```
list = [2018, 101.11, 'China', 66.6]
print list				// [2018, 101.11, 'China', 66.6]
list [ 1] = 123
print list				//[123, 101.11, 'China', 66.6]

tuple = (2018, 101.11, 'China', 66.6)
print tuple				// (2018, 101.11, 'China', 66.6)
tuple[1] = 2345
print tuple				// 出错，元组不能二次赋值。

dict = {}
dict['one'] = 'This is one'
dict[2] = "This is two"
print dict[2]				// This is two
dict [2] = 124		
print dict[2]				// 124
print [“one”]			// This is one	 
//字典的索引（key）只和数据类型有关，和不同表达形式无关。其中one均为字符串。

tinydict = {3.3: 8899, 'ept':'sales'}	
					// 索引（key）不限定为字符串类型
print tinydict.keys()		// [3.3, 'ept']
print tinydict.values()		// [8899, 'sales']	单独取出索引和值，均表示为列表类型
```
**python 的所有数据类型都是类，可以通过 type() 查看该变量的数据类型。**

 
## 4 Python运算符
### 4.1 算术运算符
假设：`a = 10`, `b = 20`

运算符|描述|实例
:-----:|:-------|:------
`+`	|加 —— 两个对象相加	                    |a + b 输出结果 30
`-`	|减 —— 得到负数或是一个数减去另一个数	  |a - b 输出结果 -10
`*`	|乘 —— 两个数相乘或是返回一个被重复若干次的字符串	|a * b 输出结果 200
`/`	|除 —— x除以y	                        |b / a 输出结果 2
`%`	|取模 —— 返回除法的余数	                 |b % a 输出结果 0
`**`	|幂 —— 返回x的y次幂	                    |a**b 为10的20次方， 输出结果 100000000000000000000
`//`	|取整除 —— 返回商的整数部分               |9//2 输出结果 4 , 9.0//2.0 输出结果 4.0
	
### 4.2 比较（关系）运算符
假设：`a = 10`, `b = 20`

运算符|描述|实例
:-----:|:-------|:------
`==`	|等于 —— 比较对象是否相等		    |(a == b) 返回 False。
`!=`	|不等于 —— 比较两个对象是否不相等	|(a != b) 返回 true.
`< >`	|不等于 —— 比较两个对象是否不相等	|(a <> b) 返回 true
`>`	    |大于 —— 返回x是否大于y	            |(a > b) 返回 False
`<`	    |小于 —— 返回x是否小于y。所有比较运算符返回1表示真，返回0表示假。这分别与特殊的变量True和False等价。	            |(a < b) 返回 true
`>=`	|大于等于	—— 返回x是否大于等于y。	|(a >= b) 返回 False
`<=`	|小于等于 —— 返回x是否小于等于y。	|(a <= b) 返回 true


### 4.3 赋值运算符
假设：`a = 10`, `b = 20`

运算符|描述|实例
:----:|:-------|:------
`=`	    |简单的赋值运算符	
`+=`	|加法赋值运算符	
`-=`	|减法赋值运算符	
`*=`	|乘法赋值运算符	
`/=`	|除法赋值运算符	
`%=`	|取模赋值运算符	
`**=`	|幂赋值运算符	
`//=`	|取整除赋值运算符	


### 4.4 位运算符

运算符|描述|实例
:------:|:-------|:-------
`&`	|按位与运算符：参与运算的两个值,如果两个相应位都为1,则该位的结果为1,否则为0	
`\|`	|按位或运算符：只要对应的二个二进位有一个为1时，结果位就为1。	
`^`	|按位异或运算符：当两对应的二进位相异时，结果为1	
`~`	|按位取反运算符：对数据的每个二进制位取反,即把1变为0,把0变为1	
`<<`	|左移动运算符：运算数的各二进位全部左移若干位，由 << 右边的数字指定了移动的位数，高位丢弃，低位补0。	
`>>`	|右移动运算符：把">>"左边的运算数的各二进位全部右移若干位，>> 右边的数字指定了移动的位数。	

### 4.5 逻辑运算符

运算符|逻辑表达式|描述
:----:|:-------|:------
and	|x and y	|如果 x 为 False，x and y 返回 False；否则它返回 y 的计算值。	
or	|x or y	    |如果 x 是非 0，它返回 x 的值；否则它返回 y 的计算值。	
not	|not x	    |如果 x 为 True，返回 False。如果 x 为 False，它返回 True。	


### 4.6 成员运算符

运算符|描述|实例
:----:|:-------|:------
in	|如果在指定的序列中找到值返回 True，否则返回 False。	
not in	|如果在指定的序列中没有找到值返回 True，否则返回 False。	

### 4.7 身份运算符

运算符|描述|实例
:----:|:-------|:------
is	|is 是判断两个标识符是不是引用自一个对象	x is y, 类似 id(x) == id(y) , 如果引用的是同一个对象则返回 True，否则返回 False
is not	|is not 是判断两个标识符是不是引用自不同对象	x is not y ， 类似 id(a) != id(b)。如果引用的不是同一个对象则返回结果 True，否则返回 False。

注： id() 函数用于获取对象内存地址。   
为了提高内存利用效率对于一些简单的对象，如一些数值较小的int对象，python采取重用对象内存的办法，如指向a=2，b=2时，由于2作为简单的int类型且数值小，python不会两次为其分配内存，而是只分配一次，然后将a与b同时指向已分配的对象。

+ is 与 == 区别：  
is 用于判断两个变量引用对象是否为同一个， == 用于判断引用变量的值是否相等。

### 4.8 运算符优先级

运算符|描述
:----:|:-------
`**`                | 指数 (最高优先级)
`~` `+` `-`	        |按位翻转, 一元加号和减号 (最后两个的方法名为 +@ 和 -@)
`*` `/` `%` `//`	|乘，除，取模和取整除
`+` `-`	            |加法减法
`>>` `<<`	        |右移，左移运算符
`&`	                |位 'AND'
`^`  `\|`	        |位运算符
`<=`  `<` `>` `>=`	|比较运算符
`< >` `==` `!=`	    |等于运算符
`=` `%=` `/=` `//=` `-=` `+=`  `*=`  `**=`	|赋值运算符
`is` `is not`	|身份运算符
`in` `not in`	|成员运算符
`not` `or` `and`	|逻辑运算符

 
## 5 Python语句
### 5.1 条件语句

任何非0和非空（null）值为true，0 或者 null为false。
```
if 判断条件：
    执行语句……
else：
    执行语句……
```
```
if 判断条件1:
    执行语句1……
elif 判断条件2:
    执行语句2……
elif 判断条件3:
    执行语句3……
else:
    执行语句4……
```

### 5.2 循环语句
```
while 判断条件：
    执行语句……
```
```
for iterating_var in sequence:
   statements(s)
```
```
#!/usr/bin/python

fruits = ['banana', 'apple',  'mango']
for  index  in  range(len(fruits)):
   print   fruits[index]
 
print  "Good bye!"
```
+ 函数 len( ) 返回列表的长度，即元素的个数。
+ range( )返回一个序列的数。

**else 语句**  
for … else 表示这样的意思，for 中的语句和普通的没有区别，
else 中的语句会在循环正常执行完（即 for 不是通过 break 跳出而中断的）的情况下执行，while … else 也是一样。
```
#!/usr/bin/python

for num in range(10,20): 	 # 迭代 10 到 20 之间的数字
   for i in range(2,num): 	# 根据因子迭代
      if num%i == 0:      		# 确定第一个因子
         j=num/i          		# 计算第二个因子
         print '%d 等于 %d * %d' % (num,i,j)
         break           		 # 跳出当前循环
   else:                 			 # 循环的 else 部分
      print num, '是一个质数'
```
```
for line in f:
    if line.strip():
        if line[-2] == ':':
            typename = line[0:-2].strip()
        elif line[0:2] == '- ':
            partname = line[2:].split('#')[0].strip()
            parttypes[partname] = typename
        else:
            pass
```


### 5.3 break / continue / pass语句

+ `break语句`用来终止循环语句，即循环条件没有False条件或者序列还没被完全递归完，也会停止执行循环语句。break语句用在while和for循环中。
+ `continue语句`用来告诉Python跳过当前循环的剩余语句，然后继续进行下一轮循环。continue语句用在while和for循环中。
+ `pass是空语句`，是为了保持程序结构的完整性。pass 不做任何事情，一般用做占位语句。
```
# 输出 Python 的每个字母
for letter in 'Python':
   if letter == 'h':
      pass
      print '这是 pass 块'
   print '当前字母 :', letter
print "Good bye!"
```

## 6 Python函数和模块
### 6.1 函数定义和调用
基本规则：  
+ 函数代码块以 def 关键词开头，后接函数标识符名称和圆括号( )。
+ 任何传入参数和自变量必须放在圆括号中间。圆括号之间可以用于定义参数。
+ 函数的第一行语句可选择性地使用文档字符串—用于存放函数说明。
+ 函数内容以冒号起始，并且缩进。
+ `return [表达式]` 结束函数，选择性地返回一个值给调用方。不带表达式的return相当于返回 None。

语法：  
```
def  functionname ( parameters ):
    "函数_文档字符串"		# 这行不会被打印
    function_suite
    return [expression]
```
Example: 
```
def  printme( str ):
    "打印传入的字符串到标准显示设备上"
    print str
    return
```

### 6.2 参数传递
#### 6.2.1 可更改（mutable）与不可更改（inmutable）对象
+ 不可更改对象：**数字（numbers）**、**字符串（strings）**、**元组（tuples）**  
变量赋值 a=5 后再赋值 a=10，这里实际是新生成一个 int 值对象 10，再让 a 指向它，而 5 被丢弃。  
参数传递时，类似 c++ 的值传递，如整数、字符串、元组。如`fun（a）`，传递的只是a的值，没有影响a对象本身。比如在 `fun（a）`内部修改 a 的值，只是修改另一个复制的对象，不会影响 a 本身。

+ 可更改对象：列表（list）、字典（dict）  
变量赋值 `la=[1,2,3,4]` 后再赋值 `la[2]=5` 则是将 list la 的第三个元素值更改，本身la没有动，只是其内部的一部分值被修改了。  
类似 c++ 的引用传递，如 列表，字典。如 `fun（la）`，则是将 la 真正的传过去，修改后fun外部的la也会受影响。


#### 6.2.2 函数调用时的参数类型
+ 必备参数
    + 须以正确的顺序传入函数。调用时的数量必须和声明时的一样。

+ 关键字参数
    + 关键字参数和函数调用关系紧密，函数调用使用**关键字参数**来确定传入的参数值。
    + 使用关键字参数**允许函数调用时参数的顺序与声明时不一致**，因为 Python 解释器能够用参数名匹配参数值。  


```
#!/usr/bin/python

#可写函数说明
def printme( str ):
   "打印任何传入的字符串"
   print str;
   return;
 
#调用printme函数
printme( str = "My string");
```

+ 默认（缺省）参数  
    + 调用函数时，缺省参数的值如果没有传入，则被认为是默认值。


```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
#可写函数说明
def printinfo( name, age = 35 ):
   "打印任何传入的字符串"
   print "Name: ", name;
   print "Age ", age;
   return;
 
#调用printinfo函数
printinfo( age=50, name="miki" );
printinfo( name="miki" );
```

+ 不定长参数  
语法：加了星号（*）的变量名会存放所有未命名的变量参数。
```
def functionname([formal_args,] *var_args_tuple ):
       "函数_文档字符串"
       function_suite
       return [expression]
```


```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 可写函数说明
def printinfo( arg1, *vartuple ):
   "打印任何传入的参数"
   print "输出: "
   print arg1
   for var in vartuple:
      print var
   return;
 
# 调用printinfo 函数
printinfo( 10 );
printinfo( 70, 60, 50 );
```

### 6.3 匿名函数
使用 lambda 来创建匿名函数：  
+ lambda只是一个表达式，函数体比def简单很多。
+ lambda的主体是一个表达式，而不是一个代码块。仅仅能在lambda表达式中封装有限的逻辑进去。
+ lambda函数拥有自己的命名空间，且不能访问自有参数列表之外或全局命名空间里的参数。
+ 虽然lambda函数看起来只能写一行，却不等同于C或C++的内联函数，后者的目的是调用小函数时不占用栈内存从而增加运行效率。  
语法：  
`lambda  [arg1  [,arg2,.....argn]] : expression`


```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 可写函数说明
sum = lambda arg1, arg2: arg1 + arg2;
 
# 调用sum函数
print "相加后的值为 : ", sum( 10, 20 )
print "相加后的值为 : ", sum( 20, 20 )
```

### 6.4 变量作用域
+ 全局变量：定义在函数外
+ 局部变量：定义在函数内部的变量
	+ 全局变量想作用于函数内，需加 global

### 6.5 模块
Python 模块(Module)，是个 Python 文件，以 `.py` 结尾，包含了 Python 对象定义和Python语句。
+ `import 语句`  
语法：  

```
import module1[, module2[,... moduleN]
```
引用：  **模块名.函数名**

+ `from…import语句`  
从模块中导入一个指定的部分到当前命名空间中。  
```
from  modname  import  name1[, name2[, ... nameN]]
from  fib import  fibonacci
```
+ `from…import* 语句`   
把一个模块的所有内容全都导入到当前的命名空间。
```
from modname import *
```

+ `dir()函数`   
dir() 函数返回一个排好序的字符串列表，内容是一个模块里定义过的名字。
返回的列表容纳了在一个模块里定义的所有模块，变量和函数。

```
import math

content = dir(math)

print content;
['__doc__', '__file__', '__name__', 'acos', 'asin', 'atan', 
'atan2', 'ceil', 'cos', 'cosh', 'degrees', 'e', 'exp', 
'fabs', 'floor', 'fmod', 'frexp', 'hypot', 'ldexp', 'log',
'log10', 'modf', 'pi', 'pow', 'radians', 'sin', 'sinh', 
'sqrt', 'tan', 'tanh']
```

其中：特殊字符串变量`__name__`指向模块的名字，`__file__`指向该模块的导入文件名。


### 6.6 包
包是一个分层次的文件目录结构，它定义了一个由模块及子包，和子包下的子包等组成的 Python 的应用环境。  
包就是文件夹，但该文件夹下必须存在 `__init__.py`文件, 该文件的内容可以为空。`__init__.py`用于标识当前文件夹是一个包。   


```
test.py
package_runoob
|--  __init__.py
|--  runoob1.py
|--  runoob2.py
```

```
package_runoob/runoob1.py
#!/usr/bin/python
def runoob1():
   print "I'm in runoob1"
```

```
package_runoob/runoob2.py
#!/usr/bin/python

def runoob2():
   print "I'm in runoob2"
```

```
package_runoob/__init__.py
#!/usr/bin/python

if __name__ == '__main__':
    print '作为主程序运行'
else:
print 'package_runoob 初始化'
```

```
test.py
#!/usr/bin/python

# 导入 Python 包
from package_runoob.runoob1 import runoob1
from package_runoob.runoob2 import runoob2
 
runoob1()
runoob2()
```

```
Result : 
package_runoob 初始化
I'm in runoob1
I'm in runoob2
```

 
## 7 Python文件I/O
### 7.1 打印
+ `print`：可以传递零个或多个用逗号隔开的表达式。此函数把传递的表达式转换成一个字符串表达式，并将结果写到标准输出。

### 7.2 读取键盘输入
+ `raw_input`函数  
从标准输入读取一个行，并返回一个字符串（去掉结尾的换行符）。
`name=raw_input('输入姓名：')`
所有输入的数据均转换为字符串。

+ `input`函数  
可以接收一个Python表达式作为输入，并将运算结果返回。

```
#!/usr/bin/python
# -*- coding: UTF-8 -*- 
 
str = input("请输入：")
print "你输入的内容是: ", str


请输入：[x*5 for x in range(2,10,2)]
你输入的内容是:  [10, 20, 30, 40]
```

### 7.3 打开和关闭文件
#### 7.3.1 open 函数

```
file object = open(file_name  [, access_mode] [, buffering])
```

+ `file_name`：file_name变量是一个包含了要访问的文件名称的字符串值。
+ `access_mode`：access_mode决定了打开文件的模式：只读，写入，追加等。所有可取值见如下的完全列表。这个参数是非强制的，默认文件访问模式为只读(r)。
+ `buffering`：如果buffering的值被设为0，就不会有寄存。如果buffering的值取1，访问文件时会寄存行。如果将buffering的值设为大于1的整数，表明了这就是的寄存区的缓冲大小。如果取负值，寄存区的缓冲大小则为系统默认。

符号|描述
:----:|:-------
r	|以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式。  
rb	|以二进制格式打开一个文件用于只读。文件指针将会放在文件的开头。这是默认模式。一般用于非文本文件如图片等。  
r+	|打开一个文件用于读写。文件指针将会放在文件的开头。  
rb+	|以二进制格式打开一个文件用于读写。文件指针将会放在文件的开头。一般用于非文本文件如图片等。  
w	|打开一个文件只用于写入。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。  
wb	|以二进制格式打开一个文件只用于写入。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。一般用于非文本文件如图片等。  
w+	|打开一个文件用于读写。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。  
wb+	|以二进制格式打开一个文件用于读写。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。一般用于非文本文件如图片等。  
a	|打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。  
ab	|以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。  
a+	|打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾。文件打开时会是追加模式。如果该文件不存在，创建新文件用于读写。  
ab+	|以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。如果该文件不存在，创建新文件用于读写。  

```
with open () as f:
```

#### 7.3.2 File对象的属性
属性|描述
:----:|:-------
`file.closed`	|返回true如果文件已被关闭，否则返回false。
`file.mode`	|返回被打开文件的访问模式
`file.name`	|返回文件的名称。
`file.softspace`	|如果用print输出后，必须跟一个空格符，则返回false。否则返回true。
```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
# 打开一个文件
fo = open("foo.txt", "w")
print "文件名: ", fo.name
print "是否已关闭 : ", fo.closed
print "访问模式 : ", fo.mode
print "末尾是否强制加空格 : ", fo.softspace
```

#### 7.3.3 close( )方法
File 对象的 close（）方法刷新缓冲区里任何还没写入的信息，并关闭该文件，这之后便不能再进行写入。
```
fileObject. close()
```

#### 7.3.4 write( )方法
可将任何字符串写入一个打开的文件。需要重点注意的是，**Python字符串可以是二进制数据，而不是仅仅是文字**。**write()方法不会在字符串的结尾添加换行符('\n')**
```
fileObject. write(string)
```


#### 7.3.5 read( )方法
从一个打开的文件中读取一个字符串。需要重点注意的是，Python字符串可以是二进制数据，而不是仅仅是文字。
```
fileObject. read([count])
```
被传递的参数是**要从已打开文件中读取的字节计数**。该方法**从文件的开头开始读入**，如果没有传入count，它会尝试尽可能多地读取更多的内容，很可能是直到文件的末尾。

#### 7.3.6 文件定位
+ `tell()`方法：告诉你文件内的当前位置，换句话说，下一次的读写会发生在文件开头这么多字节之后。
+ `seek(offset[,from])`方法：改变当前文件的位置。Offset变量表示要移动的字节数。From变量指定开始移动字节的参考位置。  
如果from被设为0，这意味着将文件的开头作为移动字节的参考位置。  
如果设为1，则使用当前的位置作为参考位置。  
如果被设为2，那么该文件的末尾将作为参考位置。  

#### 7.3.7 重命名和删除文件（os模块）
`import os`
+ `rename()`方法  
rename()方法需要两个参数，当前的文件名和新文件名。
```
os.rename(current_file_name, new_file_name)
```

+ `remove()`方法  
用remove()方法删除文件，需要提供要删除的文件名作为参数。
```
os.remove(file_name)
```
+ `mkdir()`方法  
可以使用os模块的mkdir()方法在当前目录下创建新的目录们
```
os.mkdir("newdir")
```
+ `chdir()`方法  
用chdir()方法来改变当前的目录。chdir()方法需要的一个参数是你想设成当前目录的目录名称。
```
os.chdir("newdir")
```
+ `getcwd()`方法  
显示当前的工作目录。
```
os.getcwd()
```
+ `rmdir()`方法  
删除目录，目录名称以参数传递。
```
os.rmdir('dirname')
```

### 7.4 File（文件）方法

方法|描述
:----:|:-------
`file.close()`	|关闭文件。关闭后文件不能再进行读写操作。
`file.flush()`	|刷新文件内部缓冲，直接把内部缓冲区的数据立刻写入文件, 而不是被动的等待输出缓冲区写入。
`file.fileno()`	|返回一个整型的文件描述符(file descriptor FD 整型), 可以用在如os模块的read方法等一些底层操作上。
`file.isatty()`	|如果文件连接到一个终端设备返回 True，否则返回 False。
`file.next()`	|返回文件下一行。
`file.read([size])`	|从文件读取指定的字节数，如果未给定或为负则读取所有。
`file.readline([size])`	|读取整行，包括 "\n" 字符。
`file.readlines([sizehint])`	|读取所有行并返回列表，若给定sizeint>0，则是设置一次读多少字节，这是为了减轻读取压力。
`file.seek(offset[, whence])`	|设置文件当前位置
`file.tell()`	|返回文件当前位置。
`file.truncate([size])`	|截取文件，截取的字节通过size指定，默认为当前文件位置。
`file.write(str)`	|将字符串写入文件，没有返回值。
`file.writelines(sequence)`	|向文件写入一个序列字符串列表，如果需要换行则要自己加入每行的换行符。


## 8 Python异常处理
### 8.1 异常捕捉try....except...else
```
try:
<语句>        #运行别的代码
except <名字>：
<语句>        #如果在try部份引发了'name'异常
except <名字>，<数据>:
<语句>        #如果引发了'name'异常，获得附加的数据
else:
<语句>        #如果没有异常发生
```
工作原理：当开始一个try语句后，python就在当前程序的上下文中作标记，这样当异常出现时就可以回到这里。
+ 如果当try后的语句执行时**发生异常**，python就跳回到try并执行第一个匹配该异常的except子句，异常处理完毕，控制流就通过整个try语句（除非在处理异常时又引发新的异常）。
+ 如果在try后的语句里**发生异常**，却没有匹配的except子句，异常将被递交到上层的try，或者到程序的最上层（这样将结束程序，并打印缺省的出错信息）。
+ 如果在try子句执行时**没有发生异常**，python将执行else语句后的语句（如果有else的话），然后控制流通过整个try语句。


### 8.2使用except而不带任何异常类型
```
try:
    正常的操作
   ......................
except:
    发生异常，执行这块代码
   ......................
else:
    如果没有异常执行这块代码
```

### 8.3使用except而带多种异常类型
```
try:
    正常的操作
   ......................
except(Exception1[, Exception2[,...ExceptionN]]]):
   发生以上多个异常中的一个，执行这块代码
   ......................
else:
    如果没有异常执行这块代码
```

### 8.4 try-finally 语句
```
try-finally 语句无论是否发生异常都将执行最后的代码。
try:
<语句>
finally:
<语句>    #退出try时总会执行
raise
```

### 8.5 异常的参数
一个异常可以带上参数，可作为输出的异常信息参数。
```
try:
    正常的操作
   ......................
except ExceptionType, Argument:
    你可以在这输出 Argument 的值...
```
变量接收的异常值通常包含在异常的语句中。在元组的表单中变量可以接收一个或者多个值。
元组通常包含错误字符串，错误数字，错误位置。
```
#!/usr/bin/python
# -*- coding: UTF-8 -*-

# 定义函数
def temp_convert(var):
    try:
        return int(var)
    except ValueError, Argument:
        print "参数没有包含数字\n", Argument

# 调用函数
temp_convert("xyz");

$ python test.py 
参数没有包含数字
invalid literal for int() with base 10: 'xyz'
```

### 8.6 触发异常
```
raise [Exception [, args [, traceback]]]
```
语句中 Exception 是异常的类型（例如，NameError）参数标准异常中任一种，args 是自已提供的异常参数。最后一个参数是可选的（在实践中很少使用），如果存在，是跟踪异常对象。
```
#!/usr/bin/python
# -*- coding: UTF-8 -*-

# 定义函数
def mye( level ):
    if level < 1:
        raise Exception,"Invalid level!"
        # 触发异常后，后面的代码就不会再执行
try:
    mye(0)            # 触发异常
except Exception,err:
    print 1,err
else:
    print 2
```

## 9 Python面向对象
### 9.1 单下划线、双下划线、头尾双下划线说明
+ `__foo__`: 定义的是特殊方法，一般是系统定义名字 ，类似 __init__() 之类的。
+ `_foo`: 以单下划线开头的表示的是 protected 类型的变量，即保护类型只能允许其本身与子类进行访问，不能用于 `from module import *`
+ `__foo`: 双下划线的表示的是私有类型(private)的变量, 只能是允许这个类本身进行访问了。

### 9.2 创建类
```
class ClassName:
   '类的帮助信息'   #类文档字符串
   class_suite  #类体
```
```
#!/usr/bin/python
class Employee:
  	 '所有员工的基类'
  	 empCount = 0
  	 def __init__(self, name, salary):
     		 self.name = name
      		 self.salary = salary
     		 Employee.empCount += 1
   	  def displayCount(self):
     		  print "Total Employee %d" % Employee.empCount
   	  def displayEmployee(self):
      		print "Name : ", self.name,  ", Salary: ", self.salary
```
empCount 变量是一个类变量，它的值将在这个类的所有实例之间共享。你可以在内部类或外部类使用 Employee.empCount 访问。

### 9.3 创建实例对象
实例化类其他编程语言中一般用关键字 new，但是在 Python 中并没有这个关键字，类的实例化类似函数调用方式。

### 9.4 内置类属性
+ `__dict__` : 类的属性（包含一个字典，由类的数据属性组成）
+ `__doc__` :类的文档字符串
+ `__name__`: 类名
+ `__module__`: 类定义所在的模块（类的全名是`'__main__.className'`，如果类位于一个导入模块mymod中，那么`className.__module__` 等于 `mymod`）
+ `__bases__` : 类的所有父类构成元素（包含了一个由所有父类组成的元组）

### 9.5 类属性和方法
+ 私有属性
`__private_attrs`：两个下划线开头，声明该属性为私有，不能在类的外部被使用或直接访问。在类内部的方法中使用时 `self.__private_attrs`。

Python不允许实例化的类访问私有数据，但你可以使用`object._className__attrName`（**对象名._类名__私有属性名**）访问属性。

+ 类的方法
在类的内部，使用 `def` 关键字可以为类定义一个方法，与一般函数定义不同，类方法必须包含参数 `self`,且为第一个参数。self 代表的是类的实例，代表当前对象的地址，而 `self.class` 则指向类。

+ 类的私有方法
`__private_method`：两个下划线开头，声明该方法为私有方法，不能在类地外部调用。在类的内部调用 `self.__private_methods`。

### 9.6 访问属性
使用点号 . 来访问对象的属性。
也可以使用以下函数的方式来访问属性：
+ `getattr(obj, name[, default])` : 访问对象的属性。
+ `hasattr(obj,name)` : 检查是否存在一个属性。
+ `setattr(obj,name,value)` : 设置一个属性。如果属性不存在，会创建一个新属性。
+ `delattr(obj, name)` : 删除属性。 

## 10 Python 正则表达式
### 10.1 匹配的几种方式
+ `re.match` ：尝试从字符串的起始位置匹配一个模式，如果不是起始位置匹配成功的话，match()就返回none。
+ `re.search` ：扫描整个字符串并返回第一个成功的匹配。
+ `re.findall`：`findall(pattern, string, flags=0)` 
在string中查找所有 匹配成功的组, 即用括号括起来的部分。返回list对象，每个list item是由每个匹配的所有组组成的list

### 10.2 特殊分组方法
+ `(?P<name>)`	分组，除了原有的编号外再指定一个额外的别名	(?P<id>abc){2}	abcabc  
+ `(?P=name)`	引用别名为<name>的分组匹配到字符串	(?P<id>\d)abc(?P=id)	1abc1
5abc5  
+ `\<number>`	引用编号为<number>的分组匹配到字符串	(\d)abc\1	1abc1
5abc5

### 10.3 regEx with multiple groups
Your regex only contains a single pair of parentheses (one capturing group), so you only get one group in your match. If you use a repetition operator on a capturing group (+ or *), the group gets "overwritten" each time the group is repeated, meaning that only the last match is captured.
In your example here, you're probably better off using .split(), in combination with a regex:
```
lun_q = 'Lun:\s*(\d+(?:\s+\d+)*)'
s = '''Lun: 0 1 2 3 295 296 297 298'''

r = re.search(lun_q, s)

if r:
    luns = r.group(1).split()

    # optionally, also convert luns from strings to integers
    luns = [int(lun) for lun in luns]
```

## 11 Python常用模块
### 11.1 sys模块
+ `sys.argv`: 实现从程序外部向程序传递参数。
+ `sys.exit([arg])`: 程序中间的退出，arg=0为正常退出。
+ `sys.getdefaultencoding()`: 获取系统当前编码，一般默认为ascii。
+ `sys.setdefaultencoding()`: 设置系统默认编码，执行dir（sys）时不会看到这个方法，在解释器中执行不通过，可以先执行reload(sys)，在执行 setdefaultencoding('utf8')，此时将系统默认编码设置为utf8。（见设置系统默认编码 ）
+ `sys.getfilesystemencoding()`: 获取文件系统使用编码方式，Windows下返回'mbcs'，mac下返回'utf-8'.
+ `sys.path`: 获取指定模块搜索路径的字符串集合，可以将写好的模块放在得到的某个路径下，就可以在程序中import时正确找到。主要用于设置import的文件路径。
两个方法可以将模块路径加到当前模块扫描的路径里：
    - `sys.path.append('你的模块的名称')`
    - `sys.path.insert(0,'模块的名称')`

+ `sys.platform`: 获取当前系统平台。
+ `sys.stdin,sys.stdout,sys.stderr`: stdin , stdout , 以及stderr 变量包含与标准I/O 流对应的流对象. 如果需要更好地控制输出,而print 不能满足你的要求, 它们就是你所需要的. 你也可以替换它们, 这时候你就可以重定向输出和输入到其它设备( device ), 或者以非标准的方式处理它们

### 11.2 os模块
+ `os.listdir()方法`  
用于返回指定的文件夹包含的文件或文件夹的名字的列表。这个列表以字母顺序。 它不包括 '.' 和'..' 即使它在文件夹中。

+ `os.walk()方法`  
用于通过在目录树中游走输出在目录中的文件名，向上或者向下。
语法：os.walk (top[, topdown=True[, onerror=None[, followlinks=False]]])
参数：
    - `top` -- 是你所要遍历的目录的地址, 返回的是一个三元组(root,dirs,files)。
    - `root` 所指的是当前正在遍历的这个文件夹的本身的地址
    - `dirs` 是一个 list ，内容是该文件夹中所有的目录的名字(不包括子目录)
files 同样是 list , 内容是该文件夹中所有的文件(不包括子目录)
    - topdown` --可选，为 True，则优先遍历 top 目录，否则优先遍历 top 的子目录(默认为开启)。如果 topdown 参数为 True，walk 会遍历top文件夹，与top 文件夹中每一个子目录。
    - `onerror` -- 可选， 需要一个 callable 对象，当 walk 需要异常时，会调用。
    - `followlinks` -- 可选， 如果为 True，则会遍历目录下的快捷方式(linux 下是 symbolic link)实际所指的目录(默认关闭)。
返回值：无

+ `os.path.join()`
对路径进行拼接。  
语法：os.path.join(path1[,path2[,......]])

+ `os.path.split(path)`  
将path分割成目录和文件名二元组返回。
```
>>> os.path.split('c:\\csv\\test.csv') 
('c:\\csv', 'test.csv') 
>>> os.path.split('c:\\csv\\') 
('c:\\csv', '')
```
+ `os.path.dirname(path)`   
返回path的目录。其实就是os.path.split(path)的第一个元素。

+ `os.path.basename(path)`  
返回path最后的文件名。如何path以／或\结尾，那么就会返回空值。  
path='D:\CSDN'	os.path.basename(path)=CSDN

+ `os.path.isfile(path)`  
如果path是一个存在的文件，返回True。否则返回False。 


### 11.3 set
集合中包含一系列的元素，在Python中这些元素不需要是相同的类型，且这些元素在集合中是没有存储顺序的。  
表示方法：集合的表示方法是花括号，这与字典是一样的，可以通过括号或构造函数来初始化一个集合。  
由于集合和字典都用{}表示，所以初始化空的集合只能通过set()操作，{}只是表示一个空的字典  
+ 集合的增加  
单个元素的增加用add方法，对序列的增加用update方法。
+ 集合的删除  
集合删除单个元素有两种方法，两者的区别是在元素不在原集合中时是否会抛出异常，`set.discard(x)`不会，`set.remove(x)`会抛出KeyError错误。
+ 集合操作
    - 并集：`set.union(s)`,也可以用a|b计算
    - 交集：`set.intersection(s)`，也可以用a&b计算
    - 差集：`set.difference(s)`，也可以用a-b计算

+ 包含关系
    - `set.isdisjoint(s)`：判断两个集合是不是不相交
    - `set.issuperset(s)`：判断集合是不是包含其他集合，等同于a>=b
    - `set.issubset(s)`：判断集合是不是被其他集合包含，等同于a<=b

### 11.4 argparse
```
telnet_patten_parser.add_argument("port", default="", nargs="?",
                                      help = "extra port number for telnet connection")
nargs="?"：可选参数为一个字符串
nargs="+" “*”： 可选参数形成列表
```

### 11.5 pexpect
+ `pexpect.buffer`   -- 动态保存每一次expect后的所有内容. before/after都依赖此内容;
+ `pexpect.before`   -- 匹配到的关键字之外的字符;expect后会设置before/after, 具体参考附录，摘录一段文字如下：
```
There are two important methods in Pexpect – expect() and send() (or sendline() which is like send() with a linefeed). The expect() method waits for the child application to return a given string. The string you specify is a regular expression, so you can match complicated patterns. The send() method writes a string to the child application. From the child’s point of view it looks just like someone typed the text from a terminal. After each call to expect() the before and after properties will be set to the text printed by child application. 
•	The before property will contain all text up to the expected string pattern. 
•	The after string will contain the text that was matched by the expected pattern. 
•	The match property is set to the re match object.
```



### 参考资料
    
[http://www.runoob.com/python/python-tutorial.html](http://www.runoob.com/python/python-tutorial.html)