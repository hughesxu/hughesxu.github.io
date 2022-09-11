---
title:  "学习笔记-TCL"
date:   2021-05-01 09:00:00 +0800
categories: [Language, TCL]
tags: [Learning-Note]
---



## Tcl 脚本初窥
```
// helloWorld.tcl
#!/usr/bin/tclsh			// 脚本解释器
puts "Hello World!"
```

## 1. Tcl 基本语法
### 1.1 命令语法
```
commandName  argument1  argument2 ...  argumentN
```

### 1.2 命令替换
在命令替换，方括号是用来计算在方括号中的脚本。
```
#!/usr/bin/tclsh
puts [expr 1 + 6 + 9]
```

### 1.3 变量替换
在变量替换，`$`使用在变量名之前，这将返回该变量的内容。

### 1.4 反斜杠替换
转义序列，每个反斜线后跟有自己的含义字母。

### 1.5 argc / argv
`argv` 从tcl文件后的第一个参数算起，初始为0  
`argc`为tcl文件后接的参数个数

 
## 2. Tcl数据类型
### 2.1 字符串
#### 2.1.1 字符串表示
- 不需要包含双引号，它只有一个字；
- 表示多个字符串，可以使用双引号或大括号。
- 需要注意的是，将整数赋给变量时，也是以字符串形式存储的。但数学运算时会转换为数字 
（llx Without a size modifier, the numeric conversion types are all subject to underflow/overflow. The ll size modifier configures a conversion type to operate on a number of arbitrary size）。
- 布尔值，可以表示为1，yes 或 true 为真值和0，no 或 false 为假值。
```
#!/usr/bin/tclsh
set  myVariable  "hello world"
puts $myVariable
set  myVariable {hello world}
puts $myVariable
```

#### 2.1.2 字符串转义序列

转义序列|意思
|:----:|:----
`\\`	|`\` 字符
`\'`	|`'` 字符
`\"`	|`"` 字符
`\?`	|`?` 字符
`\a`	|警报或铃
`\b`	|退格
`\f`	|换页
`\n`	|新一行
`\r`	|回车
`\t`	|水平制表
`\v`	|垂直制表

#### 2.1.3 字符串命令

方法|描述
|:----:|:----
`string compare string1 string2`	|比较字string1和string2字典顺序。如果相等返回0，如果string1在string2出现之前返回-1，否则返回1。
`string first string1 string2`	|返回索引string1在string2中出现的第一次。如果没有找到，返回-1。
`string index string index`	|返回索引的字符。
`string last string1 string2`	|返回索引string1在string2中出现的最后一次。如果没有找到，返回-1
`string length string`	|返回字符串的长度。
`string match pattern string`	|返回1，如果该字符串匹配模式。
`string range string index1 index2`	|返回指定索引范围内的字符串，index1到index2。
`string tolower string`	|返回小写字符串。
`string toupper string`	|返回大写字符串。
`string trim string  ?trimcharacters?`	|删除字符串两端的trimcharacters。默认trimcharacters是空白。
`string trimleft string  ?trimcharacters?`	|删除字符串左侧开始的trimcharacters。默认trimcharacters是空白。
`string trimright string  ?trimcharacters?`	|删除字符串右端trimcharacters。默认trimcharacters是空白。
`string wordend findstring index`	|返回索引字符findstring包含字符索引单词。
`string wordstart findstring index`	|返回findstring中第一个字符的含有索引中的字符索引的单词。
`string map ?-nocase?  charMap  str`	|返回根据 charMap 中输入、输出列表将 str 中的字符进行映射后而产生的新字符串。
`string  equal ?-nocase? str1   str2`	|比较字符串，相同返回 1，否则返回 0

#### 2.1.4 字符串长度
```
#!/usr/bin/tclsh
set s1 "Hello World"
puts "Length of string s1"
puts [string length $s1]
```
```
Length of string s1
11
```

#### 2.1.5 Append命令
```
#!/usr/bin/tclsh
set s1 "Hello" 
append s1 " World"
puts $s1
```
```
Hello World
```

#### 2.1.6 Format命令
指示符|使用
|:----:|:----
`%s`	|字符串表示
`%d`	|整数表示
`%f`	|浮点表示
`%e`	|指数形式浮点表示
`%x`	|十六进制表示
```
#!/usr/bin/tclsh

