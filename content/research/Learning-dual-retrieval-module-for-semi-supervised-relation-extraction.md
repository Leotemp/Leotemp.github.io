---
title: "Learning Dual Retrieval Module for Semi-Supervised Relation Extraction"
subtitle: "WWW-2019"
date: 2021-03-12T20:34:50+08:00
tags: ["关系抽取", "半监督学习"]
katex: true
---

**Learning Dual Retrieval Module for Semi-Supervised Relation Extraction (WWW-2019)** [[paper]](https://dl.acm.org/doi/abs/10.1145/3308558.3313573) [[code]](https://github.com/INK-USC/DualRE)

本文将**关系抽取**和**句子检索**视为对偶问题, 通过两者的相互迭代促进, 弥补了self-ensemble和self-training的不足, 在半监督关系抽取上的表现有了明显的提升.
本文行文流畅、创新性强、推导合理、实验充分, 绝对是近年来关系抽取领域中的一篇佳作.

### Motivation
常见的半监督关系抽取方法有两种

1. **self-ensembling method**: 多个模型对无监督样本进行预测, 当大多数模型给出相同的判断时, 为该样本分配该标签
   
    该方法的缺点是不能迭代地增加训练样本, 而少量的有标签样本不足以训练多个模型.

2. **self-training method**: 在每步迭代中, 找出高置信度的样本加入训练集中, 然后用扩大后的训练集继续训练模型

    该方法的缺点是随着伪标签样本的不断加入, 会逐渐发生语义漂移.

本文将**关系抽取**和**句子检索**视为对偶问题. 前者是输入句子, 输出关系标签; 后者是输入关系标签, 输出按照与该标签的相关性排序的句子列表.
本文提出的方法可以看作以上两者的相互融合和相互弥补, 使用一个关系抽取模型和一个检索模型来投票表决, 并将选中样本的交集加入到下一轮的训练.

### Method

![模型框架图](/images/2021-03-12.png)

本框架通过EM算法对两个模型进行迭代更新.

首先使用有标签训练集$L$对关系抽取模型$P_\theta$和检索模型$Q_\phi$进行预训练, 然后迭代进行以下EM步骤:
  
* **挑选样本**: 使用$P_\theta$和$Q_\phi$分别选出高置信度的无标签样本, 然后取其交集$L'$. 将$L'$从无标签集合$U$中移除, 并加入到训练集$L$中

* **E步**: 使用$L$训练$P_\theta$, 定义损失为cross entropy

* **M步**: 使用$L$训练$Q_\phi$, 定义损失为排序学习中的point-wise loss (最大化正例分数, 最小化负例分数)和pair-wise loss (最大化正负例之间的差距)

### Details
虽然算法过程看似简单, 但作者进行了非常有意思的分析和推导. 这些细节也是这篇文章的主要亮点.

1. **样本的选择**: 
   
   在最初的想法中, 作者是想只用检索模型来挑选样本, 但是这样挑选的样本是有偏的. 
   
   <div>
   $$\mathbb{E}_{p_{\theta}(y \mid x)}\left[\nabla_{\theta} \log p_{\theta}(y \mid x)\right]=\mathbb{E}_{p_{\theta}(y \mid x)}\left[\frac{\nabla_{\theta} p_{\theta}(y \mid x)}{p_{\theta}(y \mid x)}\right]=\sum \nabla_{\theta} p_{\theta}(y \mid x)=\nabla_{\theta} \sum p_{\theta}(y \mid x)=\nabla_{\theta} 1=0
   $$
   </div>
   
   由以上推导可得, $\mathbb{E}\_{p\_{\theta}(y \mid x)}\left[\nabla\_{\theta} \log p\_{\theta}(y \mid x)\right]$和$\mathbb{E}\_{q\_{\phi}(x \mid y)}\left[\nabla\_{\phi} \log q\_{\phi}(x \mid y)\right]$都是$0$. 因此, $\mathbb{E}\_{x \in U, y \sim q\_{\phi}(y \mid x)}\left[\nabla\_{\theta} \log p\_{\theta}(y \mid x)\right]=\mathbb{E}\_{x \in U, y \sim (q\_{\phi}(y \mid x) + p\_\theta(y \mid x))}\left[\nabla\_{\theta} \log p\_{\theta}(y \mid x)\right]$. 即, 只用检索模型挑选样本和两个模型一起用, 优化目标是相同的.

2. **样本权重**

    考虑到伪标签样本和真实标签样本混合的话, 其中的错误标签可能会误导模型, 因此为伪标签样本设置样本权重. 
    关系抽取模型和检索模型所挑选样本的权重分别为$p\_\theta(y\mid x)^\alpha$和$q\_\phi(x\mid y)^\beta$.

本模块分为三个部分:

### Experiments

作者在**SemEval**和**TACRED**上进行了多组实验, 实验结果分析如下:

1. **编码器**
   
   测试了LSTM、PCNN和PRNN三种编码器, 其中PRNN性能最佳, 因此以PRNN作为后续实验的编码器.

2. **半监督方法**
   
   测试了Mean-Teacher (Self-Ensemble)、Self-Training和RE-Ensemble (把本框架中的检索模型替换成关系抽取模型, 即两个关系抽取模型的集成方法)三种半监督的Baselines, 结果一致表明: 本方法 > RE-Ensemble > Self-Training > Mean-Teacher.

3. **排序损失**
   
   测试了Point-Wise和Pair-Wise两种检索模型的性能, Point-Wise的结果稍好.

4. **数据量**
   
   有标签数据集越大越好, 无标签数据集不一定越大越好.

5. **样本权重**
   
   $\alpha=0.5$, $\beta=2$时表现最好, $\alpha=0$, $\beta=0$ (即不设置样本权重)时最差.

6. **关系选择**
   
   使用检索模型挑选样本时, 如何选择哪些关系进行输入?
   一种方式是直接使用训练集$L$中的关系分布进行采样. 
   另一种方式是在关系抽取模型所挑选的高置信度样本中, 选择置信度最高的$n$种关系.

   实验表明$n=3$时最好.

7. **挑选样本的质量**

   测试了每轮迭代后模型的表现, 发现随着伪标签样本的加入, Precision不断下降, 而F1-Scores不断上升, 说明了所挑选的样本虽然会带来准确率的下降, 但是召回上升更快, 其质量还是过关的.