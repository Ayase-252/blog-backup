---
title: 机器学习常用概率分布速查
categories: Technology
tags:
  - Machine Learning
  - Probability
date: 2017-05-28 17:25:47
mathjax: true
---


本文收录了一些机器学习中常用的概率分布。其中的大部分摘自《Machine Learning a probalistic perspective》。这篇文章是一篇参考性质的文章，仅罗列结论而不会关心它的来源。本文将会在学习过程中随时更新。

<!-- more -->

{% note info %}
点击本页右下角的图标可以开启目录。
{% endnote %}

--------------------

## 离散分布

### 二项分布与伯努利分布

**二项分布**是\\(n\\)次独立的是非实验成功的个数\\(k\\)服从的分布。记为\\(k \sim \text{Bin}(n,\theta)\\)。其中\\(\theta\\)是单次实验成功的概率。

概率质量函数（pmf）：
$$
Bin(k|n,\theta)=\left(\begin{array}{c}n\\\\k\end{array}\right)\theta^k(1-\theta)^{n-k}
$$
均值：$\text{mean}=\theta$
方差：$\text{var}=n\theta(1-\theta)$

$n=1$时的二项分布称为**伯努利分布**，设随机变量$X$服从伯努利分布，记为$X \sim \text{Ber}(\theta)$。

概率质量函数：
$$
\\text{Ber}(x|\\theta)=\\theta^{\\mathbb{I}(x=1)}(1-\\theta)^{\\mathbb{I}(x=0)}
$$
其中：
$$
\\mathbb{I}(x=n) = \\left\\{
\\begin{eqnarray\*}
&&1 \\quad x=n \\\\
&&0 \\quad \\text{otherwise}
\\end{eqnarray\*}
\\right.
$$

均值：$\text{mean}=\theta$
方差：$\text{var}=\theta(1-\theta)$

{% note info %}
引入$\mathbb{I}$是为了简化公式的表示。比如这里伯努利函数的pmf可以表示为：
$$
\text{Ber}(k|\theta) = \left\\{
\begin{eqnarray\*}
&&\theta \quad &x=1 \\\\
&&1-\theta \quad &x=0
\end{eqnarray\*}
\right.
$$
在公式较为复杂的时候，我们会采用更为简单的方式表示。
{% endnote %}

### 多项分布与类别分布

**多项分布**是二项分布在多个可能结果的实验上的推广。

概率质量函数：
$$
\text{Mu}(\pmb{x}|n,\theta)=\left(
\begin{array}
{c}n \\\\
x\_1,\dots, x\_K
\end{array}\right)\prod\_{j=1}^K\theta^x\_j
$$

其中：
$$
\left(\begin{array}{c}n\\\\x_1,\dots, x_K\end{array}\right) \triangleq \frac{n!}{x_1!x_2!\dots x_K!}
$$

当$n=1$时，多项分布称为**类别分布**。

概率质量函数：
$$
\text{Cat}(x|\theta) = \prod_{j=1}^K\theta^{\mathbb{I}(x_j=1)}
$$

--------------------

## 连续分布
### Beta分布
**Beta分布**是一个在区间$\[0,1\]$上的一个分布。

概率密度函数：
$$
\text{Beta}(x|a,b)=\frac{1}{B(a,b)}x^{a-1}(1-x)^{b-1}
$$
其中$B(a,b) \triangleq \frac{\Gamma(a)\Gamma(b)}{\Gamma(a+b)}$

均值：$\text{mean}=\frac{a}{a+b}$
众数：$\text{mode}=\frac{a-1}{a+b-2}$
方差：$\text{var}=\frac{ab}{(a+b)^2(a+b+1)}$

--------------------
## 多维联合分布
### 多维正态分布
**多维正态分布(Multivariate Gaussian, or Multivariate normal, MVN)**是正态分布在多维的推广。

假设$\pmb{x} \in \mathbb{R}^D$，则概率密度函数为：

$$
\mathcal{N}(\pmb{x}|\pmb{\mu},\pmb{\Sigma}) \triangleq \frac{1}{(2\pi)^{D/2}|\pmb{\Sigma}|^{1/2}}\exp{\left[-\frac{1}{2}(\pmb{x}-\pmb{\mu})^T\pmb{\Sigma}^{-1}(\pmb{x}-\pmb{\mu})\right]}
$$

其中，$\pmb{\mu}=\mathbb{E}[\pmb{x}]\in\mathbb{R}^D$, $\pmb{\Sigma} =\text{cov}[{\pmb{x}}] \in \mathbb{R}^{D \times D}$为协方差矩阵。

### Dirchlet分布
**Dirchlet分布**是Beta分布在多维情况下的一个推广。由于**Direchlet分布**仅在_概率单纯形_的面上有分布，所以被认为是分布的分布。

定义概率单纯形$S\_k=\\{x:0 \le x\_k\le 1, \sum_{x=1}^K x\_k=1\\}$。

概率密度函数：
$$
\text{Dir}(x|\pmb{\alpha}) \triangleq \frac{1}{B(\pmb{a})} \prod_{k=1}^K x_k^{\alpha_k-1} \mathbb{I}(\pmb{x} \in S_k)
$$

其中
$$
B(\pmb{a}) \triangleq \frac{\prod_{k=1}^K \Gamma(\alpha_k)}{\Gamma(\alpha_0)}
$$

{% note info %}
上述概率密度函数也可以写成
$$
\\text{Dir}(x|\\pmb{\\alpha}) \\triangleq
\\left\\{
\begin{eqnarray\*}
&&\frac{1}{B(\pmb{a})} \prod\_{k=1}^K x\_k^{\alpha\_k-1} \quad & \text{if}\sum\_{x=1}^K x\_k=1 \\\\
&&0 \quad & \text{otherwise}
\end{eqnarray\*}
\right.
$$
{% endnote %}

均值：$\mathbb{E}\[x_k\]=\frac{\alpha_k}{\alpha_0}$
众数：$\text{mode}\[x_k\]=\frac{\alpha_k-1}{\alpha_0-K}$
方差：$\text{var}\[x_k\]=\frac{\alpha_k(\alpha_0-\alpha_k)}{\alpha_0^2(\alpha_0+1)}$
