---
title: FM大家族
date: 2021-03-09 14:35:48
mathjax: true
categories:
- 经典模型详解系列
tags:
- FM
- 推荐系统
- context
---

> FM(Factorization Machine)类的模型貌似是一大类吧，这里就稍微总结下，之前一直对这类模型不太了解，也算是借此机会学习一下。

<!--more-->

## 1.最基础的FM

FM要解决的就是特征交叉的问题，推荐系统的特征往往可以分为三类

- User特征: user_id, 年龄，性别等
- Item特征：item_id, 类目，价格，商标等
- Context特征：交互时间戳，rating，评价等

常见的特征形式包含四类:

- 离散特征。比如user_id, item_id，年龄，性别这种
- 连续特征。比如商品价格，timestamp这种
- 离散序列特征。比如历史item_id的一个list，商品描述的word_id的一个list
- 连续序列特征。比如我pretrain好的某种预训练特征。比如商品的文本，图片等信息。

PS：有些模糊的特征既可以作为离散也可以作为连续的，私以为就是以就结果论处了，比如time属性，对于数据集来说，他可能呈现的比较离散又比较连续，也就是unique的值不大不小，不过处理成离散的可能会引入一些空间复杂度，但应该可以更好的被train到。因为时间特征可能本身分布就比较有规律。

### FM公式解释

$$
y=\omega_{0}+\sum_{i=1}^{n} \omega_{i} x_{i}+\sum_{i=1}^{n-1} \sum_{j=i+1}^{n}<v_{i}, v_{j}>x_{i} x_{j}
$$

最基础的FM长这样，$n$是特征数量，$\omega$是参数。我们希望特征做交叉，自然是不同特征做交叉，但是注意到我们输入的item_id, user_id可能会被转化为一个one-hot向量，它自然就有很多维度，那问题来了，$n$和$x$的粒度是什么？如果是element-wise的，user_id会和自身做交叉嘛？如果有multi-hot特征怎么办？如果是feature-level的，不同特征维度难道必须一样？

实际上公式里的$x$是element-level的，而不是feature level的，所以这里$n$个特征，并不是实际我们说的那些年龄啊，用户id之类的有几个，而是所有特征展开(one-hot)后的长度之和。对于$v$的话，由于$x$是原始输入，是固定不变的，那么$x$中的每一个element，都会有一个向量$v_i$，他们维度为$k$，这个是可以train的。

PS：FM会存在类内交叉的现象，因为粒度是element。举个例子，如果特征中有multi-hot，比如用户历史观看的电影类型，如果一个用户看过10个喜剧片，但candidate是一个恐怖片，而且真实的y为0，那么在交叉时，对于喜剧片内部来讲，一个正常的模型，理论上相似的片子$v$之间的相似度高，但FM模型却希望降低$y$值，这会导致一定的错误优化。但同样的，考虑喜剧片与恐怖片的话，这时候$v$之间的相似度希望会变小，这里又是正向优化，因此使用multi-hot将是一个对抗的过程，比较难train。

因为FM中$n$实际上是总维度，那么其就可以处理所有的特征，但要求就是输入中的离散特征必须是one-hot，=。而且不管你是multi-hot还是连续的序列，还是就是一个值，都是一个位置对应一个embedding，所以大的方面来看，特征不要求等长。

仅使用FM原公式中二阶部分，不做简化的话，就是$\frac{1}{2}x(VV^t-diag(VV^t))x^t$，其中$x\in R^n，V\in R^{n \times k}$。

### FM原理解析

前面这一部分可以看作是一个线性模型，是LR模型的一部分，后面则是一个二阶特征交叉，$<V_i,V_j>$表示点积，之所以使用点积是因为如果依然用$\omega$来表示，会使得$\omega$不太好train，因为$x_i, x_j$的组合不一定在整个数据集中都有，可能会有冷启动的一些特征pair，那么就会导致$\omega$train不好，也就是稀疏性。所以他替换$\omega$为了向量点积的形式。

PS：实际上向量点积就是利用矩阵分解解决稀疏性问题，这也正是他优于SVM的原因，因为SVM在测试里不能有没见过的特征pair。话说，MF类模型也算是老鼻祖了，万物不离其中啊

### FM的简化

FM模型最大的优点其实是快，理论上计算二阶交叉，至少也是$o(kn^2)$，但是他做了简化，就变成了$o(kn)$了，这里我简单的推导一下:
$$
\begin{align}
& \sum_{i=1}^{n} \sum_{j=i+1}^{n}\left\langle\mathbf{v}_{i}, \mathbf{v}_{j}\right\rangle x_{i} x_{j}  \tag{1}\\
=& \frac{1}{2} \sum_{i=1}^{n} \sum_{j=1}^{n}\left\langle\mathbf{v}_{i}, \mathbf{v}_{j}\right\rangle x_{i} x_{j}-\frac{1}{2} \sum_{i=1}^{n}\left\langle\mathbf{v}_{i}, \mathbf{v}_{i}\right\rangle x_{i} x_{i} \tag{2}\\
=& \frac{1}{2}\left(\sum_{i=1}^{n} \sum_{j=1}^{n} \sum_{f=1}^{k} v_{i, f} v_{j, f} x_{i} x_{j}-\sum_{i=1}^{n} \sum_{f=1}^{k} v_{i, f} v_{i, f} x_{i} x_{i}\right)  \tag{3}\\
=& \frac{1}{2} \sum_{f=1}^{k}\left(\left(\sum_{i=1}^{n} v_{i, f} x_{i}\right)\left(\sum_{j=1}^{n} v_{j, f} x_{j}\right)-\sum_{i=1}^{n} v_{i, f}^{2} x_{i}^{2}\right)  \tag{4}\\
=& \frac{1}{2} \sum_{f=1}^{k}\left(\left(\sum_{i=1}^{n} v_{i, f} x_{i}\right)^{2}-\sum_{i=1}^{n} v_{i, f}^{2} x_{i}^{2}\right) \tag{5}
\end{align}
$$
**(1)->(2)**

