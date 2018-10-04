---
title: 解决图的动态连接性问题：Union-Find算法
tags:
  - Algorithm
  - Graph
  - Union-Find
  - Dynamic Connectivity
categories: Note
mathjax: true
date: 2018-07-31 19:59:36
---


图是一种非常有用的数据结构。很多问题可以看作是图来处理，如迷宫、网络等等。有的时候，我们需要寻找图的连通分量。如[LeetCode的721题（账号整合）](https://leetcode.com/problems/accounts-merge/description/)，解法的中心思路就是把Email看作是图中的节点，通过动态的增加边（把在同一个账户中的Email连接起来），最后读出图中的所有连通分量。这里，我们介绍一种专门处理图的动态连接性问题的算法——Union-Find。
<!--more-->

_关于Union-Find算法，在Robert Sedgewick等所著的《Algorithms 4 ed.》的1.5节有图文并茂的描述，有兴趣的话可以参阅。_

## 中心思想
对于图的动态连接性问题，我们最关心的问题是：图中节点间之间是否存在路径（节点是否处于同一个连通分量中）。对于图的动态变化，我们允许动态地添加边，但是不允许删除边。因此，定义API如下：

| API签名                        | 描述                        |
| ------------------------------ | --------------------------- |
| `bool connected(int p, int q)` | 节点`p`与节点`q`是否连通    |
| `void union(int p, int q)`     | 将节点`p`与节点`q`连接起来  |
| `int find(int p)`              | 寻找`p`处于哪一个连通分量中 |

很显然，对于`connected`方法，只要检查`p`与`q`是否处于一个连通分量中就行，实现很直观：

```Python
class UnionFind:
    def find(self, p):
        # To be implemented
        pass

    def connected(self, p, q):
        return find(p) == find(q)
}
```

Union-Find算法的核心在于`union`方法与`find`方法的实现上，根据侧重操作的不同可以演变为三种不同的版本Quick-Find, Quick-Union与Weighted-Union。

## Quick-Find
Quick-Find算法是理解起来最简单直接的Union-Find算法。核心思想就是通过一个数组持续地追踪节点处在的连通分量。实现如下：

```Python
class QuickFind:
    def __init__(self, numNode):
        # _id数组表示的是各个节点的连通分量号
        self._id = list(range(numNode))

    def union(self, p, q):
        pId = self.find(p)
        qId = self.find(q)
        # 将所有处于q所在连通分量的节点放在p所在的连通分量中
        for i in range(len(self._id)):
            if self._id[i] == qId：
                self._id[i] = pId

    def find(self, p):
        return self._id[p]

```

通过分析这个简单的实现，不难看出为什么它叫Quick-Find版本。`find`操作仅仅需要返回`_id`数组中所储存的数，时间复杂度为常数，即$O(1)$。但是对于`union`操作，假设节点数为$N$，它每次都需要遍历所有的节点，因此时间复杂度为$O(N)$。对于动态变化非常多的图，Quick-Find版本的性能就会比较低了。我们有什么办法能够提高`union`操作的性能呢？

## Quick-Union
联想到对于元素插入频繁的情况，链表的表现要比数组来的好。因此我们可以模拟一个“链表”，准确来说是一颗树，把属于同一连通分量的元素放在一棵树中，用这棵树的根节点作为连通分量的编号。在这种想法下，我们可以实现Quick-Union：

```Python
class QuickUnion:
    def __init__(self, numNode):
        # parent数组表示各个节点的父节点
        self.parent = list(range(numNode))
    
    def union(self, p, q):
        # 找到p所在连通分量（树）的根节点
        pRoot = self.find(p)
        qRoot = self.find(q)

        # 将p树的根节点接在q树的根节点下
        self.parent[pRoot] = qRoot

    def find(self, p):
        # 寻找p树的根节点
        last = p
        while self.parent[last] != last:
            last = self.parent[last]
        return last
```

从上面的实现可以看出，`union`除去寻找根节点的所需的耗时，只需要一个将一棵树的根节点放在另外一棵树的根节点下的操作，是常数时间，因此`union`操作的时间复杂度取决于`find`操作，但是`find`操作需要从指定节点遍历到根节点，在最坏情况下需要$O(N)$的时间。

    对于有4个节点的图的最坏情况

             A
            /
           B
          /
         C
        /
       D
    如果find(D)的话需要遍历所有的节点。

Quick-Union版本的`find`操作效率变得比较低。但是我们可以看到，`find`的最坏情况就是需要从树的叶子节点遍历到根节点，所需要的步数就是树的高度。如果我们有办法去降低树的高度，`find`操作效率就会提升。所幸，我们可以用几条代码就可以提高Quick-Union的性能。

## Weighted-Union
为了使树的高度降低，在整合两颗树的时候，把较小的树放在较大的树的下面是比较好的。因此，为了实时地掌握树的规模（树的节点数），我们增加一个数组去追踪这一情况。

```Python
class WeightedUnion:
    def __init__(self, numNode):
        # parent数组表示各个节点的父节点
        self.parent = list(range(numNode))
        # size数组表示以某个节点形成的树包含的节点数
        self.size = [1] * numNode
    
    def union(self, p, q):
        # 找到p所在连通分量（树）的根节点
        pRoot = self.find(p)
        qRoot = self.find(q)

        # 如果p树的节点比q树多，将q树接在p树的下面
        if self.size[pRoot] >= self.size[qRoot]:
            self.parent[qRoot] = pRoot
            self.size[pRoot] += self.size[qRoot]
        else:
            self.parent[pRoot] = qRoot
            self.size[qRoot] += self.size[pRoot]

    def find(self, p):
        # 寻找p树的根节点
        last = p
        while self.parent[last] != last:
            last = self.parent[last]
        return last
```

可以看到Weighted-Union版本的代码只比Quick-Union版本的代码多6行，但是由于小树接在大树规则的限制下，形成的树的高度要降低很多。可以证明其`union`操作以及`find`操作的时间复杂度是$O(\log N)$。对数阶的复杂度说明这已经是一个比较实用算法了。

然而我们还可以进一步缩减树的高度，理想情况下，我们希望树的高度为1，这样的话，`find`操作就可以以近乎常数时间得到树的根节点。理想情况在我们目前实现的版本中是不可能的，但是如果我们能够在运行的过程中动态地压缩非根节点到根节点的路径的话，是可以进一步降低树的高度的。

## 带路径压缩的Weighted-Union

注意到`find`的遍历操作，我们遍历了从指定节点到根节点路径上的所有节点，我们最后也得到了根节点。为了下一次遍历的时候不用再重复一遍这条路径。我们直接将这些节点接到根节点的下面。这仅仅需要添加一个循环：

```Python
class WeightedUnionWithPathCompression:
    def __init__(self, numNode):
        # 与WeightedUnion相同

    def union(self, p, q):
        # 与WeightedUnion相同

    def find(self, p):
        # 寻找p树的根节点
        last = p
        while self.parent[last] != last:
            last = self.parent[last]
        root = last
        
        # 再遍历一遍压缩指定节点到根节点路径上所有节点到根节点的路径
        last = p
        while last != root:
            tempLast = self.parent[last]
            self[last] = root
            last = tempLast
        
        return root
```

可以证明带路径压缩版本的`union`与`find`操作的平摊时间复杂度近乎为$O(1)$。

## 各种版本的Union-Find的时间复杂度汇总

版本|`union`|`find`|
--|--|--|
Quick-Find|$O(N)$|$O(1)$
Quick-Union|最坏$O(N)$|最坏$O(N)$
Weighted-Union|$O(\log N)$| $O(\log N)$
Weighted-Union with Data Compression | 平摊接近$O(1)$| 平摊接近 $O(1)$

## 总结

图的动态连通性问题是非常多实际问题的抽象，这里介绍的Union-Find算法将会是解决这一类问题的利器。