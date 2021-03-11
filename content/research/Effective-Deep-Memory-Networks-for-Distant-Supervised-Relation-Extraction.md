---
title: "Effective Deep Memory Networks for Distant Supervised Relation Extraction"
subtitle: "IJCAI-2017"
date: 2020-03-11T22:39:40+08:00
tags: ["关系抽取", "远程监督", "记忆网络"]
---

**Effective Deep Memory Networks for Distant Supervised Relation Extraction (IJCAI-2017)** [[paper]](https://www.ijcai.org/Proceedings/2017/0559.pdf)

本文的出发点很好, 考虑到了句子中不同词的重要性和关系标签之间的依赖两点. 但是就模型架构而言, 与其说是Memory Networks, 不如说更像是多层的注意力网络. 考虑到文章发表时Transformers还没有大行其道 (虽然注意力机制已经非常流行), 本文还是比较有创新性的.

### Motivation
1. 句子中不同的词对于目标实体对的重要性是不同的
2. 不同关系之间的依赖(如包含, 冲突等), 对于某些隐式关系的推断很有帮助
   
### Method

![模型框架图](/images/feng2019effective.png)

模型主要分为三块:

1. 词级别
   
   在左下角模块里, 使用多层注意力模块来融合上下文信息, 即**Motivation 1**. 
   在第一层里, 以实体对作为输入, 然后计算**每个词**对**该实体对**的注意力加权和, 作为该层的输出.
   此后每层均以上一层的输出作为输入, 然后输出该层中上下文单词的注意力加权和.
   
   最终, 将注意力模块的输出与CNN编码的句子向量拼接, 构成了本模型的句子向量.
   
   文中4.4实验部分表明, 层数为6时模型表现最佳.

2. 句子级别
   
   在右下角模块里, 包含了两层不同的注意力模块.

    * 在第一层中, 计算**每条句子**对**该关系**的注意力加权和, 作为该层的输出. 即DSRE里最常见的Selective Attention.
    * 在第二层中, 计算**其他关系**对**该关系**的注意力加权和, 可看作关系之间的自注意力,即**Motivation 2**. 
  
   最终为每个关系输出一个向量表示.

3. 分类器
   
   由于是multi-label classification, 因此分别对每个关系向量进行sigmoid分类.

### Experiment

实验在过滤版NYT数据集上进行, 结果没有想象中好, 只比PCNN+ATT好了一丢丢, 但是模型复杂度却要高很多. 虽然Motivation很好, 但是这种简单的注意力模块并没有把Motivation完全挖掘出来. 个人感觉这个Motivation, 包括Memory Networks和关系依赖, 都还有很大的挖掘空间.