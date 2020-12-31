---
title: STL源码剖析-容器-slist
date: 2020-04-12
categories:
  - 笔记
  - STL
tags:
  - C++
  - STL
description: slist也就是single linked list（单向链表）。
; sticky: 
---

STL list是一个双向链表（double linked list）。SGI STL提供了另一个单向链表（single linked list）也就是`slist`。

`slist`与`list`差异：

1. slist迭代器是单向的Forward Iterator，list迭代器是双向的Bidirectional Iterator。（slist迭代器没有`--`操作，因为是单向的Forward Iterator）。
2. 单向链表所耗空间更小。
3. list是**双向环形**链表，slist是**单向**链表。
4. slist不提供push_back()，仅提供**push_front()**（头插法）。

`slist`与`list`相同点：

1. 都具有“头节点”，且都不放置元素数据。
2. 进行插入、删除、接合等操作，均不会导致原迭代器失效（当然，指向被移除的那个节点的迭代器肯定会失效）。
3. `list`和`slist`都不能使用 STL 算法 `sort()` ，必须使用自己的 `sort()` 。

slist基本操作：

`push_front()`：从头部插入元素

`pop_front()`：从头部取出元素

`front()`：取头部元素

