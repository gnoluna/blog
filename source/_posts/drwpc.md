---
title: "论文阅读: Distributed Representation of Words and Phrases and their Compositionality"
date: 2016-06-23 14:41:10
tags: [nlp, word2vec]
categories: Deep Learning
---


### 简介

这篇文章是 [Word2Vec](https://zh.wikipedia.org/zh/Word2vec) 作者关于 [Mikolov](https://scholar.google.com/citations?user=oBu8kMMAAAAJ&hl=zh-CN) 的 Skip-gram 模型提出的改进和扩展，主要是提升训练速度和词向量的质量。

### 关于 Skip-gram

Skip-gram 能够高效地从大量文本中训练出词向量。与其他神经网络架构模型不同的是，它不涉及矩阵乘法，大大提升了训练效率。经过学习后的词向量甚至能够进行线性运算。比如： vec(“Madrid”) - vec(“Spain”) + vec(“France”) is closer to vec(“Paris”)

![skipgram](/uploads/skipgram.png)

Skip-gram 的训练目标是能够预测文本中某个词周围可能出现的词。比如，现在有一份文档（去掉标点符号）由 $T$ 个词组成，$w\_1,w\_2,w\_3,…,w\_T$， skip-gram 的目标函数就是最大化它们的平均对数概率

$$ \frac{1}{T} \sum\_{t=1}^{T} \sum\_{-c \leq j \leq c} \log p(w\_{t+j} | w\_t) $$

c 是上下文长度，也就是中心词 $w\_t$ 的前后各 c 个词， c 越大意味着训练样本越多，精度也就越高，但时间开销大。以 softmax 的方式计算 $p(w\_O|w\_I)$

$$ p(w\_O|w\_I) = \frac{\exp({v’\_{w\_O}}^\top v\_{w\_I})}{\sum\_{w=1}^{W} \exp({v’\_{w}}^\top v\\_{w\_I})} $$

$W$ 是总词汇量，$v\_w$ 和 $v’\_w$ 是词 $w$ 的输入和输出变量。然而这个公式是不现实的，因为计算开销随着总词汇量 $W$ 而增长。

### 基于 Skip-gram 的扩展

#### Hierarchical Softmax

skip-gram 中需要 $W$ 个输出节点来计算概率分布，而 Hierarchical Softmax 只需要大约 $\log\_2(W)$ 个输出节点，因为它的输出层采用了二叉树的表现形式。每个节点是一个词，它的子节点存储是与这个词相关的词。节点中只存储该词与子节点词的相对概率。总词汇中的每个词都有一条从二叉树根部到自身的路径。用 $n(w,j)$ 来表示从根节点到 w 词这条路径上的第 j 个节点。用 $ch(n)$ 来表示 $n$ 的任意一个子节点。

那么 Hierarchical Softmax 可以表示为

$$ p(w | w\_I) = \prod\_{j=1}^{L(w)-1} \sigma ([[ n(w,j+1) = ch(n(w,j))]]\cdot {v’\_{n(w,j)}}^\top v\_{w\_I}) $$

$$ s.t. \sigma (x)= \frac{1}{1+\exp(-x)} $$

$ \sum\_{w=1}^{W}p(w|w\_I) = 1 $ 是可以验证的，这样一来 $ \log p(w\_O|w\_I)$ 和 $\nabla \log p(w\_O|w\_I)$ 的计算开销就和 $L(W\_O)$ 成比例。而 $L(W\_O)$ 不会超过 $\log W$. Word2Vec 中使用的是哈夫曼二叉树，高频词汇编码较短，低频词汇编码较长。

#### Negative Sampling

NCE(Noise Contrastive Estimation):好的模型能够通过逻辑斯蒂回归将数据从噪声中区别出来

因为 skip-gram 更关注于学习高质量的词向量表达，所以可以在保证词向量质量的前提下对 NCE 进行简化。于是定义了 NEG(Negative Sampling)

$$ \log \sigma({v’\_{w\_O}}^\top v\_{w\_I} )+ \sum\_{i=1}^{k}\mathbb{E}\_{w\_i \sim P\_n(w)}[\log \sigma(-{v’\_{w\_i}}^\top v\_{w\_I})] $$

这个公式用来替代 skip-gram 目标函数中的 $\log p(w\_O|w\_I)$. $ P\_n(w)$ 是词 $n$ 周围的噪声词分布（NCE 和 NEG 都有这个参数）。这个公式是用逻辑斯蒂回归从 $k$ 个负例中区别出目标词汇。对于小的训练集 $k$ 的最佳取值是 5-20， 对于大的训练集 $k$ 的取值会更小，大概 2-5. NCE 和 NEG 的区别在于 NCE 在计算时需要样本和噪音分布的数值概率，而 NEG 只需要样本。

#### Subsampling

subsampling 用于解决这样的问题：在英文中有大量高频词诸如 “in”, “the”, “a”, 这些词提供的信息量非常少。比如 “Paris” 和 “France” 的共现率明显比 “France” 和 “The” 的共现率低，但 “Paris” 和 “France” 明显更有用。

为了解决高频和低频词汇间的不平衡，每个词汇的频率有一定概率被丢弃，概率由公式得出：

$$ P(w\_i) = 1 - \sqrt{\frac{t}{f(w\_i)}}$$

$ f(w\_i)$ 是词 $w\_i$ 的概率. $t$ 是参数，作为淘汰词的门限，通常情况下约为 $10^{-5}$. 当词汇的频率高于 $t$ 就有一定概率被淘汰掉。

### 处理 Phrase

这部分主要解决专有名词的问题，比如期刊杂志的名字 “New York Times” 并不是单词 “New York” 和 “Times” 词义的简单组合，所以要把它们当成一个词组来看待。作者用了一种简单的数据驱动方式来识别这些专有名词。对于两个词 $w\_i, w\_j$：

$$ score(w\_i, w\_j) = \frac{count(w\_i w\_j) - \delta}{count(w\_i) \times count(w\_j)}$$

其中 $\delta$ 是协同系数，用来避免太多无关的词被组合到一起。

### 资源下载

[Distributed Representation of Words and Phrases and their Compositionality](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0ahUKEwjf3prT0b3NAhVI4mMKHVQCCwAQFggeMAA&url=https%3A%2F%2Fpapers.nips.cc%2Fpaper%2F5021-distributed-representations-of-words-and-phrases-and-their-compositionality.pdf&usg=AFQjCNFvn2t3S41dxIocYbx5EpeOwmjXVQ&sig2=4bbJwW1O7AnHYZfFbA2pIQ)



