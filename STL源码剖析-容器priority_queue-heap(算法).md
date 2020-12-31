---
title: STL源码剖析-容器priority_queue-heap(算法)
date: 2020-08-09
categories:
  - 笔记
  - STL
tags:
  - C++
  - STL
description: heap也就是数据结构中的堆，逻辑上可以看成一颗完全二叉树。binary max heap 作为 priority_queue 的底层机制。STL提供的是max-heap。
; sticky: 
---

<img src="https://i.loli.net/2020/08/09/KF12cDuCQVkaYmT.png" alt="完全二叉树即其array表述式.png" style="zoom: 67%;" />

## heap

heap的实现就是数据结构中的**堆**（大顶堆、小顶堆）。STL提供`max-heap`。

STL中并没有把`heap`作为一种容器组件，heap的实现亦需要更低一层的容器组件，诸如list，array，vector。heap并不属于STL容器，但它是其中一个容器`priority queue`必不可少的一部分。

**heap没有迭代器，**heap的所有元素都必须遵循特别的排列规则，所以heap不提供遍历功能，也不提供迭代器。

### push_heap

> 1. 最新加入的源放在最下层叶子节点，填补从左至右第一个空闲单元
>
> 2. percolate up（上溯）：与父节点比较，调整位置
>
>    > 将新节点与父节点比较，如果其键值比父节点大，就交换父子的位置，如此一直上溯，直到不需要交换或者到根节点为止。

<img src="https://i.loli.net/2020/08/09/5Dz7RYnrlGKIkmc.png" alt="push_heap演示.png" style="zoom: 67%;" />

### pop_heap

> 1. 把原尾端节点的值拿出来放至一临时变量里，然后该位置放根节点的值（最后会被pop_back()给移除）
>
> 2. 重新构建大顶堆，实施如下调整堆：
>
>    1）**percolate down**（下溯）：从根节点开始将空洞节点（一开始是根节点）和较大子节点交换，并持续向下进行，直到到达叶节点为止。然后将已保存的原容器vector尾端节点赋给这个已到达叶层的空洞节点。
>
>    2）此时可能尚未满足次序特性，再执行一次**percolate up**（上溯）操作

***注意：pop_heap之后，最大元素只是被放置于底部容器的最尾端，尚未被取走。如果要取其值，可使用底部容器的back()函数。如果要移除它，可使用底部容器所提供的pop_back()函数。***

<img src="https://i.loli.net/2020/08/09/PXJQOjWxi9Scezq.png" alt="pop_heap演示.png" style="zoom: 67%;" />

### sort_heap

**堆排序算法**。执行此操作之后，容器vector中的元素按照从小到大的顺序排列。

<img src="https://i.loli.net/2020/08/09/fnZlGrWJxibYPo2.png" alt="sort_heap演示.png" style="zoom:67%;" />

### make_heap

构建堆实质是一个不断调整堆的过程---通过不断调整子树，使得子树满足堆的特性来使得整个树满足堆的性质。

第一个需要执行调整操作的子树的根节点是从后往前的第一个非叶结点。从此节点往前到根节点对每个子树执行调整操作，即可构建堆。

```c++
 // 将 [first,last) 排列为一个 heap。
 template <class RandomAccessIterator>
 inline void make_heap(RandomAccessIterator first, RandomAccessIterator last) {
     __make_heap(first, last, value_type(first), distance_type(first));
 }
 
 template <class RandomAccessIterator, class T, class Distance>
 void __make_heap(RandomAccessIterator first, RandomAccessIterator last, T*,
     Distance*) {
     if (last - first < ) return; // 如果長度為 0 或 1，不必重新排列。
     Distance len = last - first;
     // 找出第一个需要重排的子树头部，以 parent 标示出。由于任何叶子节点都不需执行
         // perlocate down（下沉），所以有以下计算。
         Distance parent = (len - ) / ;
 
     while (true) {
         // 重排以 parent 为首的子树。len 是为了让 __adjust_heap() 判断操作范围
         __adjust_heap(first, parent, len, T(*(first + parent)));
         if (parent == ) return; // 直至根节点，就结束。
         parent--; // 未到根节点，就将（即将重排的子树的）索引值向前一個节点
     }
 }
```

## priority_queue

`priority_queue` 是一个拥有权值观念的 queue。

默认情况下以 `vector` 为底部容器完成其所有工作，再加上 `heap` 处理规则。

priority_queue的所有元素，都不一定的进程出规则，只有queue顶端的元素（权值最高）才有机会被外界取用。故`priority_queue`不提供遍历功能，也不提供迭代器。

priority_queue 基本操作：

`priority_queue()`：调用 `make_heap()`， 使进入的元素后，始终保持一个堆。

`top()`：队顶元素。

`push()`：`push_back()`尾端插入元素，然后调用 `push_heap()` 重排堆。

`pop()`：用 `pop_heap() `将最大元素放到底部容器的最尾端，**并不是真正弹出**，再调用底部容器 vector 所提供的 `pop_back()` 弹出元素。

## 参考

[STL源码剖析——序列式容器#5 heap](https://www.shuzhiduo.com/A/E35pRNwRzv/)