puts [format "%f" 43.5]
puts [format "%e" 43.5]
puts [format "%d %s" 4 tuts]
puts [format "%s" "Tcl Language"]
puts [format "%x" 40]
```
```
43.500000
4.350000e+01
4 tuts
Tcl Language
28
```

#### 2.1.7 Scan命令
`scan`命令用于分析基于对格式说明的字符串，可以认为是`format`命令的逆，其功能类似于ANSI C中的`sscanf`函数。它按`format`提供的格式分析string字符串，然后把结果存到变量`varName`中，注意除了空格和TAB键之外，`string`和`format`中的字符和`%`必须匹配。    
语法：`scan  string  format  varName  ?varName ...?`  `scan`命令的返回值是匹配的变量个数。
```
% scan "some 26 34" "some %d %d" a b
2
% set a
26
% set b
34
% scan "12.34.56.78" "%d.%d.%d.%d" c d e f
4
% puts [format "the value of c is %d,d is %d,e is %d ,f is %d" $c $d $e $f]
the value of c is 12,d is 34,e is 56 ,f is 78
```

```
#!/usr/bin/tclsh
puts [scan "90" \{\%\[0-9\]\} m]
puts [scan "abc" \{\%\[a-z\]\} m]
puts [scan "abc" \{\%\[A-Z\]\} m]
puts [scan "ABC" \{\%\[A-Z\]\} m]

1
1
0
1
```

### 2.2 列表——List
#### 2.2.1 列表表示
一组单词或者使用双引号或大括号可以用来表示一个简单的列表。
```
set listName { item1 item2 item3 .. itemn }
# or
set listName [list item1 item2 item3]
# or 
set listName [split "items separated by a character" split_character]
```
**split_character需要用特定的符号代替。**
```
#!/usr/bin/tclsh

set colorList1 {red green blue}
set colorList2 [list red green blue]
set colorList3 [split "red_green_blue" _]
puts $colorList1
puts $colorList2
puts $colorList3

#!/usr/bin/tclsh
set  myVariable  {red green blue}
puts [lindex $myVariable 2]
set myVariable "red green blue"
puts [lindex $myVariable 1]
```

#### 2.2.2 追加列表项目
```
append listName split_character value
# or
lappend listName value
```		
**Note**:  
- lappend默认以空格对list的各个元素进行分割。
- append 和lappend均不需要在list前面加变量 $符号
eg:
```
set listname [list red blue green]
puts $listname
puts [lindex $listname 2]
append listname "." yellow
puts $listname
puts [lindex $listname 2]
lappend listname black
puts $listname
```
```
Result :
red blue green
green
red blue green.yellow
green.yellow
red blue green.yellow black
```

#### 2.2.3 列表长度
```
[ llength  $listName ]
```
Note : 不能单独作为一行语句，要结合puts和set等

#### 2.2.4 列表索引项
```
[ lindex  $listname  index ]
```
Note : 不能单独作为一行语句，要结合puts和set等

#### 2.2.5 插入索引项目
```
[ linsert   $listname index value1 value2..valuen ]
```
Note : 不能单独作为一行语句，要结合puts和set等

#### 2.2.6 更换索引项目
```
[ lreplace  $listname  firstindex lastindex value1 value2..valuen ]
```
eg:
```
set var {orange blue red green}
set var [lreplace $var 2 3 black white]
```
Note : 不能单独作为一行语句，要结合puts和set等

#### 2.2.7 设置指数项目
```
lset  listname  index  value 
```
用于设置列表项在特定索引下面的语法给出。
```
#!/usr/bin/tclsh
set var {orange blue red green}
lset var 0 black 
puts $var
```
Note:   `lset`不需要在list前面加变量 `$`符号

#### 2.2.7 转换列表变量
```
lassign  $listname  variable1 variable2.. variablen
set var {orange blue red green}
lassign $var colour1 colour2
puts $colour1
puts $colour2
```

#### 2.2.8 排序列表
```
lsort  listname
```
```
set var {orange blue red green}
set var [lsort $var]
puts $var
```

#### 2.2.9 列表元素合并字符串
```
join
```
- 语法：`join list ?joinString?`  
- 描述：list必须是一个有效的列表，这个命令返回一个用joinString间隔开list的每个元素所组成的字符串。

#### 2.2.10 列表中寻找特定元素
- 语法 : `lsearch ?options?  list   pattern`
- 描述：这个命令寻找list中与pattern匹配的元素。如果匹配到了返回第一次找到这个元素的索引（除非指定了-all或-inline），如果没有匹配到返回-1。可选变元指出了列表的元素怎样去匹配pattern

- **匹配风格可选项**  
默认的匹配风格为`-glob`，如果给出了多个匹配风格，那么最后给出的匹配风格有效。
    - `-exact`  
pattern是字符串并且必须严格匹配每一个list元素。
    - `-glob`
pattern是通配风格匹配每一个列表元素，与string match命令类似。
    - `-regexp`  
pattern被当作一个正则表达式来匹配列表中的每一个元素，与re_syntax命令类似。
    - `-sorted`  
列表元素被排序，如果指定了这个可选项，`lsearch`将会使用更加有效的查询算法去查列表。如果没有指定其它的可选项，`list`将被排列成升序并且包含ASCII码。这个可选项与`-glob`和`-regexp`互斥，当指定了`-all`或`-not`时与`-exact`非常象。

- **一般修饰可选项**  
这些可选项可以在任何一种匹配风格中给出。
    - `-all`  
返回一个列表，列表的元素为所有匹配到的索引，返回的索引按照数字顺序排列，如果同时指定了-inline返回数值，数值的顺序就是在列表中的顺序。

    - `-inline`  
返回匹配到的第一个元素。如果-all也指定了，则返回一个列表，列表的元素为所有匹配到的数值。

    - `-not`  
与匹配相反，返回第一个不匹配的数值的索引。

    - `-start index`  
从列表的index个索引开始搜索。

- **内容描述可选项**  
描述如何去解释在列表中寻找到的元素，只有当`-exact`和`-sorted`指定时才有效。如果给出了多个可选项，那么最后给出的可选项有效。默认为-ascii。
    - `-ascii`    
列表元素作为Unicode字符串来检查。
    - `-dictionary` 
列表元素在比较时使用字典关系（查看lsort以获取详细描述），只有当-sorted可选项指定时才有有效。
    - `-integer`  
列表中的元素被当作整数来处理。
    - `-nocase`  
忽略大小写。与 `-dictionary`、`-integer`和`-real`搭配无效。
    - `-real`  
列表中的元素被当作浮点数来处理。

- **列表排序可选项**  
这个可选项指定了列表如何排序，只有指定了`-sorted`时才有效。如果给出了多个可选项，那么最后给出的可选项有效。
    - `-decreasing`  
列表元素为降序排列，只有指定了-sorted时才有效。
    - `-increasing`  
列表元素为升序排列，只有指定了-sorted时才有效。

- **内嵌列表可选项**  
这些可选项被用来寻找子列表，可以与任何其它可选项一起使用。
    - `-index indexList`  
这个可选项寻找内嵌的列表，indexList变元必须给出一个索引列表（与lindex和lset类似），indexList给出的索引列表在list中必须存在对应的索引，否则会出错。
    - `-subindices`  
这个可选项只返回寻找到的子列表元素，必须和-index配合使用。


#### 2.2.11 将多个列表合并成一个列表
`语法 : concat ?arg arg ...?`  
这个命令把每个参量用空格连接在一起，参量开头和结尾的空白符被去掉，如果参量是列表，效果就是把所有的参量连接成一个单独的列表，可以有任意个参量，如果不提供参量，结果返回一个空字符串。

示例
```
concat将连接多个列表（如下：
concat a b {c d e} {f {g h}}
将返回结果“a b c d e f {g h}”） 也可以全部都是非列表元素：
concat " a b {c   " d "  e} f"
将返回结果 “a b {c d e} f”。
```
这个命令不会去掉参数中间的空白：
```
concat "a   b   c" { d e f }
将返回结果“a b c d e f” 。
```

### 2.3 数组
#### 2.3.1 创建数组
```
set ArrayName(Index) value
```
```
#!/usr/bin/tclsh
set   marks(english)  80
puts  $marks(english)
set  marks(mathematics)  90
puts  $marks(mathematics)

