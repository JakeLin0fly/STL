---
title: STL源码剖析-迭代器-iterators
date: 2020-04-10
categories:
  - 笔记
  - STL
tags:
  - C++
  - STL
description: 迭代器扮演容器与算法之间的桥梁，是所谓的 “泛型指针”。迭代器统一访问不同容器时的访问方式。通过迭代器可以将容器和通用算法结合在一起，同一套算法代码可以利用在完全不同的容器中。
; sticky: 
---

迭代器(iterator)是一种抽象的设计理念，通过迭代器可以在不了解容器内部原理的情况下遍历容器。除此之外，STL中迭代器一个最重要的作用就是作为容器(vector,list等)与STL算法的粘结剂，只要容器提供迭代器的接口，同一套算法代码可以利用在完全不同的容器中，这是抽象思想的经典应用。

从实现的角度来看，迭代器是一种将 `operator*`，`operator->`，`operator++`，`operator--` 等指针相关操作予以重载的 class template。 所有 STL 容器都附带有自己专属的迭代器。 native pointer 也是一种迭代器。

为什么每一种 STL 容器都提供有专属迭代器？主要是暴露太多细节，所以把迭代器的开发工作交给容器去完成，这样所有实现细节可以得到封装，不被使用者看到。

## Traits

<img src="https://i.loli.net/2020/08/10/MQOyeBwmSgTt8j9.png" alt="iteratortraits.png" style="zoom: 80%;" />

## 迭代器所指对象类型(associated types)

迭代器所指的对象类型有五种：

```c++
tempalte<typename I>
struct iterator_traits
{
    typedef typename I::iterator_category  iterator_category;	//迭代器的分类
    typedef typename I::value_type  value_type; 				//迭代器所指对象的型别
    typedef typename I::difference_type  difference_type;		//两迭代器之间的距离
    typedef typename I::pointer  pointer;						//指针，指向迭代器所指之物
    typedef typename I::reference  reference;					//类似引用类型，允许改变“所指对象的值”
};
```

## 迭代器的分类

1. Input Iterator ：能从所指向元素读取的迭代器（**只读**）。仅保证单趟算法的合法性。
2. Output Iterator ：能写入所指元素的迭代器（**只写**）。
3. Forward Iterator ：一种能从所指向元素读取数据的迭代器 。
4. Bidirectional Iterator：能**双向移动**（即自增与自减）的迭代器 。
5. Random Access Iterator ：**随机读写**，能在常数时间内移动到指向任何元素的双向迭代器。

<img src="https://i.loli.net/2020/08/10/NBfOK4GX572eFYL.png" alt="迭代器的分类于从属关系.png" style="zoom:67%;" />

## 总结

traits 本质是什么？多一层间接性，换来灵活性。

iterator_traits 负责萃取迭代器的特性，__type_traits 负责萃取类型的特性。

STL的中心思想是将数据容器与算法分开，彼此独立设计。迭代器便作为连接两者的“桥梁”，统一访问不同容器时的访问方式，并将实际细节封装，不被使用者看到。