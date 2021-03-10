---
title: "Text-Enhanced Representation Learning for Knowledge Graph"
subtitle: "IJCAI-2016"
date: 2019-09-18T18:14:51+08:00
tags: ["知识表示"]
katex: true
---

**Text-Enhanced Representation Learning for Knowledge Graph (IJCAI-2016)** [[paper]](https://www.ijcai.org/Proceedings/16/Papers/187.pdf)

### Motivation
* *TransE、TransH、TransR* 在 *1-N, N-1* 和 *N-N* 关系中的表现不佳
* 图谱结构稀疏导致学习到的表征不准确
### Abstract
利用Word2Vec从文本中学习实体和关系的上下文表征, 然后线性变换到图谱嵌入空间中.学习线性变换的参数, 使得图谱嵌入空间中, 正确三元组的$\|\widehat{\mathbf{h}}+\widehat{\mathbf{r}}-\widehat{\mathbf{t}}\|_{2}^{2}$ 接近于0.
### Input
* 知识图谱 $KG$, 以三元组$(h,r,t)$表示
* 语料库 $D=\\left\\{w_{1}, w_{2}, \cdots, w_{m}\\right\\}$, $w_{i}$为单词, $m$为文本长度
### Method
* ##### Entity Annotation
  使用实体链接工具(AIDA, TAGME, Wikify! 等)来标注$D$中的实体. 标注后得到 ${\rm{D'}}=\\left\\{x_{1}, x_{2}, \cdots, x_{m}\\right\\}$, $x\_{i}$为$KG$中的实体或$D$中的单词. 其中$m'<m$, 因为$D$中的一些词组(连续几个单词)被标为了一个实体.
* ##### Textual Context Embedding
  构建共现网络 $\mathcal{G}=(\mathcal{X}, \mathcal{Y})$ , 其中$x_{i} \in \mathcal{X}$表示单词或实体, $y_{ij} \in \mathcal{Y}$ 表示$x_{i}$和$y_{i}$的共现频率, 共现窗口设置为5.

  按下述步骤计算:

  **pointwise textual context:** 
  <div> 
  $$n\left(x_{i}\right)=\left\{x_{j} | y_{i j}>\theta\right\}$$
  </div> 

  **pairtwise textual context:**  
  <div> 
  $$n\left(x_{i}, x_{j}\right)=\left\{x_{k} | x_{k} \in n\left(x_{i}\right) \cap n\left(x_{j}\right)\right\}$$
  </div> 

  **pointwise textual context embedding:** 
  $$\mathbf{n}\left(x_{i}\right)=\frac{1}{\sum_{x_{j} \in n\left(x_{i}\right)} y_{i j}} \sum_{x_{j} \in n\left(x_{i}\right)} y_{i j} \mathbf{x}_{j}$$
  
  **pairwise textual context embedding:**  
  $${\bf{n}}\left( {{x_i},{x_j}} \right) = {1 \over {\sum\limits_{{x_k} \in n\left( {{x_i},{x_j}} \right)} {\min } \left( {{y_{ik}},{y_{jk}}} \right)}}\sum\limits_{{x_k} \in n\left( {{x_i},{x_j}} \right)} {\min } \left( {{y_{ik}},{y_{jk}}} \right){{\bf{x}}_k}$$



* ##### Entity/Relation Representation Modeling (算法核心)
  **$h$的点嵌入的线性变换**
  $$\widehat{\mathbf{h}}=\mathbf{n}(h) \mathbf{A}+\mathbf{h}$$
  **$t$的点嵌入的线性变换**
  $$\widehat{\mathbf{t}}=\mathbf{n}(t) \mathbf{A}+\mathbf{t}$$
  **$r$的对嵌入的线性变换**
  $$\widehat{\mathbf{r}}=\mathbf{n}(h,t) \mathbf{B}+\mathbf{r}$$
  其中A, B为权重矩阵, h,t和r是偏差.

  **Score Fuction**
  $$f(h, r, t)=\|\widehat{\mathbf{h}}+\widehat{\mathbf{r}}-\widehat{\mathbf{t}}\|_{2}^{2}$$

* ##### Representation Traning 
  优化目标
  $$L=\sum_{(h, r, t) \in \mathcal{S}\left(h^{\prime}, r, t^{\prime}\right) \in \mathcal{S}^{\prime}} \max \left(0, f(h, r, t)+\gamma-f\left(h^{\prime}, r, t^{\prime}\right)\right)$$
  其中$f(h, r, t)$表示正样本,  $f\left(h^{\prime}, r, t^{\prime}\right)$表示负样本.
### 4. Question
* 为什么能解决图谱的稀疏性问题?
  * 利用Word2Vec从文本中学习表征, 而不是从稀疏的图谱中学习表征.
* 为什么能更好处理 *1-N*, *N-1* 和 *N-N* 关系?
  * 例如在 $\widehat{\mathbf{h}}=\mathbf{n}(h) \mathbf{A}+\mathbf{h}$ 中, 对于不同的头实体, $\mathbf{A}$和$\mathbf{h}$是共享的, 因此只要$\mathbf{n}(h)$不同, $\widehat{\mathbf{h}}$ 就不会相同.
  * 当$t$固定时, 即使关系相同, 对于不同的$h$, 对嵌入$\mathbf{n}(h,t)$也不尽相同, 也就是说同一个关系会有多个表征.
  * 由以上两点可知, 在训练时, $\widehat{\mathbf{r}}$ 会不断改变, 因此不会出现TransE中相同的关系下不同头(尾)实体的表示近似相等的情况.
* 同样的关系有不同的表示, 那么要使用某个关系的时候要用哪个表示?
* 在 *1-1* 关系上的性能不如 *TransH* 和 *TransR*
