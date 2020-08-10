---
title: 矩阵的matmul运算
date: 2020-01-26 22:49:13
mathjax: true
categories:
- pytorch备忘录
tags:
- torch.matmul
- boardcast
---

>这个是一个很容易搞晕的东西，比如什么时候两个矩阵乘法是做广播(boardcast)，什么时候是做矩阵点积(matmul product)，有时候搞错了也找不到bug在哪里，所以我在这里总结一下。具体内容参考了[matmul](https://pytorch.org/docs/stable/torch.html?highlight=matmul#torch.matmul)和[boardcast](https://pytorch.org/docs/stable/notes/broadcasting.html#broadcasting-semantics)的官方文档。

<!--more-->

### 矩阵的matmul操作

- 都是一维的话就是正常的点积,不存在广播

  ```python
  # 自然是必须维度一致了
  a = torch.ones(5)
  b = torch.ones(5)
  print(a.matmul(b))
  # output => 
  # tensor(5.)
  ```

- 都是二维就是做矩阵的正常的点积,不存在广播

  ```python
  a = torch.ones(4, 2)
  b = torch.ones(2, 3)
  print(a.matmul(b))
  # output => 
  # tensor([[2., 2., 2.],
  #        [2., 2., 2.],
  #        [2., 2., 2.],
  #        [2., 2., 2.]])
  ```

- 如果前者是一维的$m$，后者是二维$m \times n$的，则运算过程相当于将前者转化为$1 \times m$，然后运算后去掉补充的$1$

  ```python
  a = torch.ones(2)
  b = torch.ones(2, 3)
  print(a.matmul(b).shape)
  # output => 
  # torch.Size([3])
  ```

- 与上一条对称，如果如果前者是二维的$m \times n$，后者是一维的$n$的情况

  ```python
  a = torch.ones(4, 2)
  b = torch.ones(2)
  print(a.matmul(b).shape)
  # output => 
  # torch.Size([4])
  ```

- 如果其中一个二维的或者一维的，另一个是$N$维的，$N > 2$，那么相当于做的事`batched matrix multiply`，当然其对称情况也同理，**$N$维的只有后两维才参与运算！！**

  ```python
  # 假如a是1维的，这个就很容易出错了，注意其对其方式
  a = torch.ones(      4)  # 如果是2,3或者5会报错！！！
  b = torch.ones(2,3,4,5)
  # 相当于是(1,4),与一个((2,3),(4,5))相乘得到((2,3),(1,5)),然后去掉1就是(2,3,5)
  print(a.matmul(b).shape)
  # output => 
  # torch.Size([2, 3, 5])
  # 如果a是二维的 
  # a = torch.ones(    6,4)
  # output => 
  # torch.Size([2,3,6,5])
  ```

  自然其对称情况也同理

  ```python
  a = torch.ones(2,3,4,5)
  b = torch.ones(      5)  # 注意这里应该是5了,因为扩展后是(5,1)
  # 相当于是((2,3),(4,5))与一个(5,1)相乘得到((2,3),(4,1)),然后去掉1就是(2,3,4)
  print(a.matmul(b).shape)
  # output => 
  # torch.Size([2, 3, 4])
  # 如果b是二维的 
  # a = torch.ones(    5,6)
  # output => 
  # torch.Size([2,3,4,6])
  ```

- 如果是非单纯的batch点乘形式的（不是上面几种情况的），那么就要对一部分做boardcast，也就是广播，那么其必须就要满足广播的条件，那么广播的条件是什么呢？

  - 两个张量可以广播运算，当且仅当**从最后一维向前看，他们的维度要么相等，要么其中有一个是$1$，要么其中一个维度不存在，也就是到头了。注意！！！这里的广播是点乘的广播！！！，所以实际是从倒数第三位开始看**

  也就是说我们可以把后两个维度看作是一个整体，比如说是一个元素$k$，那么一个$m\times n \times p \times q$的矩阵与一个$r \times q \times v$的矩阵如果使用`torch.matmul()`，那么可以理解为一个$m\times n$的矩阵与一个$r$的一维矩阵的运算，只不过这里矩阵里的元素不再是一个数，而是一个矩阵$k$，而这种运算是点乘而已。

  ```python
  a = torch.ones(2,1,3,4)
  b = torch.ones(  6,4,5) 
  # 注意这里的点积是发生在(3,4)和(4,5)之间的，广播是发生在1和6之间的，这里的1和6相当于广播中的最后一位。
  print(a.matmul(b).shape)
  # output => 
  # torch.Size([2, 6, 3, 5])
  
  # 我这里再举例一个复杂的例子
  a = torch.ones(2,1,3,4,5,6,7)
  b = torch.ones(  2,3,1,5,7,8) 
  print(a.matmul(b).shape)
  # output => 
  # torch.Size([2, 2, 3, 4, 5, 6, 8])
  # 可以自行分析一个这个例子，按照传统的加减乘除的广播类比到这里理解就可以了，也就是说相当于是
  # (2,1,3,4,5) 和 (2,3,1,5)之间的广播。
  ```

### Summary

`torch.matmul()`运算的两个对象中，如果有任何一个是二维以内的矩阵，那么做的就是带batch的普通点积，如果两个都是二维以外的话，那么做的就是带有广播性质的点乘，它广播的内容是后两位的点积，那么可以将后两位构成的矩阵，当作一个整体，看作是前$N-2$维矩阵的元素，那么matmul运算就与$N-2$维矩阵的加减乘除的广播类似，只不过这里的运算不再是加减乘除，而是点积。