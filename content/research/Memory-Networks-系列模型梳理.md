---
title: "Memory Networks 系列模型梳理"
date: 2021-03-14T10:32:06+08:00
tags: ["记忆网络"]
katex: true
---

本文对Memory Networks系列文章进行总结, 为了不局限于在某个领域的应用, 不赘述模型的具体操作步骤, 而对该系列模型的中心思想角度进行梳理.

传统的神经网络模型(如CNN、RNN)等使用隐藏层来记忆所有的训练数据信息, 这种做法存在以下不足:

* 多个隐藏层的堆叠, 使得输出的表示过于抽象, 难以表示原有数据的全部信息
* 经过多次迭代更新后, 模型只能记住训练数据主要的抽象特征, 长尾数据往往会被遗忘
* 模型的计算过程可以被设计, 但其记忆过程却不受人为控制. 训练集的偏差可能被模型认为是重要特征

上述不足的原因都可以归咎于传统神经网络过小的记忆容量. 为此, Memory Networks被提出, 跟传统神经网络不同的是, 它使用外部的记忆单元来显式存储所需要记忆的信息, 并通过读写操作来实现对记忆单元的更新.
**传统神经网络可以看作是一块CPU, 推理运算能力极强, 却只能使用容量极小的Cache来存储信息. 而Memory Networks则是CPU搭配内存, CPU负责推理运算, 内存用于存储大规模的信息, 两者各司其职.**