array set arrName { index1 value1 index2 value2 ...}
```

#### 2.3.2 数组大小
```
[array  size  variablename]
```
eg : 
```
#!/usr/bin/tclsh

set languages(0) Tcl
set languages(1) "C Language"
puts  [array size languages]
```

#### 2.3.3 数组迭代
```
#!/usr/bin/tclsh
set languages(0) Tcl
set languages(1) "C Language"
for { set index 0 }  { $index < [array size languages] }  { incr index } {
      puts "languages($index) : $languages($index)"
}
```

#### 2.3.4 关联数组
关联数组有一个索引（key）但不一定是整数。
```
#!/usr/bin/tclsh
set personA(Name) "Dave"
set personA(Age) 14
puts  $personA(Name)
puts  $personA(Age)
```

#### 2.3.5 关联数组的索引
```
[array names variablename]
```
```
#!/usr/bin/tclsh
set personA(Name) "Dave"
set personA(Age) 14
puts [array names personA]
Age Name
```

#### 2.3.6 关联数组的迭代
```
#!/usr/bin/tclsh

set personA(Name) "Dave"
set personA(Age) 14
foreach index [array names personA] {
   puts "personA($index): $personA($index)"
}
```

### 2.4 字典
#### 2.4.1 创建字典
```
dict set dictname key value
# or 
dict create dictname key1 value1 key2 value2 .. keyn valuen
```
```
dict set colours  colour2 green
set colours [dict create colour1 "black" colour2 "white"]
```

#### 2.4.2 字典大小
```
[dict size dictname]
```

#### 2.4.3 字典的键值
```
[dict get $dictname $keyname]
```

#### 2.4.4 字典的所有键
```
[dict keys  $dictname]
```

#### 2.4.5 字典的所有值
```
[dict values $dictname]
```

#### 2.4.6 键存在于字典
```
[dict exists $dictname $keyname]
```

#### 2.4.7 字典迭代
```
set colours [dict create colour1 "black" colour2 "white"]
foreach item [dict keys $colours] {
    set value [dict get $colours $item]
    puts $value
}
```

### 2.5 句柄
TCL句柄通常用于表示文件和图形对象。
```
set  myfile  [open "filename" r] 
```

## 3. Tcl运算符
### 3.1 算术运算符
假设变量A=10，变量B=20

运算符|描述|实例
:---:|:----|:---
`+`	|两个操作数相加	|A + B = 30
`-`	|第一个操作数减去第二个操作数	|A - B = -10
`*`	|两个操作数相乘	|A * B = 200
`/`	|除法分子通过去分母	|B / A = 2
`%`	|模运算及整数除法后的余数	|B % A = 0

```
#!/usr/bin/tclsh
set a 21
set b 10
set c [expr $a + $b]
puts "Line 1 - Value of c is $c\n"
set c [expr $a - $b]
puts "Line 2 - Value of c is $c\n"
set c [expr $a * $b]
puts "Line 3 - Value of c is $c\n"
set c [expr $a / $b]
puts "Line 4 - Value of c is $c\n"
set c [expr $a % $b]
puts "Line 5 - Value of c is $c\n"
```

### 3.2 关系运算符
假设变量A=10，变量B=20


运算符|描述|实例
:----:|:----|:---
`==`	|检查两个操作数的值是否相等，如果是的话那么条件为真。	|(A == B) 不为 true.
`!=`	|检查两个操作数的值是否相等，如果值不相等，则条件为真。	|(A != B) 为 true.
`>`	|检查左边的操作数的值是否大于右操作数的值，如果是的话那么条件为真。	|(A > B) 不为  true.
`<`	|检查左边的操作数的值是否小于右操作数的值，如果是的话那么条件为真。	|(A < B) 为 true.
`>=`	|检查左边的操作数的值是否大于或等于右操作数的值，如果是的话那么条件为真。|	(A >= B) 不为 true.
`<=`	|检查左边的操作数的值是否小于或等于右操作数的值，如果是的话那么条件为真。|	(A <= B) 为 true.

```
#!/usr/bin/tclsh
set a 21
set b 10
if { $a == $b } {
	puts "Line 1 - a is equal to b\n"
} else {
	puts "Line 1 - a is not equal to b\n" 
}
if { $a < $b } {
	puts "Line 2 - a is less than b\n"
} else {
	puts "Line 2 - a is not less than b\n"
}
if { $a > $b } {
	puts "Line 3 - a is greater than b\n"
} else {
	puts "Line 3 - a is not greater than b\n"
}
```

### 3.3 逻辑运算符
假设变量A=1和变量B=0

运算符|描述|实例  
:----:|:----|:---
`&&`	|所谓逻辑与操作。如果两个操作数都非零，则条件变为真。	|`(A && B) `为 false.
`\|\|`	|所谓的逻辑或操作。如果任何两个操作数是非零，则条件变为真。	| `(A \|\| B)` 为 true.
`!`	|所谓逻辑非运算符。使用反转操作数的逻辑状态。如果条件为真，那么逻辑非运算符为假。	 | `!(A && B)` 为 true.

### 3.4 位运算符

运算符|描述|实例
:----:|:----|:---
`&`	|二进制和操作符副本位的结果，如果它存在于两个操作数。	|`(A & B) = 12`, 也就是 0000 1100
`|`	|二进制或操作拷贝位，如果它存在一个操作数中。	|`(A | B) = 61`, 也就是 0011 1101
`^`	|二进制异或操作符的副本，如果它被设置在一个操作数而不是两个比特。|	`(A ^ B) = 49`, 也就是 0011 0001
`<<`	|二进制左移位运算符。左边的操作数的值向左移动由右操作数指定的位数。	| `A << 2 = 240` 也就是 1111 0000
`>>`	|二进制向右移位运算符。左边的操作数的值由右操作数指定的位数向右移动。	| `A >> 2 = 15` 也就是 0000 1111

```
#!/usr/bin/tclsh
set a 60  ;# 60 = 0011 1100   
set b 13  ;# 13 = 0000 1101 
set c [expr $a & $b] ;# 12 = 0000 1100 
```

### 3.5 三元运算符

运算符|描述|实例
:----:|:----|:---
? :	|条件表达式	|条件为真 ? X : 否则Y

```
#!/usr/bin/tclsh
set a 10;
set b [expr $a == 1 ? 20: 30]
puts "Value of b is $b\n"
set b [expr $a == 10 ? 20: 30]
puts "Value of b is $b\n"
```

### 3.6 运算符优先级

分类|操作符|关联
:----:|:----|:---
Unary	|+ -	|Right to left
Multiplicative	|* / %	|Left to right
Additive	|+ -	|Left to right
Shift	|<< >>|	Left to right
Relational	|< <= > >=|	Left to right
Equality	|== !=	|Left to right
Bitwise AND	| &	 |Left to right
Bitwise XOR	|^	|Left to right
Bitwise OR	| \|	|Left to right
Logical AND	|&&	 |Left to right
Logical OR	| \|\|	|Left to right
Ternary	    |?:	    |Right to left



 
## 4. Tcl语句
### 4.1 条件语句
```
if { boolean_expression } {
   # statement(s) will execute if the boolean expression is true
}
```
```
if {boolean_expression} {
  # statement(s) will execute if the boolean expression is true 
} else {
  # statement(s) will execute if the boolean expression is false
}
```
```
if {boolean_expression 1} {
   # Executes when the boolean expression 1 is true
} elseif {boolean_expression 2} {
   # Executes when the boolean expression 2 is true 
} elseif {boolean_expression 3} {
   # Executes when the boolean expression 3 is true 
} else {
   # executes when the none of the above condition is true 
}
```
```
switch switchingString {
   matchString1 {
      body1
   }
   matchString2 {
      body2
   }
...
   matchStringn {
      bodyn
   }
}
```

### 4.2 循环语句
```
while {condition} {
   statement(s)
}
```
```
for {initialization} {condition} {increment} {
   statement(s);
}
```
eg :
``` 
for { set a 10}  {$a < 20} {incr a} {
   puts "value of a: $a"
}
```

eg :
``` 
#!/usr/bin/tclsh

