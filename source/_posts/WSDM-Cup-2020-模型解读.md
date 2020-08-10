---
title: WSDM Cup 2020 模型解读
date: 2020-03-25 17:13:12
categories：WSDM-Cup-模型解读
tags:
---

- 比赛背景

  - 任务
- 数据介绍
  - 评估指标
- 模型方法
- 特征工程
- 附录

## 1. 比赛背景

### 1.1 任务

任务要求参赛者根据论文中对某项科研工作的描述，从论文库中找出与该描述最匹配的Top3论文。

其中描述指的是如下这种形式，`[[##]]`是原论文中的引用标号位置。

> *Rat brain membrane preparation and opioid binding was performed as described previously by Loukas et al. [[**##**]]. Briefly, binding was performed in Tris-HCl buffer (10 mM, pH 7.4), in a final volume of 1.0 ml. The protein concentration was 300 μg/assay.*

### 2.2 数据介绍

![表1 评测数据分析表](https://mmbiz.qpic.cn/mmbiz_png/hEx03cFgUsUW7fE5Otnx3pXs7zrgRics5BggHTFX2IG1LtHhia3jvEIxicVqRKlSIce8qibQiaVW0K6ibdo6A2I72kXw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 2.3 评估指标

本次评测使用的评价指标为Mean Average Precision @3 (MAP@3), 形式如下：
$$
MAP@3=\frac{1}{|U|} \sum_{u=1}^{|U|} \sum_{k=1}^{\min(3,n)}P(k)
$$
其中，|U|是需要预测的description总个数，P(k)是在k处的精度，n是paper个数。举例来说，如果在第一个位置预测正确，得分为1；第二个位置预测正确，得分为1/2；第三个位置预测正确，得分为1/3。

## 2. 模型方法

模型主要部分分为**召回**和**排序**两部分，那就不是end2end的模型了吧。

![img](https://mmbiz.qpic.cn/mmbiz_png/hEx03cFgUsUW7fE5Otnx3pXs7zrgRics5iaIVuHQDjI7fIqj6PAicbq93235EwQCM6cwoy3YRx5pCYpndGBm0dU7Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**召回部分**：

- 使用了倒排索引，可以提高召回速度，就是一个小trick，因为要求词频嘛。

- 相比于传统的[BM25](https://en.wikipedia.org/wiki/Okapi_BM25)和[TFIDF](https://en.wikipedia.org/wiki/Tf–idf)等算法，F1EXP、F2EXP等公理检索模型（Axiomatic Retrieval Models）可以取得更高的召回覆盖率，该类模型增加了一些公理约束条件，例如基本术语频率约束，术语区分约束和文档长度归一化约束等等。简单说就是使用了更加复杂的指标，这些指标在论文[^1]可以找到。比如F2EXP的

  ![img](https://mmbiz.qpic.cn/mmbiz_png/hEx03cFgUsUW7fE5Otnx3pXs7zrgRics5y1tJgQE9fw3eMfA2qpLOG6AEW9pCvjDwIs7gXmcmdDCqSO4aDS8m1Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  其中，Q表示查询query ,D表示候选文档，C(t, Q)是词t在Q中的频次，|D|表示文档长度，avdl为文档的平均长度，N为文档总数，df(t)为词t的文档频率。

![img](https://mmbiz.qpic.cn/mmbiz_png/hEx03cFgUsUW7fE5Otnx3pXs7zrgRics5JggdnHd4Sq7u1ATd2kzffA8uaa8nwOAntGVia9NpmFX7HxWr6h5OxYw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**排序过程**

- pairwise的BERT模型，使用了两个别人训练好的，分别是关于sciBERT和bioBERT，因为本次任务大多和生物方面的论文有关，所以使用了这些，然后做了fine-tune，用了pairwise的损失函数
- LightGBM，这个就不多说了，速度比XGB快。树模型，弥补BERT的不足，与BERT算是互补吧，融合一下会有提升。

### 训练trick

- 它论文中说一个描述可能对应多个paper_id，那么如果看作分类任务(相关，不相关)，就会正负样本不齐，因此似乎召回的样例不能过多，但是它发现正例的覆盖率似乎更加重要，所以还是选择召回了更多的样本。
- 还有各种预处理啊，诸如单词的去除停用词、全部改小写，过滤一些特殊字符等。还有一些各种特殊的处理，比如缺失值之类的，代码太长了，我就没仔细看，

## 3. 特征工程

基于Light GBM的方案需要特征工程的配合。特征工程它讲解里写的不是十分详细，我这里按照论文以及自己理解说几点：

- Semantic feature，就是一些诸如Glove、Doc2vec、Word2Vec模型训练的向量，然后通过求描述与摘要之间的相似度(代码里有余弦、曼哈顿、欧氏距离、jacarrd等），将这些作为feature，论文中提到的是取平均值。
- Statistical features and word frequency features. 简单来说就是在Semantic feature的基础上又考虑了词频等等的影响，比如得到query和doc的tfidf然后求余弦相似，或者BM25等直接求描述与摘要之间的相似性。此外还有F1EXP等，论文种说1.5%提升。
- Rank features 很好理解，就是召回阶段的排在第几位，那么n种排序方法就会再多出n个排序特征。论文说boosting有3%提升

![img](https://mmbiz.qpic.cn/mmbiz_png/hEx03cFgUsUW7fE5Otnx3pXs7zrgRics5k4G1a0c70hs3S5RbT8UItEcaBDiaocQwmAWk5JgQnIFXOgtzRVl4IFA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 4. 附录

[WSDM Cup 2020 Reports](http://www.wsdm-conference.org/2020/wsdm-cup-2020-reports.html)

[公众号解读](https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651751493&idx=2&sn=cd6eb487779b5936f88e58539af2bc18&chksm=bd125d088a65d41ea48ae4f1d757deba99b25e9cef61a7d62603adcc0ee62a1b5534c136a369&mpshare=1&scene=1&srcid=&sharer_sharetime=1585060490833&sharer_shareid=c4f945cf686d198441452241f87a651d&key=c6bd44209e6ec513b14ce33c2082a923742b3e9324233c7edb7a6aaf33abebf6a2d6cc6b06e5436e262d31e537c114c29d484769d15c6f01a8eb8621d247ef86803a99b0ae3d36418d06df8a1baab6bf&ascene=1&uin=MTkxNjUzMTQ3MQ%3D%3D&devicetype=Windows+10&version=62080079&lang=zh_CN&exportkey=A2RJI%2F%2F0YlnQfDtpj38893Q%3D&pass_ticket=BXwhz2dhldhVnhUsp%2Beg8qE0nhhrp4unTlE%2F4by1lVC6rs2pdrDzx7A8ETKjRVoK)

[ ^1]: 专栏论文(http://www.wsdm-conference.org/2020/wsdm_cup_reports/Task1_Ferryman.pdf)

