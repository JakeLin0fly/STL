---
title: STL源码剖析-容器-stack-queue
date: 2020-08-09
categories:
  - 笔记
  - STL
tags:
  - C++
  - STL
description: stack是一种先进后出(First In Last Out, FILO)的数据结构。只有一个出口。queue是一种先进先出(First In First Out, FIFO)的数据结构。有两个出口，从最底端加入元素、最顶端取出元素。
; sticky: 
---

## stack

stack是一种先进后出(First In Last Out, **FILO**)的数据结构。只有一个出口。

SGI STL 默认是以**deque**作为缺省情况下的stack底层结构。也可以 **list** 作为 stack 的底层容器。

STL stack 往往不被归类为 container(容器)，而被归类为 **container adapter**。

stack所有的元素都必须符合“先进后出”的条件，只有从stack顶端对其进行新增、移除、读取操作，即stack**不允许遍历**行为，stack**不提供迭代器**。

<img src="https://i.loli.net/2020/08/09/SLDxIahQRYBuc5y.jpg" alt="stack.jpg" style="zoom: 25%;" />

stack基本操作：

`push()`：顶部入栈

`pop()`：顶部出栈

`top()`：：指向栈顶

`empty()`：判断栈空

## queue

queue是一种先进先出(First In First Out, FIFO)的数据结构。有两个出口，从最底端加入元素、最顶端取出元素。

SGI STL 默认是以 **deque** 作为缺省情况下的 queue 底部结构。也可以 **list** 作为 stack 的底层容器。

STL queue往往不被归类为 container(容器)，而被归类为 **container adapter**。

stack所有的元素都必须符合“先进先出”的条件，只有从queue尾端增加元素、头端取出元素。queue**不允许遍历**行为，queue**不提供迭代器**。

<img src="https://i.loli.net/2020/08/09/jFngpJdNtEyoqAm.jpg" alt="queue.jpg" style="zoom:25%;" />

queue基本操作：

`push()`：尾端入队

`pop()`：首端出队

`front()`：返回队首

`back()`：返回队尾