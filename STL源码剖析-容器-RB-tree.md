---
title: STL源码剖析-容器-RB-tree
date: 2020-09-09
categories:
  - 笔记
  - STL
tags:
  - C++
  - STL
description: 红黑树本质是一个二叉搜索树，每个节点非黑即红，再加上一些特性，变成平衡二叉搜索树（平衡性比AVL-tree弱一些）。红黑树的插入、删除、查找操作的时间复杂度是 O(logN)。
; sticky: 
---

**在线演示：**[红黑树在线操作演示](https://www.cs.usfca.edu/~galles/visualization/RedBlack.html)

## 红黑树概念

**RB Tree**，全称是Red-Black Tree，又称为“红黑树”。红黑树本质上是一种**二叉查找树**，但它在二叉查找树的基础上额外添加了一个标记（颜色），同时具有一定的规则。这些规则使红黑树保证了一种平衡，插入、删除、查找的最坏时间复杂度都为 `O(logn)`。

## 红黑树的性质(重点) 

1. 每个节点不是<font color=red>**红色**</font>就是**黑色**
2. **根结点**永远都是**黑色**
3. 所有**叶节点**都是**黑色**（注意这里说叶子节点其实是上图中的 `NIL` 节点）
4. 父子节点不同为<font color=red>**红色**</font>
5. 从任一节点到其子树中每个叶子节点的r任一路径都包含**相同数量**的**黑色**节点

**注意**：

> 性质(3)中的叶子节点，是只为空(NIL或null)的节点。
>
> 性质(5)，确保没有一条路径会比其他路径长出俩倍。因而，红黑树是相对是接近平衡的二叉树。

![红黑树.jpg](https://i.loli.net/2020/08/10/vfWSnHY2ZmdX3D6.jpg)

## 树的旋转操作

### 左旋

简略记忆：<font color=red size=5>**“右子变老子”**</font>，**左旋中的“左”，意味着“被旋转的节点将变成一个左节点”**。如图所示

1. 旋转点右孩子的左子树(B)变成旋转点的右子树
2. 旋转点右孩子顶替旋转点位置
3. 修改旋转点与右孩子关系

<img src="https://i.loli.net/2020/09/07/7JhabscBD6kIler.png" alt="左旋.png" style="zoom: 80%;" />

{% fold 点击显示/隐藏-左旋代码%}

```cpp
/*
 * 左旋 右子节点变为父节点（原父节点变为右子节点的左节点）
*/
inline void 
_Rb_tree_rotate_left(_Rb_tree_node_base* __x, _Rb_tree_node_base*& __root)
{
  _Rb_tree_node_base* __y = __x->_M_right;
  __x->_M_right = __y->_M_left; // 右子节点
    /* Step 1: right_child 的左子树 B 变成 p 的右子树 */
  if (__y->_M_left !=0)
    __y->_M_left->_M_parent = __x; // B 的父节点修改

    /* Step 2: right_child 顶替 p 的位置 */
  __y->_M_parent = __x->_M_parent;
  if (__x == __root)// right_child 的父节点修改 
    __root = __y;
  else if (__x == __x->_M_parent->_M_left) // p 为父节点的左孩子
    __x->_M_parent->_M_left = __y; 
  else                                     // p 为父节点的右孩子
    __x->_M_parent->_M_right = __y;

  /* Step 3: 修改 p 和 right_child 关系 */
  __y->_M_left = __x;
  __x->_M_parent = __y;
}
```

{% endfold %}

### 右旋

简略记忆：<font color=red size=5>**“左子变老子”**</font>，**右旋中的“右”，意味着“被旋转的节点将变成一个右节点”**。如图所示

1. 旋转点左孩子的右子树(B)变成旋转点的左子树
2. 旋转点左孩子顶替旋转点位置
3. 修改旋转点与右孩子关系

<img src="https://i.loli.net/2020/09/07/VOxf3ajq8hH7ETd.png" alt="右旋.png" style="zoom: 80%;" />

{% fold 点击显示/隐藏-右旋代码%}

```cpp
/*
 * 右旋 左子节点变为父节点（原父节点变为左孩子的右孩子）
*/
inline void 
_Rb_tree_rotate_right(_Rb_tree_node_base* __x, _Rb_tree_node_base*& __root)
{
  _Rb_tree_node_base* __y = __x->_M_left; // 左孩子
  /* Step 1: left_child 的右子树 B 变成 p 的左子树 */
  __x->_M_left = __y->_M_right; 
  if (__y->_M_right != 0)
    __y->_M_right->_M_parent = __x; // B 的父节点修改

  /* Step 2: left_child 顶替 p 的位置 */
  __y->_M_parent = __x->_M_parent; // left_child 的父节点修改
  if (__x == __root) // p 为根节点
    __root = __y;
  else if (__x == __x->_M_parent->_M_right) // p 为父节点的左孩子
    __x->_M_parent->_M_right = __y;
  else                                      // p 为父节点的右孩子
    __x->_M_parent->_M_left = __y;
   
   /* Step 3: 修改 p 和 left_child 关系 */
  __y->_M_right = __x;
  __x->_M_parent = __y;
}
```

{% endfold %}

## STL红黑树插入节点

红黑树的插入操作主要步骤如下：

* 插入节点，和**二叉查找树**一样
* 调整结构，保证满足红黑树状态
  * 旋转操作
  * 重新着色（新插入节点初始为<font color=red>**红色**</font>）

**假设：**

>**X**：新节点（默认<font color=red>**红色**</font>）
>
>**P**：父节点
>
>**S**：伯父节点（父节点的兄弟节点）
>
>**G**：祖父节点
>
>**GG**：曾祖父节点

<table><tr><td bgcolor=yellow><b>强调：只有当父节点P为<font color=red >红色</font>时需要调整树形!!!</b></td></tr></table>

**插入节点分为以下情况：**

* **状况1：无父**

  > 没有父节点（即插入后为根节点）

  直接着色新节点为**黑色**。

* **状况2：父黑**

  > 父节点为黑色

  不用变色，新节点为<font color=red>**红色**</font>。

* **状况3：父红，伯黑（或NIL），父子同侧**

* **状况4：父红，伯黑（或NIL），父子异侧**

* **状况5：父红，伯红**


### 状况3：父红，伯黑（或NIL），父子同侧

> 父节点是红色，伯父节点也是黑色（或NIL），祖父节点必定是黑色。
>
> 父子节点同侧，即，子节点、父节点、以及祖父节点是直线型。

2. 父节点、祖父节点**变色**（父节点变为黑色，祖父节点变为红色）
2. 单旋，以**祖父节点**为旋转点
   * 父节点在左侧------右旋
   * 父节点在右侧------左旋

<img src="https://i.loli.net/2020/09/10/nSV3irRho1fXECK.png" alt="红黑树-父红-伯黑_或NIL_-父子同侧.png" />

### 状况4：父红，伯黑（或NIL），父子异侧

> 父节点是红色，伯父节点也是黑色（或NIL），祖父节点必定是黑色。
>
> 父子节点异侧，即，子节点、父节点、以及祖父节点是非直线型。

1. 单旋，以父节点为旋转点
   * 子节点在左侧------右旋
   * 子节点在右侧------左旋
2. 把**父节点**作为**“新节点”**
3. 按 **状况3：父红，伯黑（或NIL），父子同侧** 继续处理

<img src="https://i.loli.net/2020/09/10/WaumZbHoJGAKRU4.png" alt="红黑树-父红-伯黑_或NIL_-父子异侧.png" style="zoom:67%;" />

### 状况5：父红，伯红

> 父节点是红色，伯父节点也是红色，祖父节点必定是黑色。
>

1. 变色

   * 父节点变**黑色**
   * 伯父节点变**黑色**
   * 祖父节点变<font color=red>**红色**</font>

2. 把**祖父节点**作为**“新节点”**

3. 向上继续调整：

   3.1 曾祖父节点为**黑色**，结束

   3.2 当前新节点为**根**，直接设置成**黑色**，结束

   3.3 曾祖父节点为<font color=red>**红色**</font>，所属状况，继续调整树形

<img src="https://i.loli.net/2020/09/10/Xzra3KJARyQPIWB.png" alt="红黑树-父红-伯红.png" style="zoom:67%;" />

### STL红黑树调整平衡源码

{% fold 点击显示/隐藏-STL红黑树调整平衡%}

```cpp
inline void 
_Rb_tree_rebalance(_Rb_tree_node_base* __x, _Rb_tree_node_base*& __root)
{
  __x->_M_color = _S_rb_tree_red; // 新节点 设置为红
  /* 当前新节点不是根节点，并且其父节点为红色 */
  while (__x != __root && __x->_M_parent->_M_color == _S_rb_tree_red) {
    /* 父红，父节点为祖父节点的左孩子 */
    if (__x->_M_parent == __x->_M_parent->_M_parent->_M_left) {
      _Rb_tree_node_base* __y = __x->_M_parent->_M_parent->_M_right; // 伯父节点
      /* 状况5：父(左)红，伯(右)红 */
      if (__y && __y->_M_color == _S_rb_tree_red) { 
        __x->_M_parent->_M_color = _S_rb_tree_black; //父变黑
        __y->_M_color = _S_rb_tree_black; //伯变黑
        __x->_M_parent->_M_parent->_M_color = _S_rb_tree_red; //祖父变红
        __x = __x->_M_parent->_M_parent; //设置祖父为 当前新节点
      }
      else { // 伯父节点为黑色（或NIL）
      /* 状况4：父(左)红，伯(右)黑(或NIL)，新节点为右子 #附加# */
        if (__x == __x->_M_parent->_M_right) {
          __x = __x->_M_parent;
          _Rb_tree_rotate_left(__x, __root);
        }
        /* 状况3：父(左)红，伯(右)黑(或NIL)，新节点为左子 */
        __x->_M_parent->_M_color = _S_rb_tree_black;
        __x->_M_parent->_M_parent->_M_color = _S_rb_tree_red;
        _Rb_tree_rotate_right(__x->_M_parent->_M_parent, __root);
      }
    }
    else {
      _Rb_tree_node_base* __y = __x->_M_parent->_M_parent->_M_left; // 伯父节点
      /* 状况5：父(右)红，伯(左)红 */
      if (__y && __y->_M_color == _S_rb_tree_red) {
        __x->_M_parent->_M_color = _S_rb_tree_black; //父变黑
        __y->_M_color = _S_rb_tree_black; //伯父变黑
        __x->_M_parent->_M_parent->_M_color = _S_rb_tree_red; //祖父变红
        __x = __x->_M_parent->_M_parent; //祖父为 新节点
      }
      else {
        /* 状况4：父(右)红，伯(左)黑(或NIL)，新节点为左子 */
        if (__x == __x->_M_parent->_M_left) {
          __x = __x->_M_parent;
          _Rb_tree_rotate_right(__x, __root);
        }
        /* 状况3：父(右)红，伯(左)黑(或NIL)，新节点为右子 */
        __x->_M_parent->_M_color = _S_rb_tree_black;
        __x->_M_parent->_M_parent->_M_color = _S_rb_tree_red;
        _Rb_tree_rotate_left(__x->_M_parent->_M_parent, __root);
      }
    }
  }
  __root->_M_color = _S_rb_tree_black; // 根节点设置为黑色
}
```

{% endfold %}

## STL红黑树删除节点

红黑树可以看成是具有特殊性质的二叉查找树，因此红黑树的删除过程可分为两步：

* **第一步：二叉树的删除过程**
* **第二步：红黑树的调整过程**

### 二叉查找树的删除

二叉查找树的删除分为以下情况：

1. 删除节点是叶子节点，直接删除；
2. 删除节点只有左孩子（或右孩子），孩子节点替代删除节点位置；
3. 删除节点有两个孩子，选择一个合适的**子孙节点**替代删除节点位置，该节点称为**继承节点**。

### 红黑树的调整过程

按照二叉查找树的规则删除节点后，还需要检查是非满足红黑树的性质。

根据红黑树的性质，主要是性质4）父子节点不同为<font color=red>**红色**</font>，性质5）从任一节点到其子树中每个叶子节点的r任一路径都包含**相同数量**的**黑色**节点。



待补充。。。



## 参考

[重温数据结构：深入理解红黑树](https://blog.csdn.net/u011240877/article/details/53329023) 

[红黑树(一)之 原理和算法详细介绍](https://www.cnblogs.com/skywang12345/p/3245399.html)