很简单，这实际上是二次型中的上半角矩阵，但不包含对角线(包含的话就是自身交叉)，其自然等于(全部的-对角的)，然后除以二

**(2)->(3)**

将向量点乘展开了，这个也没啥，多了一个求和符号而已。

**(3)->(4)**

k的求和提到外面，然后两两合并，可以从二重积分上下限无关的角度理解这个变换，或者从(4)->(3)来逆向理解，就是展开嘛，都很简单。

**(4)->(5)**

可不就是自己的平方嘛

## 2.FM大家族

- **Field-aware FM**

> 文章链接：[https://www.csie.ntu.edu.tw/~r01922136/slides/ffm.pdf](https://link.zhihu.com/?target=https%3A//www.csie.ntu.edu.tw/~r01922136/slides/ffm.pdf)

$$
y_{f f m}=w_{0}+\sum_{i=0}^{n} w_{i} x_{i}+\sum_{i=1}^{n} \sum_{j=i+1}^{n}<\mathbf{v}_{i, f_{j}}, \mathbf{v}_{j, f_{i}}>x_{i} x_{j}
$$

思想很简单，就是不同特征之间交叉，需要有域这个概念，比如<user, item, context>这种，三个域有其各自的的属性。那么之前有$n$个特征，对应的是$n$个向量，现在就对应$n\times3$个向量了。计算相似度的时候要分别对对面域的向量。比如一个user域和一个item域的向量<user,item> 那么计算$x_ix_j$的时候，就要用分别用对面的域来计算，<item,user>来计算。挺绕的，而且这种复杂度很高，因为无法用FM方法优化。

- **Attentional FM**

> 文章链接：[Attentional Factorization Machines: Learning the Weight of Feature Interactions via Attention Networks](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1708.04617v1)

$$
y_{a f m}=w_{0}+\sum_{i=1} w_{i} x_{i}+\mathbf{p}^{T} \sum_{i=1} \sum_{i=j+1} a_{i j}\left(\mathbf{v}_{i} \odot \mathbf{v}_{j}\right) x_{i} x_{j} \\
$$

思路更进一步，引入了attention。注意到$p$以及$a_{ij}$都为$1$的时候，退化为FM，attention的计算方式为
$$
\begin{aligned}
a_{i j}^{\prime} &=\mathbf{h}^{T} \operatorname{ReLU}\left(\mathbf{W}\left(\mathbf{v}_{i} \odot \mathbf{v}_{j}\right) x_{i} x_{j}+\mathbf{b}\right) \\
a_{i j} &=\frac{\exp \left(a_{i j}^{\prime}\right)}{\sum_{(i, j) \in \mathcal{R}_{x}} \exp \left(a_{i j}^{\prime}\right)}
\end{aligned}
$$
这种attention也是很常见的，也是偏早期的了。$\odot $就是element-wise的乘法，也叫哈达玛加积。这个模型整体复杂度还是偏高的，哈达玛积要计算$\frac{n(n-1)}{2}$个pair，那么最后attention实际上就要得到这么多个$a_{ij}$，那么对于user_id以及item_id这种one-hot，将是相当的恐怖。所以实践中都不用id类特征？而是以embedding为初始?

- **Field-weighted FM**

> 文章链接：[[1806.03514\] Field-weighted Factorization Machines for Click-Through Rate Prediction in Display Advertising](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1806.03514)

$$
y_{f w f m}=w_{0}+\sum_{i=1}^{n} w_{i} x_{i}+\sum_{i=1}^{n} \sum_{j=i+1}^{n}<\mathbf{v}_{i}, \mathbf{v}_{j}>x_{i} x_{j} r_{f_{i}, f_{j}}
$$

没啥大改进，引入了域相关性，类似FFM，比如有$m$个域，那就引入了$m$个向量。其它似乎没啥，然后论文还提到了，线性部分，可以用$v$和field的相似度代替。
$$
\sum_{i=1}^{m} x_{i}\left\langle\boldsymbol{v}_{i}, \boldsymbol{w}_{F(i)}\right\rangle
$$
其他的似乎也没啥了。

## 3. 总结

其实还有很多FM类模型没列举，而且也有一些与FM结合的入NFM，DeepFM等，这些都是以FM作为一个模块嵌入到模型中的，相对就比较简单了，这里就不赘述了。u1s1，看了几个模型，发现还是FM好，毕竟效率很重要，其他模型肉眼可见的超高复杂度，实际公司真的会用吗?