set j 0;
for {set i 2} {$i<100} {incr i} {
   for {set j 2} {$j <= [expr $i/$j] } {incr j} {
      if {  [expr $i%$j] == 0 } {
         break
        } 
     }
   if {$j >[expr $i/$j] } {
      puts "$i is prime"
    }
}
```

**break**  
**continue**
 
## 5. Tcl函数和模块
### 5.1 过程
#### 5.1.1 过程的基本语法
用于避免相同的代码被重复在多个位置。程序相当于许多编程语言中使用的功能。
```
proc procedureName {arguments} {
   body
}
```

#### 5.1.2 过程的参数
- **多个参数**：
```
proc add {a b} {
   return [expr $a+$b]
}
puts [add 10 30]
```

- **可变参数**：
```
proc avg {numbers} {
    set sum 0
    foreach number $numbers {
      set sum  [expr $sum + $number]
	}
    set average [expr $sum / [llength $numbers]]
    return $average
}
puts [avg {70 80 50 60}]
puts [avg {70 80 50 }]
```

- **默认参数**：
```
proc add {a {b 100} } {
   return [expr $a+$b]
}
puts [add 10 30]
puts [add 10]
```

#### 5.1.3 过程的递归
```
proc factorial {number} {
   if {$number <= 1} {
      return 1
    } 
   return [expr $number * [factorial [expr $number - 1]]]
}
puts [factorial 3]
puts [factorial 5]
```

### 5.2 包
### 5.3 命名空间

 
## 6. Tcl文件I/O
### 6.1 打开和关闭文件
#### 6.1.1 打开文件
```
open  fileName  accessMode
```
**accessMode**: 

模式|描述
:---:|:----
`r`	|打开一个现有的文本文件读取并且文件必须存在。这是没有指定accessMode时使用的默认模式。
`w`	|打开用于写入的文本文件中，如果它不存在，则一个新文件创建，其他现有的文件将被截断。
`a`	|打开写在追加模式，文件必须存在一个文本文件。在这里，程序将开始追加到现有的文件内容的内容。
`r+`	|打开用于读取和写入两种的文本文件。文件必须已经存在。
`w+`	|打开用于读取和写入两种的文本文件。如果它存在首先截断文件为零长度，否则创建该文件。
`a+`	|打开用于读取和写入两种的文本文件。如果它不存在，创建该文件。读数将从头开始，但写只能追加。

#### 6.1.2 关闭文件
当程序完成使用该文件已被打开的一个程序中的任何文件都必须关闭。在大多数情况下，文件不需要被明确地关闭;它们会自动关闭，当文件对象会自动终止。
```
close fileName
```

### 6.2 写入和读取文件
#### 6.2.1 写入文件
puts命令用于写入一个打开的文件。
```
puts $filename "text to write"
```

eg:
```
set  fp  [open "input.txt" w+]
puts  $fp  "test"
close  $fp
```

#### 6.2.2 读取文件
```
set  file_data  [read  $fp]
set fp [open "input.txt" w+]
puts $fp "test"
close $fp
set fp [open "input.txt" r]
set file_data [read $fp]
puts $file_data
close $fp
```

- **一行一行读取**： 

```
set fp [open "input.txt" w+]
puts $fp "test\ntest"
close $fp
set fp [open "input.txt" r]

