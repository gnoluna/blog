---
title: Introduction to RNNs 
date: 2016-06-27 15:02:57
tags: [Deep Leaning, RNN]
categories: Deep Learning
---

### Abstract | 简介
#### Language Model | 语言模型

语言模型主要解决两件事

- 能够对计算机生成的句子进行打分，分数高低取决于这个句子有多大概率在现实中出现。分数作为一种语法和语义上的度量
- 能够产生新的句子。比如用莎士比亚作品作为语料库，可以产生莎士比亚文风的句子。

#### Recurrent Nerual Network | 循环神经网络

关于统计语言模型， Bengio 曾提出过一个使用定长文本训练的前馈神经网络。但这个模型的主要问题是前馈网络依赖于一个预设的文本长度，不能针对不同语境作出动态变化。循环神经网络针对这点作出改进：RNN 不限制文本的长度，而且文本信息能够在网络中循环任意时间。

RNN 的主要思想是充分利用序列化信息之间时间上的关联。传统神经网络将所有输入信息作为相互独立的信息来看待，但在很多情况下（尤其是自然语言处理的问题上）并不是个好的想法。因为语境信息对于语言模型是至关重要的，一个词在某个位置出现的概率很大程度上受前文中其他词的影响。RNN 之所以叫循环神经网络是因为它对序列中每个词的处理是基于前面词的状态。可以理解为 RNN 是一个带有记忆的模型，它能够缓存之前的运算结果，这些状态结果会一直影响接下来的神经元运算。理论上 RNN 能够充分利用任意长度序列的信息，但出于对效率因素的考虑只回看若干步。

[trask 博客](https://iamtrask.github.io/2015/11/15/anyone-can-code-lstm/)中的这个 gif 图片能够很直观得看出 RNN 的运行过程。

![RNN_GIF](/uploads/recurrence_gif.gif)

图中展示了 4 个时间步 (timestep) 中隐层 (hidden layer) 的变化。第一个时间步中隐层的计算完全依赖于输入；第二个时间步中隐层的输入来源于第一个隐层的输出和第二个时间步的输入。到了第四个时间步，隐层已经包含了所有输入信息。到第五个时间步，隐层就要决定哪些信息要留下，哪些信息要被覆盖。因此隐层节点越多就能记住更长的时间段内的信息，隐层节点数量意味着循环神经网络的记忆容量。

这样的架构中，输入不仅仅是单纯的输入，输入信息不停地更新 RNN 的 “memory”， 输出是基于某个状态下的 “memory”. 即便某个时间步内没有任何输入, memory 也会随着时间动态发生变化。

先通过一个最基础的 RNN 了解一下它的架构。 Simple recurrent neural network (or Elman Network）

| 符号表示 |	含义 |
| --- | --- |
|$x(t)$ 	|t 时间步的输入信息|
|$y(t)$ |t 时间步的输出信息|
|$s(t)$ |	t 时间步的隐层（神经网络的状态）|
|$U$ 	|隐层和隐层之间的传导矩阵 synapse_h|
|$V$ 	|隐层和输出层之间的传导矩阵 synapse_1|
|$f(z)$ 	|sigmoid 激活函数|
|$g(z_m)$ 	|softmax 函数|

$$ x(t) = w(t) + s(t-1) $$

$$ s_j(t) = f\left(\sum_i x_i(t)u_ij \right)$$

$$ y_k(t) = g\left(\sum_j s_j(t)v_kj \right)$$

$$ s.t. f(z) = \frac{1}{1+e^{-z}} , g(z\_m) = \frac{e^{z\_m}}{\sum\_{k}e^{z\_k}}$$

$s(0)$ 被初始化为值很小的向量，比如 0.1 。处理大量数据时初始化不是关键因素。隐层节点数量大约在 30-500 之间，反应了训练数据的数量。

### Training

神经网络大约要经过 10-20 个 epoch 才能达到比较好的收敛效果。BP 算法和 SGD(Stochastic Gradient Descent) 是比较常见的训练方法。设置 learning rate 为 0.1, 每完成一个 epoch 就要在验证集上进行测试，通过观察验证集的对数似然来判断模型是否收敛。如果模型没有显著提高，就将 learning rate 减半，接着进行下一个 epoch .

#### Backpropagation Through Time (BPTT)

gif 动态图来自[trask 博客](https://iamtrask.github.io/2015/11/15/anyone-can-code-lstm/)

![bptt](/uploads/backprop_through_time.gif)

BPTT 是 BP 算法针对循环神经网络的一个改进版本。BPTT 的参数被所有时间步共享。每个输出的梯度依赖于之前的时间步，其实就是对应微积分中的链式法则。例如为了计算 $t = 4$ 时的梯度，需要反向传播 3 次更新权重。BPTT 算法仍有不能解决的问题，比如词汇间的长期依赖(long-term dependency). 两个词隔得很远，但它们之间确有联系。在时间步到达第二个词的时候，第一个词已经被遗忘了。这就是所谓的梯度消失/梯度爆炸 (vanishing/exploding gradient) 问题。针对这个问题已经有了好的解决方法，如 [LSTM](https://en.wikipedia.org/wiki/Long_short-term_memory).  LSTM 的架构原则上和一般 RNN 没有什么不同，但在隐层的计算上用了一些技巧，能够选择哪些重要的信息值得记忆。对于捕获词汇间的长期依赖有很大的贡献。

### Reference | 参考资料

[Mikolov: Recurrent neural network based language model](http://www.fit.vutbr.cz/research/groups/speech/publi/2010/mikolov_interspeech2010_IS100722.pdf)
[Mikolov: Extensions of recurrent neural network language model](http://www.fit.vutbr.cz/research/groups/speech/publi/2011/mikolov_icassp2011_presentation_rnnlm-extension.pdf)
[WildML:Recurrent Neural Networks Tutorial, Part 1 – Introduction to RNNs](http://www.wildml.com/2015/09/recurrent-neural-networks-tutorial-part-1-introduction-to-rnns/)
[trask: Anyone Can Learn To Code an LSTM-RNN in Python (Part 1: RNN)](https://iamtrask.github.io/2015/11/15/anyone-can-code-lstm/)