### Memory Networks
**Memory Networks (ICLR-2015)** [[paper]](https://arxiv.org/abs/1410.3916)
![Memory Networks](/images/2021-03-14-1.png)

如图, Memory Networks由一个记忆单元M和4个计算模块I、G、O、R组成.

* **Memory (M)**: 外部记忆单元, 通过内部的N个Memory Slots (m<sub>1</sub>, ..., m<sub>N</sub>) 来存储信息 (N可以非常大)
* **Input (I)**: 将输入X编码为向量形式
* **Generalization (G)**: 将输入X中所需记忆的信息写入M
* **Output (O)**: 根据输入X, 从M中读取所需的信息
* **Response (R)**: 对O的输出进行解码, 得到模型的输出

更为常见的形式是用于QA问答中(如下图), 其输入是一段参考文本X和一个问题q, 要求模型从X中得到q的答案.

![Memory Networks on QA](/images/2021-03-14-2.png)

在之前解读的文章[**Effective Deep Memory Networks for Distant Supervised Relation Extraction**](/research/effective-deep-memory-networks-for-distant-supervised-relation-extraction/)中, 使用的就是这个形式的模型. 
在该模型中, 以目标实体对作为q, 以上下文单词作为X.

### End-To-End Memory Networks
**End-To-End Memory Networks (NIPS-2015)** [[paper]](https://dl.acm.org/doi/abs/10.5555/2969442.2969512)
![End-To-End Memory Networks](/images/2021-03-14-3.png)

主要介绍本模型对于前一个Memory Networks在想法上的改进:

1. **QKV形式的注意力机制**
   
   如上图(a)所示, M中通过Key-Value形式(即图中的Input和Output模块)对信息进行存储.
   使用q来查询M的输出时, 采用的就是QKV形式的注意力计算.
   按照原文的说法, 这种加权和的形式比原先的递归计算形式更利于反向传播.
   但本质上, 将m划分为key和value两个部分, key用于查询与query的相关程度, 而value则用于存储记忆内容, 两个部分各司其职, 分别负责推理与记忆, 使得模型更容易学习, 可解释性也更强.

2. **多层记忆单元 (Multi-hop)**
   
   如上图(b)所示, 记忆单元从单层被扩展到多层. 通过多层的QKV计算得到最终的输出. 
   
由此也可以看出, 这时候的多层QKV注意力机制已经出现了`Attention is all you need`的雏形了 (差别在于MLP、multi-head和self-attention).
💥罗马果然不是一天建成的.

### Key-Value Memory Networks
**Key-Value Memory Networks for Directly Reading Documents (ACL-2016)** [[paper]](https://www.aclweb.org/anthology/D16-1147.pdf)
![Key-Value Memory Networks](/images/2021-03-14-4.png)

其实本模型的KV Memory, 在前一个End-To-End Memory Networks中已经出现了, 只是在前一个模型中被定义为Input-Output, 而在本模型中正式命名为Key-Value.
主要的区别在于, End-To-End Memory Networks中的key和value都来自于同样的输入, 只是经过不同的Embedding矩阵(类似于Transformer). 
而Key-Value Memory Networks是为了更加直接地引进外部知识(如文本、知识库等)而设计的, 其key和value可以根据知识形式的设计, 来自于不同的输入(最简单的例子, key表示文本标题, 而value表示文本内容).

### Dynamic Memory Networks
**Ask Me Anything: Dynamic Memory Networks for Natural Language Processing (JMLR-2016)** [[paper]](http://proceedings.mlr.press/v48/kumar16.pdf)

本模型主要是针对End-To-End Memory Networks的改进, 之前的Memory Networks每层中的记忆值都是固定的(通过对Text进行Embedding得到, 与Question无关), 而本模型借助RNN模型的递归输出机制, 可以根据Question的不同而得到不同的记忆值, 故谓之**动态记忆**.

![Dynamic Memory Networks](/images/2021-03-14-5.png)

个人感觉原文对于计算过程的描述过于繁杂(主要是公式符号与图示不一致), 下边简单介绍一下记忆模块中的计算过程. 其中最重要的记忆模块(Episodic Memory Module)可以看作是多层GRU.

* $\mathbf{e}^i_t$表示GRU的第$i$层第$t$步的隐藏层参数($\mathbf{e}^0_t=\mathbf{s}_t$, 其中$\mathbf{s}_t$表示第$t$条句子在Input Module中得到的嵌入)

* $\mathbf{m}^i=\mathbf{e}^i_T$(即第$i$层最后一步的隐藏层输出)

* $\mathbf{q}$表示输入的问题的嵌入.

**第i层第t步的门控值**: 

$$g^i_t = G(\mathbf{e}^{i-1}_t, \mathbf{m}^{i-1}, \mathbf{q})$$ 

其中$G$为特征拼接后经过两层MLP和sigmoid归一化的门控函数

**第i层第t步的隐藏层**: 
$$\mathbf{e}^i_t = g^i_t \cdot GRU(\mathbf{e}^{i-1}_t, \mathbf{e}^i\_{t-1}) + (1 - g^i_t) \cdot \mathbf{e}^i\_{t-1}$$

### Recurrent Entity Networks
**Tracking the World State with Recurrent Entity Networks (ICLR-2017)** [[paper]](https://arxiv.org/abs/1612.03969)

前述的所有Memory Networks, 存储的都是当前的输入, 而并没有像内存一样, 长期地记忆真实世界里的一些东西.
而这正是Recurrent Entity Networks创新的地方, 它的每个记忆单元都对应真实世界里的一个实体, 记忆单元之间相互独立, key用于标识实体, value则用于表示实体属性.
从某种角度上来说, 之前的Memory Networks更像是各种Seq2Seq或Attention Networks的变种, 而Recurrent Entity Networks更加贴近"记忆"这个概念.

![Recurrent Entity Networks](/images/2021-03-14-6.png)

下边介绍如何对第$j$个记忆单元进行更新. 注意由于key只用来标识, 真正的记忆内容在value中, 因此只更新value中的内容, key自始至终都不变.

* $\mathbf{w}_j$表示第$j$个记忆单元的key
* $\mathbf{h}_j$表示第$j$个记忆单元的value
* $\mathbf{s}_t$表示第$t$条句子

对第$j$个记忆单元的value值进行更新的步骤为:

<div>
$$
\begin{array}{l}
g_{j} \leftarrow \sigma\left(\mathbf{s}_{t}^{T} \mathbf{h}_{j}+\mathbf{s}_{t}^{T} \mathbf{w}_{j}\right) \\
\tilde{\mathbf{h}}_{j} \leftarrow \phi\left(\mathbf{U} \mathbf{h}_{j}+\mathbf{V} \mathbf{w}_{j}+\mathbf{W} \mathbf{s}_{t}\right) \\
\mathbf{h}_{j} \leftarrow \mathbf{h}_{j}+g_{j} \odot \tilde{\mathbf{h}_{j}} \\
\mathbf{h}_{j} \leftarrow \frac{\mathbf{h}_{j}}{\left\|\mathbf{h}_{j}\right\|}
\end{array}
$$
</div>

### Gated End-to-End Memory Networks
**Gated End-to-End Memory Networks (ACL-2017)** [[paper]](https://www.aclweb.org/anthology/E17-1001)
![Gated End-to-End Memory Networks](/images/2021-03-14-7.png)

此论文的目的只有一个: 通过最少的步骤, 将End-To-End Memory Networks改造为动态记忆网络.
为此, 它将多层记忆之间的传递, 改造成了门控形式. 

* $\mathbf{u}^k$为第$k$层的输入表示
* $\mathbf{o}^k$为第$k$层记忆单元的注意力加权和

改造前:

$$\mathbf{o}^{k+1} = \mathbf{o}^{k} + \mathbf{u}^{k}$$

改造后:

<div>
$$
\begin{aligned}
\mathbf{T}^{k}\left(\mathbf{u}^{k}\right) &=\sigma\left(\mathbf{W}_{T}^{k} \mathbf{u}^{k}+\mathbf{b}_{T}^{k}\right) \\
\mathbf{u}^{k+1} &=\mathbf{o}^{k} \odot \mathbf{T}^{k}\left(\mathbf{u}^{k}\right)+\mathbf{u}^{k} \odot\left(1-\mathbf{T}^{k}\left(\mathbf{u}^{k}\right)\right)
\end{aligned}
$$
</div>

### Working Memory Networks
**Working Memory Networks: Augmenting Memory Networks with a Relational Reasoning Module (ACL-2018)** [[paper]](https://www.aclweb.org/anthology/P18-1092/)

还记得上面说到End-To-End Memory Networks中已经有了Transformer的影子.
时间来到2018年, `Attention is all you need`已经卷席了整个NLP领域. 
这篇文章将multi-head attention引入到Memory Networks中, 同样是在多层记忆处下手.

![Working Memory Networks](/images/2021-03-14-8.png)

与原始的End-To-End Memory Networks中通过QKV计算得到$\mathbf{o}\_i$一样, 此处以$f_t(\mathbf{o}\_{i-1})$作为Q ($f_t$为MLP层, $\mathbf{o}\_{0}=\mathbf{u}$) , 而每个记忆单元都有多头的K和V, 然后完全按照multi-head attention的计算过程, 得到注意力输出$\mathbf{o}_{i}$. 

疑惑的是这里只用到了堆叠的multi-head attention, 却没有skip-connection, 在multi-hops上不会出现梯度消失吗?

### 总结

记忆网络可以分为静态记忆网络([Memory Networks](#memory-networks), [End-To-End Memory Networks](#end-to-end-memory-networks), [Key-Value Memory Networks](#key-value-memory-networks))和动态记忆网络([Dynamic Memory Networks](#dynamic-memory-networks), [Recurrent Entity Networks](#recurrent-entity-networks), [Gated End-to-End Memory Networks](#gated-end-to-end-memory-networks), [Working Memory Networks](#working-memory-networks)).

静态记忆网络的记忆单元一旦写入, 就不会进行更新, 改进点主要在于读取部分(如递归读取、注意力加权等).

而动态记忆网络会随着输入的不同, 记忆单元的内容也会发生改变, 改进点主要在于对记忆单元的更新, 也就是各种门控函数的设计.

### 致谢

知乎专栏: [记忆网络-Memory Network](https://www.zhihu.com/column/c_129532277)