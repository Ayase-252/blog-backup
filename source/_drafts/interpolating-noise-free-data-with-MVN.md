---
title: 利用多元正态分布进行估计
categories: "Theory"
tags:
  - Machine Learning
  - Multivariate Normal Distribution
mathjax: true
---

在日常生活中，常常有根据一些数据去推断另外一些数据的问题。如小学的时候做的一些看图找规律的问题。这些问题要求我们先观察图形，找出它们背后的规律，然后选出符合规律的图形。在处理数据的时候，我们常常会遇见有一些数据因为某些原因丢失。一个很自然的想法就是利用相关的数据去**猜测**丢失的数据是什么。这就是我们这节要考虑的问题。

## 假设
为了进行我们的猜测，我们先假设我们的特征向量$\pmb{x}$服从多元正态分布（Multivariate Normal Distribution, MVN）。对于连续变量，假设其服从MVN是非常自然的。这里，我们先不加证明地引入一个定理用于计算MVN各个分量的边缘分布以及分量之间的条件分布。

> 定理1
> 假设我们特征向量分成两个部分$\pmb{x} = (\pmb{x}\_1, \pmb{x}\_2) \sim \mathcal{N}(\pmb{\mu}, \pmb{\Sigma})$，其中
>
> $$
> \newcommand{\blockmatrix}[1]{\left(\begin{matrix}#1\end{matrix}\right)}
> \newcommand{\inv}[1]{ \pmb{ #1 }^{-1} }
> \pmb{\mu}=\blockmatrix{\pmb{\mu}_1 \\ \pmb{\mu}_2},
> \pmb{\Sigma} = \blockmatrix{\pmb{\Sigma}_{11} & \pmb{\Sigma}_{12} \\
> \pmb{\Sigma}_{21} & \pmb{\Sigma}_{22}},
> \pmb{\Lambda} = \inv{\Sigma} =
> \blockmatrix{\pmb{\Lambda}_{11} & \pmb{\Lambda}_{12} \\
> \pmb{\Lambda}_{21} & \pmb{\Lambda}_{22}}
> $$
> 那么，我们可以得到两个部分分别的边缘分布：
> $$
> \begin{eqnarray*}
> p(\pmb{x}_1) &=& \mathcal{N}(\pmb{x}_1|\pmb{\mu}_1,\pmb{\Sigma}_{11}) \\
> p(\pmb{x}_2) &=& \mathcal{N}(\pmb{x}_1|\pmb{\mu}_2,\pmb{\Sigma}_{22})
> \end{eqnarray*}
> $$
> 以及，在$\pmb{x}\_2$条件下的$\pmb{x}\_1$的分布：
> $$
> p(\pmb{x}_1|\pmb{x}_2) = \mathcal{N}(\pmb{x}_1|\pmb{\mu}_{1|2},\pmb{\Sigma}_{1|2})
> $$
> 其中，
> $$
> \begin{eqnarray*}
> \pmb{\mu}_{1|2} &=& \pmb{\mu}_1 + \pmb{\Sigma}_{12} \inv{\Sigma}_{22}(\pmb{x}_2 - \pmb{\mu}_2)\\
> &=& \pmb{\mu}_1 + \inv{\Lambda}_{11}\pmb{\Lambda}_{22}(\pmb{x}_2 - \pmb{\mu}_2) \\
> &=& \pmb{\Sigma}_{1|2} (\pmb{\Lambda}_{11}\pmb{\mu}_1 - \pmb{\Lambda}_{12}(\pmb{x}_2-\pmb{\mu}_2)) \\
> \pmb{\Sigma}_{1|2} &=& \pmb{\Sigma}_{11} - \pmb{\Sigma}_{12}\inv{\Sigma}_{22}\pmb{\Sigma}_{21} = \inv{\Lambda}_{11}
> \end{eqnarray*}
> $$

值得注意的是，定理中$\pmb{x}\_1$、$\pmb{x}\_2$,以及边缘分布$\pmb{x}\_1|\pmb{x}\_2$均同样服从MVN。

## 离散化
要对一个连续函数$f(t)$进行插值，首先我们先将这个函数切分成一个个小区间便于处理。这里我们将$f(t)$的定义域$\[0,T\]$分成$D$个等长的小区间，并且将这个区间内的函数值用其端点处的函数值代替，即：

$$
x_j=f(s_j), s_j=jh, h=\frac{T}{D}, 1 \le j \le D
$$

在插值的时候，我们会很自然假设被插值的未知函数是光滑的。所以，我们假设$x\_j$是其邻居$x\_{j-1}$和$x\_{j+1}$的平均值，加上一个高斯噪声，即

$$
x_j = \frac{1}{2}(x_{j-1}+x_{j+1}) + \epsilon_j, 2 \le j \le D-2
$$
其中$\epsilon\_j \sim \mathcal{N}(0,(1/\lambda^2)\pmb{I})$。$\lambda$越大，$\episilon\_j$的方差就越小，函数曲线就越光滑。

上述一共$D-3个$
