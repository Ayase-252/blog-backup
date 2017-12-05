---
title: 从零开始的CNN——卷积与CNN的Motivation
categories: "Notes"
tags:
  - Convolution Neural Network
  - Deep Learning
mathjax: true
---
在学习深度学习时，经常会看到一个非常熟悉的缩写——CNN出现。可是在这里，它的意义可不是电视台哦。它就是在图像识别领域大名鼎鼎的卷积神经网络(Convolution Nerual Network, CNN)。在2012年的ImageNet图像识别竞赛中，利用CNN制作的分类器[1]将分类错误率降低到了15.3%，得到了学术界和工业界的瞩目。当然，CNN不止可以被应用在图像识别中。在语音处理、视频处理中也有相当的应用。

<!--more-->

# 什么是CNN

卷积神经网络，从结构来说就是至少运用了一层卷积的神经网络。所以问题来了，卷积这玩意到底是什么？

在引出卷积之前，我们先来看一种概率学中非常常见的操作——求随机变量$x$的期望。如果连续随机变量服从某个概率模型，那么随机变量的期望为

## What

### 参考文献
[1]	A. Krizhevsky, I. Sutskever, and G. E. Hinton, "Imagenet classification with deep convolutional neural networks," in Advances in neural information processing systems, 2012, pp. 1097-1105.