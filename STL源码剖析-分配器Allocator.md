---
title: STL源码剖析-分配器Allocators
date: 2020-04-06
categories:
  - 笔记
  - STL
tags:
  - C++
  - STL
description: STL六大组件之一：分配器Allocators
; sticky: 
---

# 先谈operator new()和malloc()

 **new**：指我们在C++里通常用到的**运算符**

**operator new()**：指对new的重载形式，它是一个**函数**，并不是运算符

函数operator new中调用malloc()进行内存分配。

malloc实际内存分配得到的内存空间如下：

![malloc-new-分配得到的内存空间.png](https://i.loli.net/2020/08/04/xN2TpBZyt3eKSoP.png)



new 运算符执行过程：

1. 调用operator new分配内存（可重载）
2. 调用构造函数构造生成类对象
3. 返回相应的指针

# SGI标准的分配器 --- allocator

最重要的两个函数：`allocate` 、`deallocate` 

```cpp
//调用 operator new()
pointer allocator::allocate(size_type n, const void* = 0)

//调用 operator delete()
void allocato::deallocate(pointer p, size_type n)
```

不建议使用，效率不佳，只对C++的`::operator new` 和 `::operator delete` 做了简单的包装在STL实际使用中，并没有使用 `allocator` 这个分配器。

# SGI STL分配器 --- alloc（G2.9)

***注意：G4.9以后默认的分配器又变成了allocator；G4.9的__pool_alloc就是G2.9的alloc***

16条链表，每一条链表负责不同大小的区块。第0条负责8字节大小，以8字节大小增长

优点：减少内存分配时多余空间的分配

<img src="https://i.loli.net/2020/08/04/NJt2yKDZ7E4Xo6P.png" alt="SGI-STL使用的分配器alloc原理图.png" style="zoom:70%;" />