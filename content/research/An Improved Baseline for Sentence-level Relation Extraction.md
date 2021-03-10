---
title: "An Improved Baseline for Sentence-level Relation Extraction"
subtitle: "Arxiv-2021"
tags: ["关系抽取"]
date: 2021-03-10T13:07:52+08:00
draft: true
katex: true
---

**An Improved Baseline for Sentence-level Relation Extraction (Arxiv-2021)** [[paper]](https://arxiv.org/abs/2102.01373)

本文聚焦于当前关系抽取实践中两个非常重要却经常被忽视的部分: **实体的表示方式**和**NA类的处理**. 通篇语言流畅, 而且叙述结构非常清晰, 个人认为是今年最好的一篇关系抽取文章.

### Entity Representation

替换实体对的方式:

* **Entity Mask**: [SUBJ-类型名], [OBJ-类型名]
* **Entity Marker**: [E1]头实体[/E1], [E2]尾实体[/E2]
* **Entity Marker(punct)**: @头实体@, #尾实体#
* **Type Entity Marker**: [E1-类型名]头实体[/E1-类型名], [E2-类型名]尾实体[/E2-类型名]
* **Type Entity Marker(punct)**: @\*类型名\*头实体@, #^类型名^尾实体#

实验表明后两种表现最好.

下面是作者关于训练时是否应该掩盖实体名的分析:

>有人认为训练时不掩盖实体名, 会导致测试集的标签泄漏, 而且无法扩展到新实体. 但我认为, 实体名提供了重要的实体信息, 而且, 如果实体名不能使用, 那么那些基于外部三元组知识的方法也不该使用, 因为他们同样泄露了目标实体的信息.

为了证实自己的观点, 作者将测试集中包含训练实体的句子过滤掉, 最终结果表明, 非mask的仍然优于mask.

### Confidence-based Classification

作者认为, 不应该将NA视作关系分类中的一个独立类别, 因为NA中包含着无数种不同语义的实例, 很难由一个类别建模.

本方法设立阈值进行分类, 若预测分数低于阈值, 则被视作NA. 类似于Openset Classification和Out-of-distribution Detection.

Loss的设计:

1. NA类具有极低的置信度
2. 正关系具有极高的置信度

2可以通过Cross Entropy实现. 但对于1, 直接降低置信度是很难优化的, 因为最大预测概率所对应的关系也会被更新, 因此, 提出了最小化NA类的置信度代理:

<div>
$$\begin{aligned}
    c_{\text {sup }} &=\sum_{r \in \mathcal{R}} \boldsymbol{p}_{r}^{2} \\
    \mathcal{L}_{\text {conf }} &=\log \left(1-c_{\text {sup }}\right)
\end{aligned}$$
</div>

由计算可知, 置信度$c=\max_{r\in R}p_{r}\leq C_{\text{sup }}$, 最小化$\mathcal{L}_{\text {conf }}$相当于最小化$c$, 使得训练更稳定.

对关系$r$的logit值$l_r$求导:

<div>
$$\frac{\partial \mathcal{L}_{\mathrm{conf}}}{l_{r}}=-\frac{2 \boldsymbol{p}_{r}\left(\boldsymbol{p}_{r}-\sum_{r \in \mathcal{R}} \boldsymbol{p}_{r}^{2}\right)}{1-\sum_{r \in \mathcal{R}} \boldsymbol{p}_{r}^{2}}$$
</div>

由此可知:

1. 当$\boldsymbol{p}\_{r}=\frac{1}{\mid \mathcal{R}]}$, $\forall r \in \mathcal{R}$, $\mathcal{L}_{\text {conf }}$最小, 即此时各关系为均匀分布.
2. 训练实例的$\mathcal{L}\_{\text {conf }}$惩罚为$\frac{1}{1-\sum_{r \in \mathcal{R}} \boldsymbol{p}_{r}^{2}}$, 因此置信度高的实例会受到更多惩罚.

最终, 结合**Type Entity Marker(punct)**和**Confidence-based Classification**的方法在TACRED和SemEval 2010 Task 8上都取得了SOTA结果.