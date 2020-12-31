---
title: STL源码剖析-容器-vector
date: 2020-04-10
categories:
  - 笔记
  - STL
tags:
  - C++
  - STL
description: STL容器vector与array非常相似，唯一区别在于空间的运用的灵活性。array是静态空间，vector是动态空间。值得注意的是vector是以两倍增长的方式进行自行扩容空间容纳新元素。
; sticky: 
---

vector 常被称为向量容器，因为该容器擅长在**尾部**插入或删除元素，在常量时间内就可以完成，时间复杂度为`O(1)`；而对于在容器**头部或者中部**插入或删除元素，则花费时间要长一些（移动元素需要耗费时间），时间复杂度为线性阶`O(n)`。

vector实现的关键在于其对**大小的控制**以及**重新配置时的数据移动效率**。

## vector

* vector 与 array 唯一区别是空间的运用的灵活性。
  * array 是**静态空间**，一旦配置了就不能改变。
  * vector是**动态空间**。
* vector 维护的是一个**连续线性空间**，所以不论其元素类型为何，普通指针都可以作为 vector 的迭代器而满足所有必要条件。
* 增加新元素时，如果超过当时的容量，则容量会扩大至**两倍**。

<img src="https://i.loli.net/2020/08/07/5KPtrc42gBWpMnz.png" alt="vector-两倍增长示意图.png" style="zoom:67%;" />

* 所谓动态增加大小，并不是在原空间之后接续新空间(因为无法保证原空间之后尚有可供配置的空间)，而是以原空间的两倍大小另外配置一块较大空间，然后将原内容拷贝过来并插入新元素，释放原空间，最后更新迭代器！！！

* 一旦引起空间重新配置，指向原 vector 的所有迭代器就都失效了。

* `at()` 函数 比 `[]` 运算符更加安全, 因为它不会让你去访问到Vector内越界的元素。

  ```c++
  #ifdef __STL_THROW_RANGE_ERRORS
    void _M_range_check(size_type __n) const {
      if (__n >= this->size())
        __stl_throw_range_error("vector");
    }
  
    reference at(size_type __n)
      { _M_range_check(__n); return (*this)[__n]; }
    const_reference at(size_type __n) const
      { _M_range_check(__n); return (*this)[__n]; }
  #endif /* __STL_THROW_RANGE_ERRORS */
  ```

## vector动态增加容量原理_M_insert_aux

**过程分析：**

> 1. 配置一块更大空间（2倍，初始为0则配置新的空间大小为1）
>
> 2. 拷贝原数据并插入新元素
>
>    1）拷贝插入位置前的数据
>
>    2）在插入位置插入新元素
>
>    3）拷贝插入位置后的数据
>
> 3. 释放原空间
>
> 4. 更新迭代器！！！

源码参考：

```c++
template <class _Tp, class _Alloc>
void 
vector<_Tp, _Alloc>::_M_insert_aux(iterator __position, const _Tp& __x)
{ /* 判断是否有备用空间 */
  if (_M_finish != _M_end_of_storage) {
    construct(_M_finish, *(_M_finish - 1));
    ++_M_finish;
    _Tp __x_copy = __x;
    copy_backward(__position, _M_finish - 2, _M_finish - 1);
    *__position = __x_copy;
  }
  else {
    const size_type __old_size = size();
    /* 扩容两倍。 注意：初始时为0，扩容后为1 */
    const size_type __len = __old_size != 0 ? 2 * __old_size : 1;
    iterator __new_start = _M_allocate(__len); /*分配新的连续空间*/
    iterator __new_finish = __new_start;
    __STL_TRY {
      /* 拷贝插入位置前的数据到新空间 [old_start, position) --> [new_start, position) */
      __new_finish = uninitialized_copy(_M_start, __position, __new_start);
      /* 插入新元素 position */
      construct(__new_finish, __x);
      ++__new_finish;
      /* 拷贝插入位置后的数据到新空间 [position, finish) -- [position+1, new_finish) */
      __new_finish = uninitialized_copy(__position, _M_finish, __new_finish);
    }
    /* 释放原空间 */
    __STL_UNWIND((destroy(__new_start,__new_finish), 
                  _M_deallocate(__new_start,__len)));
    destroy(begin(), end());
    _M_deallocate(_M_start, _M_end_of_storage - _M_start);
    /* 更新迭代器 */
    _M_start = __new_start;
    _M_finish = __new_finish;
    _M_end_of_storage = __new_start + __len;
  }
}
```

## vector基本操作

`push_back()`：插入操作(末尾)

`pop_back()`：删除操作(末尾)

`erase()`：清除某范围 `[first, last)` 元素，或删除某个位置上的元素

`insert()`：从某个位置，插入 n 个元素，元素初值为x

`clear()`：清除所有元素

`begin()`：返回第一个元素的迭代器

`end()`：返回最末元素的迭代器(译注:实指向最末元素的下一个位置)



注意：

`reserve(size_type __n)`：配置vector容量为`__n`，如果 vector 的容量已经大于或等于`__n`个元素，那么什么也不做；调用 reserve() 不会影响已存储的元素，也不会生成任何元素。

```c++
  vector<_Tp, _Alloc>& operator=(const vector<_Tp, _Alloc>& __x);
  void reserve(size_type __n) {
    if (capacity() < __n) {
      const size_type __old_size = size();
      iterator __tmp = _M_allocate_and_copy(__n, _M_start, _M_finish);
      destroy(_M_start, _M_finish);
      _M_deallocate(_M_start, _M_end_of_storage - _M_start);
      _M_start = __tmp;
      _M_finish = __tmp + __old_size;
      _M_end_of_storage = _M_start + __n;
    }
  }
```

