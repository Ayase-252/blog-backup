---
title: 高级数据结构系列：红黑树
categories:
  - Algorithm
tags:
  - Red Black Tree
mathjax: true
---

二叉搜索树（Binary Search Tree, BST）是一种可以进行高效搜索的数据结构。
但是，BST 在最差情况下进行搜索的时间复杂度为$O(n)$，其中 $n$ 为需要搜索的元素数量。
最差情况发生在插入序列是升序或者降序时，如将 1、2、3、4、5 依次插入 BST 时，
会形成一颗如下图的 BST：

![退化的BST](degraded-bst.png)

此时 BST 实际上已经退化成了有序数组。

在实际应用中，这种最差情况是非常常见的。比如，从 API 中可能会获取到预先排序好的数据。
因此，我们需要有一种最差情况下性能更好的数据结构。这就是本文中介绍的红黑树（Red Black Tree）。

## 2-3 搜索树

从对 BST 最差情况分析，如果我们能够“控制”树的高度，在保持 BST 的性质的同时，
用尽可能矮的树来容纳尽可能多的元素。我们就可以将最差情况下搜索时间复杂度控制到对数级别。
那么，是否有这样理想的树呢？

图灵奖得主 John Hopcroft 教授发明一种可以自动平衡的搜索树——_2-3 树_。
这种数据结构能够在插入元素时保持每一个叶子节点到根节点的距离相同，非常接近我们的需要。

> *2-3 树*是一颗树，它要么为空，要么：
>
> - 是一个 _2-节点_，该包含一个键$key$索引数据，与两个子节点，其左子节点的键要比键$key$要更小，其右节点的键要比键$key$更大；
> - 是一个 _3-节点_， 该节点包含两个键$lowerBound$、$upperBound$，与 3 个子节点，$left$、$middle$与$right$。
>   其中左子节点 $left$的键要比$lowerBound$小，
>   中间子节点$middle$的键在 $lowerBound$ 与 $upperBound$，
>   右节点 $right$的键要比$upperBound$大。
