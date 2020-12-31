---
title: STL源码剖析-容器-list
date: 2020-04-08
categories:
  - 笔记
  - STL
tags:
  - C++
  - STL
description: STL容器list。STL list是一个双向环形链表。插入（insert）和拼接（splice）都不会造成原有的list迭代器失效（区别于vector）。
; sticky: 
---

STL list 容器，又称**双向链表容器**，即该容器的底层是以双向链表的形式实现的。这意味着，list 容器中的元素可以分散存储在内存空间里，而不是必须存储在一整块连续的内存空间中。

基于这样的存储结构，list 容器具有一些其它容器（array、vector 和 deque）所不具备的优势，即它可以在序列已知的任何位置快速**插入**或**删除**元素（时间复杂度为`O(1)`）。并且在 list 容器中移动元素，也比其它容器的效率高。

查找元素需要遍历容器链表，时间复杂度为`O(n)`。

# list节点

增加一个”哨兵“，便能符合STL对于”前闭后开“区间的要求。

<img src="https://i.loli.net/2020/08/04/vMCmnAWwHZXK39Y.png" alt="list-双向环形链表.png" style="zoom:67%;" />

# list迭代器

由于STL list是一个**双向环形链表**，迭代器必须具备前移、后移的能力，所以list提供的是 `Bidirectional Iterators` 。

list一重要性质：插入（insert）和拼接（splice）都**不会**造成原有的list迭代器失效（区别于vector）。

list迭代器部分设计：

 ***因前 `++` （或前 `--` ）返回值为引用，故允许：`++++i` 、 `----i` ；***

***后 `++` (或 `--` )则不允许这样***

```cpp
//对迭代器累计+1 也就是前进一个结点
  self& operator++() { 	// 前++ (++i) 无参数
    node = (link_type)(node->next);
    return *this;
  }
  self operator++(int) {   // 后++ (i++) 有参数，参数无意义
    self tmp = *this;
    ++*this;
    return tmp;
  }
  
//对迭代器累计-1 也就是前进一个结点
  self& operator--() { 
    node = (link_type)(node->prev);
    return *this;
  }
  self operator--(int) { 
    self tmp = *this;
    --*this
    return tmp;
  }
```

# list数据结构

| 函数         | 用法                                                         |
| ------------ | ------------------------------------------------------------ |
| back()       | 返回一个引用，指向最后一个元素                               |
| begin()      | 返回指向第一个元素的迭代器                                   |
| clear()      | 删除所有元素                                                 |
| empty()      | 如果list是空的则返回true                                     |
| end()        | 返回末尾的迭代器                                             |
| erase()      | 删除以pos指示位置的元素, 或者删除*start*和*end*之间的元素。 返回值是一个迭代器，指向最后一个被删除元素的下一个元素。 |
| front()      | 返回一个引用，指向第一个元素                                 |
| **insert()** | 插入元素val到位置pos，返回值是一个迭代器，指向被插入的元素。 或者插入num个元素val到pos之前。或者插入start到end之间的元素到pos的位置。 |
| merge()      | 合并两个list（合并到this上，必须先经过**递增排序**）         |
| pop_back()   | 删除最后一个元素                                             |
| pop_front()  | 删除第一个元素                                               |
| push_back()  | 在list的末尾添加一个元素                                     |
| push_front() | 在list的头部添加一个元素                                     |
| remove()     | 从list删除所有值为val的元素                                  |
| remove_if()  | 按指定条件删除元素                                           |
| resize()     | 改变list的大小                                               |
| reverse()    | 把list的元素倒转                                             |
| size()       | 返回list中的元素个数                                         |
| **sort()**   | 给list排序                                                   |
| splice()     | 合并两个list                                                 |
| swap()       | 交换两个list                                                 |
| unique()     | 删除list中**数值相同的连续**元素（保留连续的值相同的第一个元素） |

# 注意

`list` 不能使用 STL 算法 `sort()` ，必须使用自己的 `sort()` 。

因为STL算法sort()只能接受 `RamdonAccessIterator` 。

