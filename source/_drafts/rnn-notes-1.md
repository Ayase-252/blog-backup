---
title: RNN学习笔记（I）：Recurrent Nerual Network 
categories: "Notes"
tags:
  - Recurrent Nerual Network
  - Deep Learning
mathjax: true
---

Recurrent Nerual Network，RNN，又称时间递归神经网络。用于处理可以形成一定序列的数据。
在RNN中，相邻的时刻的神经元之间存在连结。根据目的不同，连结的形式有一些区别。

最通用的一种形式如图1所示，在两个**相邻时刻**的隐含层$h(t-1)$、$h(t)$之间存在连接。其模型如下：

$$
\begin{align}
a^{(t)} &= Wh^{(t-1)} + Ux^{(t)} + b \\\\
h^{(t)} &= \tanh(a^{(t)})  \\\\
o^{(t)} &= Vh^{(t)} + c \\\\
\hat{y}^{(t)} &= \text{softmax}(o^{(t)})
\end{align}
$$

![图1 隐含层相连的RNN](/images/2017/12/rnn-f1.png)
<center>图1 隐含层相连的RNN（copy from "Deep learning" by Ian Goodfellow et al.）</center>

除此之外，还有前一时刻的输出层连接下一时刻的隐含层这样的变形。这种变形由于解开了两个隐含层之间的耦合，可以通过并行化加速计算。
![图2 输出层与隐含层相连的RNN](/images/2017/12/rnn-f2.png)
<center>图2 输出层与隐含层相连的RNN（copy from "Deep learning" by Ian Goodfellow et al.）</center>
