---
title: STL源码剖析-容器-deque
date: 2020-04-11
categories:
  - 笔记
  - STL
tags:
  - C++
  - STL
description: STL容器deque是一种双向开口的连续线性空间。双向开口，即可以在头尾两端进行插入和删除。deque是动态的以分段连续空间组合而成，维护整体连续的假象，提供随机存取接口。
; sticky: 
---

![](https://gitee.com/jakel-in/images/raw/master/2020-08/deque示意图.jpg)

## deque概述

* deque 是一种**双向开口**的**连续线性**空间。可以在头尾两端分别做元素的插入和删除操作。
* deque是有一段一段的定量连续空间构成，是**动态分段连续**。
* deque 和 vector 的差异：
  * deque 允许于常数时间内对两端进行元素的插入或移除操作。
  * deque 没有容量，它是动态地以分段连续空间组合而成。
* deque 采用一块所谓的 `map` 作为主控。这个 map 是一小块连续空间（默认初值大小**8个节点**），其中每个节点都是指针，指向另一段（较大的）连续线性空间，称为缓冲区。缓冲区才是deque的存储空间主体。
* deque 是分段连续空间，迭代器维持其“整体连续”的**假象**，并提供随机存取的接口。
* 一旦`map`提供的节点不足，就必须重新配置一个更大的一块`map`（**至少两倍+2**）。并将原来的数据放在**中央**，以使头尾两端的扩充能量一样大。
* 插入数据时，deque会根据插入位置与两端的距离判断向前还是向后。

<img src="https://i.loli.net/2020/08/08/nl75O1bUu9TzIQ4.png" alt="deque分段连续示意图.png" style="zoom:80%;" />

## deque 的中控器map

deque 采用一块所谓的 `map` 作为主控。这个 map 是一小块连续空间，其中每个节点都是指针，指向另一段（较大的）连续线性空间，称为缓冲区。缓冲区才是deque的存储空间主体。

### map默认初始大小为8

```c++
enum { _S_initial_map_size = 8 }; // map默认初始大小 8

// _M_initialize_map(size_t __num_elements)中
// map最少8个节点，最多“所需节点数+2”（前后各预留一个，扩容时可以用）
this->_M_impl._M_map_size = std::max((size_t)_S_initial_map_size, size_t(__num_nodes + 2));
```

### map空间重新配置

扩容一次至**至少两倍+2**。

> size_type __new_map_size = _M_map_size + max(_M_map_size, __nodes_to_add) + 2;

```c++
void _M_reserve_map_at_back (size_type __nodes_to_add = 1) { /*默认新增节点数=1*/
  if (__nodes_to_add + 1 > _M_map_size - (_M_finish._M_node - _M_map))
    _M_reallocate_map(__nodes_to_add, false);
}

void _M_reserve_map_at_front (size_type __nodes_to_add = 1) { /*默认新增节点数=1*/
  if (__nodes_to_add > size_type(_M_start._M_node - _M_map))
    _M_reallocate_map(__nodes_to_add, true);
}

template <class _Tp, class _Alloc>
void deque<_Tp,_Alloc>::_M_reallocate_map(size_type __nodes_to_add,
                                          bool __add_at_front)
{
  size_type __old_num_nodes = _M_finish._M_node - _M_start._M_node + 1;
  size_type __new_num_nodes = __old_num_nodes + __nodes_to_add;

  _Map_pointer __new_nstart;
   /* map节点总数 > 2倍新需节点数，则移动调整节点到中央 */
  if (_M_map_size > 2 * __new_num_nodes) {
    __new_nstart = _M_map + (_M_map_size - __new_num_nodes) / 2 
                     + (__add_at_front ? __nodes_to_add : 0);
    if (__new_nstart < _M_start._M_node)
      copy(_M_start._M_node, _M_finish._M_node + 1, __new_nstart);
    else
      copy_backward(_M_start._M_node, _M_finish._M_node + 1, 
                    __new_nstart + __old_num_nodes);
  }
   /* map节点总数不足2倍新需节点数，从新配置 */
  else {
    /* 重点!!!  现有map节点数 + max(现有map节点数, 新增节点数) +2  !!! */
    size_type __new_map_size = 
      _M_map_size + max(_M_map_size, __nodes_to_add) + 2;

    /* 拷贝原map节点数据到新map中央 */
    _Map_pointer __new_map = _M_allocate_map(__new_map_size);
    __new_nstart = __new_map + (__new_map_size - __new_num_nodes) / 2
                         + (__add_at_front ? __nodes_to_add : 0);
    copy(_M_start._M_node, _M_finish._M_node + 1, __new_nstart);
    _M_deallocate_map(_M_map, _M_map_size); /* 释放原map空间 */
	
    _M_map = __new_map;
    _M_map_size = __new_map_size;
  }
	/* 设置新的map的start、finish迭代器 */
  _M_start._M_set_node(__new_nstart);
  _M_finish._M_set_node(__new_nstart + __old_num_nodes - 1);
}
```



## deque迭代器

* 指向连续空间
* 能判断空间边界，在边界上移动能指向正确的下一个连线空间
* 随机存取

**_Deque_iterator**:

```c++
_Tp* _M_cur;		//此迭代器所指的缓冲区的当前元素位置
_Tp* _M_first;		//此迭代器所指的缓冲区的头
_Tp* _M_last;		//此迭代器所指的缓冲区的尾
_Map_pointer _M_node;	//指向管控中心 map
```

## deque插入数据

1. 若头部插入：`push_front`

2. 若尾部插入： `push_back`

3. 中间位置插入：

   1）判断距离哪端近

   2）移动元素

   3）插入新元素

## deque基本操作

`push_back()`：尾端插入

`push_front()`：头端插入

`pop_back()`：尾端取出

`pop_front()`：头端取出