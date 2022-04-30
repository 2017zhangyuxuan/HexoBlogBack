---
title: 【STL源码分析】序列式容器之vector
date: 2022-04-26 19:54:21
excerpt: 开启STL源码分析新专题！本期带来的是STL序列式容器vector的介绍。
index_img: https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202204251956218.png
categories: 
- [计算机知识,C++]
tags:
- C++
- STL
---

# 前言

本文主要内容参考：[2 万字+20 图带你手撕 STL 序列式容器源码](https://mp.weixin.qq.com/s/NcrnwsB2gjq9h7W2hIZ6PQ)，原文内容非常详尽充实，建议大家阅读学习。而本文则是摘录总结关键部分，重点关注动态扩容的机制，以便日后快速回忆其实现逻辑。



# vector基本数据结构

![vector 定义](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202204261522995.png)

通过源码可以看到，vector代表着用户数据的一段空间，里面有三个重要的成员指针，分别指向所用空间的头部，所用空间的尾部，以及可用空间的尾部。并且，vector所用的迭代器就是普通指针。



# vector关键方法源码分析

接下来对vector中一些关键方法进行源码分析。

## reserve

`reserve`方法用于调整vector可用空间的大小，如果输入参数（要改变的大小）大于当前可用空间的容量，那么就会重新分配空间，再把原来的数据拷贝到新的空间上。

![reserve源码](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202204261522613.png)



## resize

`resize`方法是改变vector已使用空间的大小，如果输入的参数小于当前的size，就会清除多余的元素（调用`erase`方法）；否则，就执行 insert 插入方法，而对 insert 来说，如果当前可用空间为0，那么就会执行扩容，也就是分配申请新的空间，把原来的数据拷贝到新的空间上，再执行插入操作。

因此对比 `reserve` 和 `resize` 方法，`reserve` 只可能改变vector可用空间的大小(capacity)，不会改变使用空间的大小(size)；而`resize`方法既可能改变使用空间的大小(size)，也可能改变可用空间的大小(capacity)。

![resize源码](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202204261523596.png)

## push_back

`push_back` 将元素插入到vector的尾部，当vector目前还有可用空间时，就在对应的位置上执行元素的构造函数；如果没有可用空间，则调用`_M_insert_aux` 插入方法，在该方法里会执行动态扩容。

![push_back源码](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202204261525188.png)

`_M_insert_aux` 方法源码如下所示，除了`push_back` 可能会调用，`insert` 方法也可能会调用，所以首先还是会先判断当前是否还有可用空间。

如果当前没有可用空间，就会申请分配新的空间大小，值得注意的是，**如果原来的size为0，则新申请的空间大小为1，否则就会申请2倍原来的size**。

然后执行拷贝移动，将先拷贝`_M_start`到`__position`位置的元素，然后再`__position`位置上赋值插入元素，最后再拷贝`__position`到`_M_finish`位置的元素。

说明一点，其中调用的`uninitialized_copy`函数的前缀uninitialized含义是对应迭代器的位置还没有初始化，需要调用相应构造函数。

![_M_insert_aux 源码](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202204261526804.png)

## erase

`erase`函数用于清除某个位置上或者某一段区间上的元素，主要实现逻辑就是用后段元素覆盖删除区间内的元素。主要调用了 `copy(first, last, result)` 函数，其大致作用是把 [first,last) 数据拷贝到result（起始位置），从前往后拷贝。

![erase源码](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202204261526218.png)

`erase`函数图解如下，可以帮助更好地理解。

![erase函数图解](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202204261527816.png)

## insert

insert函数有多个重载，这里重点分析其中一个，`void insert (iterator __pos, size_type __n, const _Tp& __x)`，函数作用为从 pos 位置开始，插入 n 个元素，元素初值为 x，真正实现逻辑在`_M_fill_insert`函数中，源码如下所示。

![insert源码](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202204261528232.png)

通过源码，不难发现 vector 的插入多个元素，大体上可以分为3种情况：

1. 如果**备用空间足够**且插入点之后的现有元素个数 **多于** 新增元素个数；
2. 如果**备用空间足够**且插入点之后的现有元素个数 **少于** 新增元素个数；
3. 如果**备用空间不够**。

下面就用三张图来描述这三种情况的执行逻辑：

![情况1 图解](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202204261529054.jpeg)

![情况2 图解](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202204261529421.jpeg)

![情况3 图解](https://kingofdark-blog.oss-cn-beijing.aliyuncs.com/picture_backend/picture_backend/img/202204261530582.jpeg)

## 注意点

在谈论vector时，经常会听到一个名词，就是所谓的**迭代器失效问题**。通过上述图解和分析我们就明白了，所谓的迭代器失效问题是由于元素空间重新配置导致之前的迭代器访问的元素不在了，总结来说有两种情况：

- 由于插入元素，使得容器元素整体迁移导致存放原容器元素的空间不再有效，从而使得指向原空间的迭代器失效。
  - 这里元素迁移，既可能是指容器内元素的次序移动，也可能是由于指向了新的分配空间

- 由于删除元素，使得某些元素次序发生变化导致原本指向某元素的迭代器不再指向期望指向的元素。

---

之前也谈到一些关键的函数，这里再记录说明一下：

- `copy(first, last, result)`：将[first, last) 之间的元素拷贝到 [result , result + (last-first))位置
  - 该全局 `copy` 函数，底层会去调用 `memmove` ，而 `memmove`用于拷贝字节，如果目标区域和源区域有重叠的话，`memmove`能够保证源串在被覆盖之前将重叠区域的字节拷贝到目标区域中，但复制后源内容会被更改。
    - 如果存在重叠区域的话，从实现角度来说，如果 first 在 result 之前，那么就从后往前拷贝，如果 first 在 result 之后，那么从前往后拷贝
  - 也就是说，即使 result 迭代器位置处于 first 和 last 之间，也能够正常实现功能，不会出现想象中迭代器值被覆盖的问题。
  
- `uninitialized_copy(first, last, result)`：具体作用是将 [first,last)内的元素拷贝到[result , result + (last-first)) ，**从前往后拷贝**

- `copy_backward(first, last, result)`：将 [first,last)内的元素拷贝到 [result , result + (last-first)) ，**从后往前拷贝**

# 总结

至此 vector 容器就分析得差不多了，提醒需要注意的是：vector 的成员函数都不做边界检查 (但at方法会抛异常)，使用者要自己确保迭代器和索引值的合法性。

最后再总结下 vector 的优缺点：

- **优点**
  - 在内存中是分配一块连续的内存空间进行存储，可以像数组一样操作，并且支持动态扩容。
  - 元素的随机访问方便，支持[ ]下标访问和 vector.at() 操作。
  
  - 节省空间（对比其他容器使用空间而言）。
  
- **缺点**
  - 由于其顺序存储的特性，vector 插入删除操作的时间复杂度是 O(n)。
  - 只能在末端进行 pop 和 push。
  - 当动态长度超过默认分配大小后，要整体重新分配、拷贝和释放空间，开销较大。
