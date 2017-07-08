---
title: 论文阅读: Long Short-Term Memory Based Recurrent Neural Network Architectures for Large Vocabulary Speech Recognition 
date: 2016-07-03 15:15:53
tags: [lstm, Deep Learning]
categories: Deep Learning
---

### LSTM 主要解决的问题

#### Vanishing Gradient Problem

> The main difficulty lies in the well-known vanishing gradient problem which means that the gradiant that is propagated back through the network either decays or grows expontentially.

举个例子

> “The man who wore a wig on his head went inside.”

这句话的主语 man 和谓语 went 之间插入了一个定语从句 who wore a wig on his head，使得两个关键词之间相隔非常远。传统 RNN 不能够捕获这种长距离词汇间的联系，当解析到 went 的时候，主语 man 可能已经被遗忘了，因为 BPTT 通常限制在若干有限步长内。根据链式法则，梯度可以这样计算

$$ \frac{\partial E\_3}{\partial W} = \sum\_{k=0}^{3}\frac{\partial E\_3}{\partial \hat{y}\_3}\frac{\partial \hat{y}\_3}{\partial s\_3} \left(\sum\_{j=k+1}^{3}\frac{\partial s\_j}{\partial s\_{j-1}}\right)\frac{\partial s\_k}{\partial W} $$

tanh 函数的导数在接近 $3$ 和 $-3$ 时趋近于 $0$，在多次矩阵乘法中能使远距离时间步的梯度迅速收缩为 $0$， 意味着几步以前的词对当前状态几乎没有贡献。

一个解决方法是使用 [ReLU](https://en.wikipedia.org/wiki/Rectifier_(neural_networks)) 取代 sigmoid 或 tanh. 因为 ReLU 的导数要么是 0 要么是 1. 更常见的解决方法是使用 LSTM(Long Short-Term Memory) 或 GRU(Gated Recurrent Unit) 来优化 RNN 中隐层的计算。
### LSTM(Long Short-Term Memory)

LSTM 本质上只是改进了 RNN 架构中隐层的计算。如果把 LSTM 的隐层当成一个黑盒的话，那么它和 RNN　并没有什么区别。LSTM 的主要特点是它的门机制 (gate mechanism)： input gate, forget gate, output gate. 这三个门进行同样的运算，都是通过 sigmoid 函数将向量的值压缩为 0 或 1. 0 表示丢弃输入值，1 表示让这个输入值通过门并参与到后续运算中。然后和作为状态值的向量(也就是下文要提到的 cell state)逐元素相乘来决定是否让输入向量对状态值作出更新。

![lstm_gate](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-gate.png)

| Gate |	含义|
| --- | --- |
|input gate |新计算出来的状态向量有多少要加入后续运算中|
|forget gate |过滤先前计算出的状态值，即选择要遗忘哪些信息|
| ouput gate |有多少内部状态信息要暴露给外层神经网络（更高层神经网络或是下一个时间步）|

如果把 input gate 的输出全部设为 1， forget gate 的输出全部设为 0 ， output gate 的输出全部设为 1， 那么这就是个普通的 RNN。而这三个门的作用就是为了控制隐层中有多少信息需要保留，有点类似于遮罩.

colah 的博客逐步描述了 LSTM 的机制，文章中还有非常直观的可视化。

LSTM 主要由下面五个激活函数组成：

$$i\_t = \sigma(W\_{ix} x\_t + W\_{im} m\_{t-1}+W\_{ic} c\_{t-1} + b\_i)$$
$$f\_t = \sigma(W\_{fx} x\_t + W\_{mf} m\_{t-1} + W\_{cf} c\_{t-1} + b\_f)$$
$$c\_t = f\_t \odot c\_{t-1} + i\_t \odot g(W\_{cx}x\_t + W\_{ch}h\_{t-1} + b\_c)$$
$$o\_t = \sigma(W\_{ox}x\_t + W\_{oh}h\_{t-1}+W\_{oc}c\_t + b\_o)$$
$$h\_t = o\_t \odot h(c\_t)$$
$$y\_t = W\_{yh}h\_t + b\_y$$

| 符号 | 	含义|
| --- | --- |
| W 	| 权重矩阵，比如 $W\_{ix}$ 表示 input gate 到输入向量的权重，以此类推|
|b |	偏差向量（bias vector），比如 $b_i$ 是 input gate bias vector|
|$\sigma$ | 	logistic sigmoid 函数|
|i |	input gate|
|f |	forget gate|
|o |	output gate|
|c |	激活向量(cell activation vector)|
|h |	输出激活向量（cell ouput activatioin vector)|
|$\odot$ | 	矩阵逐元素相乘 (elementwise product)|

#### Cell State

![lstm_cellstate](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-C-line.png)

LSTM 的状态参数是为每个隐层节点 (memory cell) 所共享的，换言之就是每个 memory cell 对整个反应链的状态作出修改。colah 将这种 cell state 更新机制类比为传送带。cell state 经过隐层中每个时间步上 memory cell 的更新，而所谓更新就是一些线性运算。

#### Forget Gate

![lstm_forgetgate](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-focus-f.png)

forget gate 的输入包含： 当前时间步的输入向量，上一个时间步中 ouput gate 的输出向量。通过一次 sigmoid 变换映射到 0,1. $f_t$ 决定了旧的状态信息中有多少要遗弃，所以直接 cell state 进行逐元素相乘。

#### Input Gate

![lstm_inputgate](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-focus-i.png)

这个步骤中 LSTM 做的是把新的信息加入 cell state 中，这项工作包含两部分： input gate layer 和 tanh layer. Input gate 层将决定 cell state 中的哪些值需要被更新，然后 tanh 层生成一组候选值 $\tilde{C}_t$ 来取代 cell state 中需要被更新的旧值. 然后将两个向量逐元素相乘，再与 cell state 相加。input gate 的输出向量也是只有 0 和 1，逐元素相乘后只有为 1 的那些状态量会被更新，所以 $i_t$ 起到的其实是一个遮罩的作用。

#### Ouput Gate

![lstm_outputgate](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-focus-o.png)

输出仍然是基于当前时间步的 cell state， 这部分同样由一个 sigmoid layer 和 tanh layer 组成。sigmoid 层决定哪些状态信息要被输出，tanh 层将当前 cell state 压缩到 $(-1,1)$ 区间内。然后执行逐元素相乘，产生这个单元的输出变量。输出变量同时作为下一个单元的 $h_t-1$ 加入到循环中。

### References | 参考资料

[Long Short-Term Memory Based Recurrent Neural Network Architectures for Large Vocabulary Speech Recognition](https://arxiv.org/pdf/1402.1128.pdf)
[Understanding LSTM Networks – colah’s blog](http://colah.github.io/posts/2015-08-Understanding-LSTMs/)
[WILDML: Backpropagation Through Time and Vanishing Gradients](http://www.wildml.com/2015/10/recurrent-neural-networks-tutorial-part-3-backpropagation-through-time-and-vanishing-gradients/)
