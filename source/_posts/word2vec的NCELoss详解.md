---
title: word2vec的NCELoss详解
date: 2020-08-31 14:11:34
tags:
---

>word2vec设计的nce(noise-contrastive estimation)损失函数，其实在推荐系统以及图神经网络等领域也是十分常见的，那么为什么这么设计呢？它又有什么思想可以追溯呢？本文就谈谈我的见解

## sigmod函数

sigmod可以说是无人不知，无人不晓，那么我现在很好奇的问大家一句？sigmod为什么长这样？机器学习为什么使用sigmod？如何推导sigmod？嗯，这些都是我不知道的东西，所以我决定首先引入sigmod。
$$
\sigma=\frac{1}{1+e^{-x}}
$$
