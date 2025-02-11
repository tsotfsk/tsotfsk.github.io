---
title: GCN要用到的数学知识系列(三)
date: 2019-12-15 17:45:32
mathjax: true
categories:
- GCN要用到的数学知识
tags:
- 拉普拉斯算子
- 拉普拉斯矩阵
- 无向图
- 热力学定律
---

## **三、拉普拉斯矩阵**

> GCN中的图傅里叶变换(Graph Fourier Transformation)和图卷积(Graph Convolution)的定义都用到了图的拉普拉斯矩阵，那么首先来介绍一下拉普拉斯矩阵。为了让读者更易理解拉普拉斯矩阵，这里借鉴了[某知乎大佬的文章](https://www.zhihu.com/question/54504471/answer/630639025)中的观点，因为发现他的角度实在是太妙了，但以我表达能力又无法转述，所以这里前半部分几乎是整段复制的大佬的文章，不过在一些小地方我也做了一些批注。

<!--more-->

众所周知，没有外接干预的情况下，热量从温度高传播到温度低的地方并且不可逆，根据著名的牛顿冷却定律（Newton Cool's Law），热量传递的速度正比于温度梯度，直观上也就是某个地方A温度高，另外一个B地方温度低，这两个地方接触，那么温度高的地方的热量会以正比于他们俩温度差的速度从A流向B。

### **从一维空间开始**

我们先建立一个一维的温度传播的模型，假设有一个均匀的铁棒，不同位置温度不一样，现在我们刻画这个铁棒上面温度的热传播随着时间变化的关系。预先说明一下，一个连续的铁棒的热传播模型需要列**温度对时间和坐标的偏微分方程**来解决，我们不想把问题搞这么复杂，我们把**空间离散化**，假设铁棒是一个**一维链条**，链条上每一个单元拥有一致的温度，温度在相邻的不同的单元之间传播，如下图

<img src="GCN要用到的数学知识系列(三)/链条.png" style="zoom:100%;" />

对于第 $i$个单元，它只和 $i-1$与$i+1$两个单元相邻，接受它们传来的热量（或者向它们传递热量，只是正负号的差异而已），假设它当前的温度为，那么就有

$$
{d\phi_i  \over dt} = k(\phi_{i+1} - \phi_i) - k(\phi_{i} - \phi_{i-1})
$$
$k$和单元的比热容、质量有关是个常数。右边第一项是下一个单元向本单元的热量流入导致温度升高，第二项是本单元向上一个单元的热量流出导致温度降低。做一点微小的数学变换可以得到：

$$
{d\phi_i  \over dt} = k(\phi_{i+1} - \phi_i) - k(\phi_{i} - \phi_{i-1}) = 0
$$
注意观察第二项，它是两个差分的差分，在离散空间中，相邻位置的差分推广到连续空间就是**导数**，那么差分的差分，就是**二阶导数**！

所以，我们可以反推出铁棒这样的连续一维空间的热传导方程就是：
$$
{\partial \phi_i  \over \partial t} = k{\partial^2 \phi_i  \over \partial x^2}
$$
同理，在高维的欧氏空间中，一阶导数就推广到**梯度**，二阶导数就是我们今天讨论的主角——**拉普拉斯算子**


其中 $ \Delta $这个符号代表的是对各个坐标二阶导数的加和，现在的主流写法也可以写作 $\nabla^2$

综上所述，我们发现这样几个事实：

1、在欧氏空间中，某个点温度升高的速度正比于该点周围的温度分布，用拉普拉斯算子衡量。

2、拉普拉斯算子，是二阶导数对高维空间的推广。

那么，你肯定会问：你扯这么多有什么用呢？我还是看不到拉普拉斯算子和拉普拉斯矩阵还有GCN有半毛钱关系啊？

不要急，目前只是第一步，让我们把这个热传导模型推广导拓扑空间，你就会发现它们其实刻画的是同一回事了！

### **图(Graph)上热传播模型的推广**

现在，我们依然考虑热传导模型，只是这个事情不发生在欧氏空间了，发生在一个Graph上面。这个图上的每个结点（Node）是一个单元，且这个单元只和与这个结点相连的单元，也就是有边（Edge）连接的单元发生热交换。例如下图中，结点$1$只和结点$0、2、4$发生热交换，更远的例如结点5的热量要通过$4$间接的传播过来而没有直接热交换。

<img src="GCN要用到的数学知识系列(三)/温度.png" style="zoom:100%;" />

我们假设热量流动的速度依然满足牛顿冷却定律，研究任一结点 $i$，它的温度随着时间的变化可以用下式来刻画
$$
{d\phi \over dt} = -k \sum_{j}A_{ij}(\phi_i - \phi_j)
$$
其中$A$是这个图的邻接矩阵(Adjacency Matrix)，定义非常直观： 对于这个矩阵中的每一个元素 ，如果节点$i$和$j$相邻，也就是它们之间存在一条边，那么$A_{ij}=1$，否则$A_{ij}=0$，我们只讨论简单那情况

- 这张图是无向图，$i$和$j$相邻，那么$j$和$i$也相邻，也就是$A_{ij}=A_{ji}$，这是个对称阵。

- 节点自己到自己是没有回环边的，也就是$A$的对角线上的元素都为0。

所以不难理解上面这个公式恰好表示了只有相邻的边才能和本结点发生热交换且热量输入（输出）正比于温度差。

我们不妨用乘法分配律稍微对上式做一个推导
$$
\begin{align*}
{d\phi \over dt} =& -k [\phi_i\sum_{j}A_{ij}-\sum_{j}A_{ij}\phi_j] \\
=& -k [deg(i) \phi_i- \sum_{j}A_{ij}\phi_j]
\end{align*}
$$
先看右边括号里面第一项，$deg(i)$代表对这个顶点求度（degree），一个顶点的度被定义为这个顶点有多少条边连接出去，很显然，根据邻接矩阵的定义，第一项的求和正是在计算顶点$i$的度。

再看右边括号里面的第二项，这可以认为是邻接矩阵的第$i$行对所有顶点的温度组成的向量做了个内积。

为什么要作上述变化呢，我们只看一个点的温度不太好看出来，我们把所有点的温度写成向量形式再描述上述关系就一目了然了。首先可以写成这样
$$
\begin{bmatrix}
d \phi_1 \over dt \\
\vdots \\
d \phi_n \over dt 
\end{bmatrix}
=-k \begin{bmatrix}
deg(1) \times \phi_1  \\
\vdots \\
deg(n) \times \phi_n 
\end{bmatrix} + kA
\begin{bmatrix}
\phi_1  \\
\vdots \\
\phi_n 
\end{bmatrix}
$$
然后我们定义向量$ \phi = [ \phi_1, \phi_2,\cdots,\phi_n ] ^T$，那么就有
$$
{d\phi \over dt }= -k D \phi + kA \phi = -k(D-A)\phi
$$
其中$ D=diag(deg(1),deg(2),\cdots,deg(n)) $被称为度矩阵，只有对角线上有值，且这个值表示对应的顶点度的大小。整理整理，我们得到
$$
{d \phi \over dt } + kL \phi = 0
$$
回顾刚才在连续欧氏空间的那个微分方程
$$
{\partial \phi \over \partial t} - k \Delta \phi = 0
$$
二者具有一样的形式！我们对比下二者之间的关系

- 相同点：刻画空间温度分布随时间的变化，且这个变化满足一个相同形式的微分方程。
- 不同点：**前者刻画拓扑空间有限结点**，用向量$\phi$来刻画当前状态，单位时间状态的变化正比于线性变换$\-L$算子作用在状态$\phi$上。**后者刻画欧氏空间的连续分布**，用函数$\phi(x,t)$来刻画当前状态，单位时间状态变化正比于拉普拉斯算子$\Delta$作用在状态$\phi$上。

不难发现，这就是**同一种变换、同一种关系在不同空间上面的体现**，实质上是一回事！

于是我们自然而然，可以把连续空间中的热传导，推广到图（Graph）上面去，我们把图上面和欧氏空间地位相同变换，以矩阵形式体现的$L$叫做拉普拉斯（Laplacian）矩阵。

需要多嘴一句的是，本文开头所说的离散链条上的热传导，如果你把链条看成一个图，结点从左到右编号1，2，3...的话，也可以用图的热传导方程刻画，此时$D$除了头尾两个结点是1其他值都是2，$A$的主对角线上下两条线上值是1，其他地方是0

### **推广到GCN**

现在**问题已经很明朗**了，只要你给定了一个空间，给定了空间中存在一种东西可以在这个空间上流动，两邻点之间流动的强度正比于它们之间的状态差异，那么**何止是热量可以在这个空间流动，任何东西都可以！**

自然而然，假设在图中各个结点流动的东西不是**热量**，而是**特征（Feature）**，而是**消息（Message）**，那么问题自然而然就被推广到了GCN。**所以GCN的实质是什么，是在一张Graph Network中特征（Feature）和消息（Message）中的流动和传播！这个传播最原始的形态就是状态的变化正比于相应空间（这里是Graph空间）拉普拉斯算子作用在当前的状态。**

抓住了这个实质，剩下的问题就是怎么去更加好的建模和解决这个问题。

建模方面就衍生出了各种不同的算法，你可以在这个问题上面复杂化这个模型，不一定要遵从牛顿冷却定律，你可以引入核函数、引入神经网络等方法把模型建得更非线性更能刻画复杂关系。

解决方面，因为很多问题在频域解决更加好算，你可以通过Fourier变换把空域问题转化为频域问题，解完了再变换回来，于是便有了所有Fourier变换中的那些故事。

扯了这么多，总结一下，问题的本质就是：

- 我们有张图，图上每个结点刻画一个实体，物理学场景下这个实体是某个有温度的单元，它的状态是温度，广告和推荐的场景下这个实体是一个user，一个item，一个ad，它的状态是一个embedding的向量。
- 相邻的结点具有比不相邻结点更密切的关系，物理学场景下，这个关系是空间上的临近、接触，广告和推荐场景下这个是一种逻辑上的关系，例如用户购买、点击item，item挂载ad。
- 结点可以传播热量/消息到邻居，使得相邻的结点在温度/特征上面更接近。

**本质上，这是一种Message Passing，是一种Induction，卷积、傅立叶都是表象和解法。**

### **拉普拉斯矩阵的性质**

$Therom :$拉普拉斯矩阵是半正定[^1]的

$Proof:$
$$
\begin{align*}
x^TLx =& x^TDx-x^TWx \\
=& {1\over2}( 2\sum_{i=1}^{n}d_{ii}x_i^2 -2 \sum_{i=1}^{n} \sum_{j=1}^{n} x_ix_jw_{ij})\\
=& {1\over2}(\sum_{i=1}^{n}d_{ii}x_i^2 -2 \sum_{i=1}^{n} \sum_{j=1}^{n} x_ix_jw_{ij} + \sum_{j=1}^{n}d_{jj}x_j^2) \\
=&{1\over2}\sum_{i=1}^{n} \sum_{j=1}^{n}w_{ij}(f_i-f_j)^2
\geq 0
\end{align*}
$$
这里忍不住多唠几句，实际上这里也可以用到图的哈密顿算子$\nabla$(我不知道在图谱理论里是不是这个叫法，但是这个符号确实是这么念，也用作梯度)。图的哈密顿算子就是图的关联矩阵的变形，图的关联矩阵以边为行，节点为列，那么对边标号，并且从小到大排列，那么对每一行来说，一条边必然关联着一个两个节点，我们把编号小的节点(也就是主元)记为$1$，编号大的节点记为$-1$，那么就可以到图的哈密顿算子$\nabla$，然后图的拉普拉斯矩阵$L$实际上可以分解为$\nabla^T\nabla$，这种分解其实也叫Cholesky分解()。因为前面我们讲过，拉普拉斯矩阵等价于$\nabla^2$嘛，那么这个结论其实还是挺显然的。从这一角度也可以很容易证明其半正定
$$
x^TLx = x^T \nabla^T \nabla x=(\nabla x)^T\nabla x=||\nabla x||^2\geq 0
$$
那么现在已知拉普拉斯矩阵是半正定的了，就可以得到很有趣的结论，比如特征值非负。而且也注意到这个矩阵是不可逆的，它有一个特征值$0$，但是GCN实际上用到的并不是原始的拉普拉斯矩阵，而是对称归一化的拉普拉斯矩阵，这又是个什么玩意呢？GCN为什么要用这么个玩意呢？

$Define:$对称归一化的拉普拉斯矩阵
$$
L_{sym}=D^{-{1 \over 2}}LD^{-{1 \over 2}}=D^{-{1 \over 2}}(D-W)D^{-{1 \over 2}}=I-D^{-{1 \over 2}}WD^{-{1 \over 2}}
$$
那么有小伙伴一定好奇了？归一化我见多了，这是个什么鬼归一化？它把谁归一了？我们先给出这个对称化矩阵的定义
$$
L_{i,j}^{sym}=
\begin{cases}
1 &if \space i=j \space and \space deg \space(v_i) \neq 0\\
-{1 \over \sqrt{deg(v_i)deg(v_j)}} &if \space i \neq j \space and  \space v_i \space is \space adjacent \space to v_j\\
0 &otherwise
\end{cases}
$$
我们不妨在此给出一个例子，比如如下是一个矩阵通过这种对称归一化的结果
$$
L = \begin{bmatrix}
3 &-1 &-1 &-1\\
-1 &2 &-1 &0\\
-1 &-1 &3 &-1\\
-1 &0 &-1 &2
\end{bmatrix}
\stackrel { 归一化 } \longrightarrow L_{sym} = \begin {bmatrix}
1 & - { 1 \over \sqrt 6 } & - { 1 \over 3} & -{  1 \over \sqrt 6 } \\    
{ -1 \over \sqrt 6 } & 1  & - { 1 \over \sqrt 6 } & 0 \\
{ -1 \over 3 } & -{ 1 \over \sqrt 6 } & 1 & - { 1 \over \sqrt 6 } \\   
{- 1 \over \sqrt 6 } & 0 & - { 1 \over \sqrt6 } & 1
\end{bmatrix}
$$
从这个例子可以明显看出归一化是把对角线归一化，之所以这样归一化是因为注意到未归一化的拉普拉斯矩阵中，只有主对角线上的值具有差异，而其他存在边的位置的特征都是$-1$，也就是说这个矩阵也许不能很好的反映不同结点之间边的特殊性(他们地位是相同的)。而如图的归一化则将不同结点的边之间的信息通过两个节点的度来衡量，进一步加强了节点之间的交互信息。

这里我再啰嗦下，我现在知道这样归一化的作用了，可是这个式子$D^{-{1 \over 2}}LD^{-{1 \over 2}}$是怎么得来的呢？注意到矩阵的左乘表示行的线性组合[^2]，而$D^{-{1 \over 2}}$又是对角阵，那么$D^{-{1 \over 2}}$的第$i$行只是筛选出$L$的第$i$行并对其附加权重$1 \over \sqrt {deg(v_i)}$，右乘则表示列的线性组合，那么同理，$D^{-{1 \over 2}}$的第$j$列只是筛选出$L$的第$j$列并对其附加权重$1 \over \sqrt{deg(v_j)}$。

显然归一化的矩阵也是半正定的，这里的证明其实也不复杂，显然$D^{-{1 \over 2}}= D^{-{1 \over 2}^T}$，这里用哈密顿算子证明比较明了$L_{sym}=D^{-{1 \over 2}^T} \nabla^T \nabla D ^{-{1 \over 2}} = (\nabla D^{-{1 \over 2}})^T  \nabla D^{ -{1 \over 2}}$，那么其半正定性也是显然的。

谢谢阅读，如有错误或者疏漏欢迎指正，码字不易...

[^1]:实对称矩阵$A$称为半正定的，如果二次型$x^TAx$半正定，即对于任意不为$0$的实列向量$x$，都有$x^TAx \geq0$
[^2]:强烈安利MIT的线性代数公开课，这门课从将线性空间看作线性代数的核心，看完犹如醍醐灌顶。