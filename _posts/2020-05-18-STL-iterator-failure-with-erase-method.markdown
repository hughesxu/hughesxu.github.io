---
title:  "STL 容器erase() 方法与迭代器失效问题"
date:   2020-05-09 17:24:22 +0800
categories: [Language, C++]
tags: stl
---

+ **背景**  
&emsp;对于C++ STL容器，当向容器中添加元素或从容器中删除元素时，要特别注意迭代器失效问题。  

> 一个失效的迭代器将不再表示任何元素。使用失效的迭代器将产生严重的错误，可类比未初始化的指针。  

举例说明：将关联容器`map<int, int>poly` 中`value`为0对应的的`pair<key, value>`从关联容器中删除。  
很容易想到的写法：  
```
    for (map<int, int>::iterator iit = poly.begin(); iit != poly.end(); iit++) {
        int key   = (*iit).first;
        int value = (*iit).second;

        if (0 == value) {
            poly.erase(iit);
        }
    }
```  
&emsp;这种方法看似正确，但仔细考虑：当利用`erase()`操作删除当前迭代器所指向的元素时，该元素位置已被删除，无法利用它获取下一个元素的位置，造成了典型的迭代器失效问题。  
&emsp;对于关联容器而言，`erase()`操作有几种版本：

`erase()`      |操作说明|返回类型
:-----------:|:-------|:-------
`c.erase(k)`   |从`c`中删除每个关键字为`k`的元素| 返回一个`size_type`值，指出删除的元素的数量
`c.erase(p)`   |从`c`中删除迭代器`p`指定的元素（`p`必须指向真实元素，不能是`c.end()`）| 返回一个指向`p`之后元素的迭代器，若`p`指向`c`中的尾元素，则返回`c.end()`
`c.erase(b, e)`|删除迭代器对`b`和`e`所表示的范围中的元素|返回`e`


+ **程序修改**  
&emsp;可以利用`c.erase(p)`操作返回指向`p`之后下一个元素的特性，对上面的程序进行修改：
```
    for (map<int, int>::iterator iit = poly.begin(); iit != poly.end(); ) {
        int key   = (*iit).first;
        int value = (*iit).second;

        if (0 == value) {
            iit = poly.erase(iit);
        } else {
	    ++iit;	
	}
    }
```  
分析：  
1. 当`value==0`时，将当前迭代器`iit`指向的元素删除，并且将指向之后元素的迭代器重新赋值给`iit`，进入`for`的下一个循环判断；  
2. 否则，将`iit`指向下一个元素。


+ **再修改**
&emsp;查询一些资料，还可以有如下更简洁的写法：  
```
    for (map<int, int>::iterator iit = poly.begin(); iit != poly.end(); ) {
        int key   = (*iit).first;
        int value = (*iit).second;

        if (0 == value) {
            poly.erase(iit++);		// Note: Postfix++ is must
        } else {
	    ++iit;	
	}
    }
```


&emsp;关于此处前置++和后置++的区别：
 
> The prefix increment operator changes an object’s state, and returns itself in the changed form. No temporary objects required.  
```  
	template<typename IteratorT>
	IteratorT & PreIncrement( )
	{
	   ++myOwnField;
	   return (*this);
	}
``` 
 
> A postfix operator also changes the object’s state but returns the previous state of the object. It does so by creating a temporary object.
```
	template<typename IteratorT>
	IteratorT PostIncrement(IteratorT& it)
	{
	   auto copy = it;
	   ++it;
	   return copy;
	}
```  

**注意**：此处的++运算符是类运算符重载实现的，和自增运算符++有一些差别，注意区分。  

&emsp;重新分析上文修改后的程序，采用PostIncrement运算符，返回给函数的只是当前迭代器`iit`的一个副本，在返回给函数前会将`iit`指向下一个元素，因此实现了将迭代器指向下一个元素的同时，删除当前所指向元素的目的。