while { [gets $fp data] >= 0 } {
   puts $data
}
close $fp
```

`gets    fileId    ?varName? `  
读 fileId 标识的文件的下一行，忽略换行符。  
如果命令中有varName就把该行赋给它，并返回该行的字符数（文件尾返回-1）    
如果没有 varName参数，返回文件的下一行作为命令结果（如果到了文件尾，就返回空字符串）。  

- **Tcl的输入和输出**：  

```  
puts "$var" ；#输出变量var的值
puts {$var}  ；#输出大括号中的内容
puts " " 会把" "中的变量替换之后再输出，而puts {} 则是把{}中的内容原封不动的输出。使用puts进行输出，默认是换行的，如果希望输出后不换行，需要使用-nonewline参数进行设置。puts -nonewline "input x = "。
```

在Tcl中先执行flush stdout，然后使用gets stdin来读取键盘的输入：
```
flush stdout； # 输出标准输出stdout内容，清空输出缓冲区
set x [gets stdin]；#把输入的数字赋值给变量x。
```


 
## 7. Tcl错误信息
### 7.1 Error语法

```
error message info code
```
`message`是错误信息，`info`是在全局变量`errorInfo`中设置，`code`是在errorCode设置的全局变量。


### 7.2 Catch语法

```
catch script resultVarName
```
脚本是要执行的代码，`resultVarName`是可变保存错误或结果。如果没有错误， catch命令返回0；如果有一个错误，返回1。

```
proc Div {a b} {
    if {$b == 0} {
     error "Error generated by error" "Info String for error" 401
    } else {
      return [expr $a/$b]
    }
}
if {[catch {puts "Result = [Div 10 0]"} errmsg]} {
   puts "ErrorMsg: $errmsg"
   puts "ErrorCode: $errorCode"
   puts "ErrorInfo:\n$errorInfo\n"
}
if {[catch {puts "Result = [Div 10 2]"} errmsg]} {
   puts "ErrorMsg: $errmsg"
   puts "ErrorCode: $errorCode"
   puts "ErrorInfo:\n$errorInfo\n"
}
```
 
## 8. Tcl内置函数
- 列表处理函数。
- 字符串处理函数。
- 数组处理函数。
- 字典处理函数。
- 文件I/O处理函数。
- 命名空间和包处理函数。
- 数学处理函数。
- 操作系统处理函数。

### 8.1 系统函数
- `clock` - 秒函数返回当前时间以秒为单位。
- `clock` - 格式化函数格式化秒到的日期和时间。
- `clock` - 扫描函数扫描输入字符串，并将其转换为秒。
- `open` - 函数用于打开一个文件。
- `exec` - 函数用于执行一个系统命令。
- `close` - 函数用于关闭一个文件。

 
## 9. Tcl正则表达式
### 9.1 规则

SN|规则|描述
:---:|:----|:---
1	|`x`	|精确匹配。
2	|`[a-z]`	|从任何小写字母 a-z.
3	|`.`	|任何字符。
4	|`^`	|开始字符串匹配
5	|`$`	|结尾字符串匹配
6	|`\^`	|间隙序列匹配特殊字符^。同样，可以使用其它字符。
7	|`()`	|添加上述序列内括号使正则表达式。
8	|`x*`	|应该匹配0或多次出现在x前。
9	|`x+`	|应该匹配1次或多次出现在x的前面。
10	|`[a-z]?`	|应该匹配0或1在发生x之前。
11	|`{digit}`	|完全匹配位数的正则表达式之前出现。数字包含0-9。
12	|`{digit,}`	|匹配前面的正则表达式的3个或更多的数字出现。数字包含0-9。
13	|`{digit1,digit2}`	|发生匹配digit1和digit2 在正则表达式以前的事件之间的范围内。

### 9.2 语法
正则表达式的语法如下：
```
regexp   optionalSwitches  patterns  searchString  fullMatch  subMatch1 ...  subMatchn
```
- 正则表达式是命令。
- 模式是如前面提到的规则。
- 搜索字符串是其进行的正则表达式的实际字符串。
- 精确匹配任何可变持有的正则表达式匹配结果的结果。 
- Submatch1到SubMatchn是持有次级匹配模式的结果可选的子变量。

### 9.3 选择正则表达式的命令
选择在Tcl中提供的列表是，
- `-nocase` : 用于忽略大小写。
- `-indices` : 匹配的子模式，而不是匹配的字符存储的位置。
- `-line` : 新行敏感匹配。换行后忽略字符。
- `-start index` : 搜索模式开始设置偏移
- `--`: 标志着开关的结束



## 10. 自查方法
### 10.1 获得环境变量
```
set card_type $::env(INS_CARD_TYPE)
```

### 10.2 info

序号|命令|描述
:---:|:----|:---
1	|`info commands ?pattern?`	|返回匹配的命令列表
2	|`info exists varName`	|变量存在返回1，否则返回0
3	|`info globals?pattern?`	|返回全局变量列表
4	|`info locals? pattern?`	|返回局部变量列表
5	|`info procs?pattern?`	|返回过程列表
6	|`info vars?pattern?`	|返回变量列表

>>[http://blog.sina.com.cn/s/blog_4b3c1f950102dz9s.html](http://blog.sina.com.cn/s/blog_4b3c1f950102dz9s.html)

```
[file dirname [info script]
info script ?filename? returns or sets the name of the file containing the script currently being evaluated.
```

- `fconfigure` - Set and get options on a channel

- `interp alias` : creates and manages aliases, both within an interpreter (great for shortcuts) and between interpreters (vital for safe tcl)  
- `Command Aliases`：Command Aliases（命令别名）是在一个解释器中的命令，并且此命令可以在其他解释器中被调用。   
创建命令别名的方法： `interp alias slave cmd1 target cmd2 arg1 arg2 ... `   
在解释器slave中创建cmd1这个命令别名，从而当slave中的cmd1被调用的时候,解释器target中的cmd2就被调用。  
eg, `interp alias slave exit {} interp delete slave`  
`{}`表示当前解释器空间，在slave中创建一个命令，命令别名`exit`,当在slave中调用`exit`时，就在当前解释空间中调用`interp delete slave`来删除slave解释器。



## 11. tclsh
```
tclsh - Simple shell containing Tcl interpreter
SYNOPSIS
tclsh ?-encoding name? ?fileName arg arg ...?

DESCRIPTION
Tclsh is a shell-like application that reads Tcl commands from its
standard input or from a file and evaluates them. If invoked with no
arguments then it runs interactively, reading Tcl commands from 
standard input and printing command results and error messages to 
standard output. It runs until the exit command is invoked or until it 
reaches end-of-file on its standard input. If there exists a file .
tclshrc (or tclshrc.tcl on the Windows platforms) in the home directory
of the user, interactive tclsh evaluates the file as a Tcl script just 
before reading the first command from standard input.

SCRIPT FILES
If tclsh is invoked with arguments then the first few arguments specify
the name of a script file, and, optionally, the encoding of the text 
data stored in that script file. Any additional arguments are made 
available to the script as variables (see below). Instead of reading 
commands from standard input tclsh will read Tcl commands from the 
named file; tclsh will exit when it reaches the end of the file. The 
end of the file may be marked either by the physical end of the medium, 
or by the character, “\032” (“\u001a”, control-Z). If this character is 
present in the file, the tclsh application will read text up to but not 
including the character. An application that requires this character in 
the file may safely encode it as “\032”, “\x1a”, or “\u001a”; or may 
generate it by use of commands such as format or binary. There is no 
automatic evaluation of .tclshrc when the name of a script file is 
presented on the tclsh command line, but the script file can always source 
it if desired.

If you create a Tcl script in a file whose first line is

#!/usr/local/bin/tclsh

then you can invoke the script file directly from your shell if you mark 
the file as executable. This assumes that tclsh has been installed in the 
default location in /usr/local/bin; if it is installed somewhere else then 
you will have to modify the above line to match. Many UNIX systems do not 
allow the #! line to exceed about 30 characters in length, so be sure that 
the tclsh executable can be accessed with a short file name.

An even better approach is to start your script files with the following 
three lines: 

#!/bin/sh
# the next line restarts using tclsh \
exec tclsh "$0" ${1+"$@"}

This approach has three advantages over the approach in the previous 
paragraph. 
•	First, the location of the tclsh binary does not have to be hard-wired 
into the script: it can be anywhere in your shell search path. 

•	Second, it gets around the 30-character file name limit in the 
previous approach. 

•	Third, this approach will work even if tclsh is itself a shell script 
(this is done on some systems in order to handle multiple architectures or 
operating systems: the tclsh script selects one of several binaries to 
run). 
The three lines cause both sh and tclsh to process the script, but the 
exec is only executed by sh. sh processes the script first; it treats the 
second line as a comment and executes the third line. The exec statement 
cause the shell to stop processing and instead to start up tclsh to 
reprocess the entire script. When tclsh starts up, it treats all three 
lines as comments, since the backslash at the end of the second line 
causes the third line to be treated as part of the comment on the second 
line.
You should note that it is also common practice to install tclsh with its 
version number as part of the name. This has the advantage of allowing 
multiple versions of Tcl to exist on the same system at once, but also the 
disadvantage of making it harder to write scripts that start up uniformly 
across different versions of Tcl.

VARIABLES
Tclsh sets the following Tcl variables:
argc
Contains a count of the number of arg arguments (0 if none), not including 
the name of the script file.
argv
Contains a Tcl list whose elements are the arg arguments, in order, or an 
empty string if there are no arg arguments.
argv0
Contains fileName if it was specified. Otherwise, contains the name by 
which tclsh was invoked.
tcl_interactive
Contains 1 if tclsh is running interactively (no fileName was specified 
and standard input is a terminal-like device), 0 otherwise.
```


变量名称|功能描述
:---:|:----
`argc`	|指命令行参数的个数。
`argv` 	|指包含命令行参数的列表。 
`argv0`  	|是指被解释的文件或由调用脚本的名称的文件名。
`env`  	|用于表示是环境变量数组元素。
`errorCode` 	|为最后的Tcl错误的错误代码。
`errorInfo` 	|为最后Tcl错误的堆栈跟踪信息。
`tcl_interactive` 	|分别将其设置为1和0交互和非交互模式之间切换。
`tcl_library` 	|用于设置的标准Tcl库的位置。 
`tcl_pkgPath` 	|提供一般都安装包的目录列表。
`tcl_patchLevel` 	|指的是Tcl解释目前的补丁级别。 
`tcl_platform`  	|用于表示使用对象，包括byteOrder, machine, osVersion平台和操作系统数组元素。
`tcl_precision`  	|指的是精度，即位数转换为浮点数时，字符串保留。默认值是12。
`tcl_prompt1` 	|指的是主提示符。
`tcl_prompt2` 	|指无效的命令二次提示。
`tcl_rcFileName` 	|为用户提供了具体的启动文件。 
`tcl_traceCompile` 	|用于控制字节码编译的跟踪。用0表示无输出，1为概要和2为详细。
`tcl_traceExec` 	|用于控制执行的字节码的跟踪。用0表示无输出，1为概要和2为详细。
`tcl_version`  	|返回Tcl解释器的最新版本。


```
That's for compatibility with the Bourne shell. The Bourne shell was an 
old shell that was first released with Unix version 7 in 1979 and was 
still common until the mid 90s as /bin/sh on most commercial Unices. 

It is the ancestor of most Bourne-like shells like ksh, bash or zsh. 

It had a few awkward features many of which have been fixed in ksh and the 
other shells and the new standard specification of sh, one of which is 
this:

With the Bourne shell (at least those variants where it has not been fixed: 
"$@" expands to one empty argument if the list of positional parameters is 
empty ($# == 0) instead of no argument at all.

${var+something} expands to "something" unless $var is unset. It is 
clearly documented in all shells but hard to find in the bash 
documentation as you need to pay attention to this sentence:

When not performing substring expansion, using the forms documented below, 
bash tests for a parameter that is unset or null. Omitting the colon 
results in a test only for a parameter that is unset.


So ${1+"$@"} expands to "$@" only if $1 is set ($# > 0) which works around 
that limitation of the Bourne shell.

Note that the Bourne shell is the only shell with that problem. Modern shs 
(that is sh conforming to the POSIX specification of sh (which the Bourne 
shell is not)) don't have that issue. So you only need that if you need 
your code to work on very old systems where /bin/sh might be a Bourne 
shell instead of a standard shell (note that POSIX doesn't specify the 
location of the standard sh, so for instance on Solaris before Solaris 
11, /bin/sh was still a Bourne shell (though did not have that particular 
issue) while the normal/standard sh was in another location (/usr/xpg4/bin/sh)).
```


## 12. tclreadline
```
::tclreadline::Loop [historyfile ]
```
```
enter the tclreadline main loop. This command is typically called from the 
startup resource file (something .tclshrc, depending on the interpreter 
you use, see the file `sample.tclshrc'). The main loop sets up some 
completion characteristics as variable -- try something like "puts 
$b<TAB>" -- and command completion -- try "puts [in<TAB>". If the optional 
argument historyfile is given, this file will be used for reading and 
writing the command history instead of the default .tclsh-history . 
::tclreadline::Loop will normally not return. If you want to write your 
own main loop and/or own custom completers, it is probably a good idea to 
start with tclreadline::Loop (see the file tclreadlineSetup.tcl).
```

```
::tclreadline::prompt1
```
```
a proc which is called by ::tclreadline::Loop and returns a string which 
will be displayed as the primary prompt. This prompt will be something 
like "[info nameofexecutable] [[pwd]]" possibly fancy colored. The default 
proc is defined on entering the ::tclreadline::Loop, if it is not already 
defined. So: If you define your own proc ::tclreadline::prompt1 before 
entering ::tclreadline::Loop, this proc is called each time the prompt is 
to be displayed. Example:
```
```
     package require tclreadline 
     namespace eval tclreadline { 
         proc prompt1 {} { 
             return "[clock format [clock seconds]]> " 
         } 
     } 
     ::tclreadline::Loop 
``` 


### 参考资料
[https://www.yiibai.com/tcl/tcl_ternary_operator.html](https://www.yiibai.com/tcl/tcl_ternary_operator.html)
