---
title: 失败笔试总结-1
tags:
  - Algorithm
categories: Note
mathjax: true
date: 2018-09-21 00:21:19
---


毫无意外地，头条的第四轮笔试看起来又GG了，这次没做出来的问题有3个，分别是（1）字符串的前缀表示、（2）三色小球组合、（3）幸运数字问题。
<!--more-->

## 字符串的前缀表示
题目给定一堆字符串，求针对每一个字符串求前缀表示，使得这个前缀表示能够唯一的代表字符串。在考试中，我的想法利用所有字符串构建一颗字典树。然后遍历字典树，从根节点到叶子节点的路径上最后一次分叉的地方就是所求的前缀表示。可惜的是，题目要求输入与输出顺序要相同。在构建完字典树之后，可以输出所有的前缀表示，但是顺序却可能与输入不一致，导致通过率为0。

针对输入与输出不一致的，一个改进的想法就按照输入顺序在构建好的字典树上去寻找前缀表示。

```Python
class TrieTreeNode:
    # 字典树节点
    def __init__(self, c):
        self.c = c
        # 指向子节点的哈希表
        self.desc_map = {}

    def get_or_append_node(self, c):
        # 得到字符为c的子节点，如果没有就创建一个
        if c not in self.desc_map:
            self.desc_map[c] = TrieTreeNode(c)

        return self.desc_map[c]
    

def prefix(root, word):
    curr_node = root
    curr_prefix = ''

    for i in range(len(word)):
        curr_node = curr_node.desc_map[word[i]]
        if len(curr_node.desc_map) >= 2:
            # 有多个子节点的节点，分叉处，题目保证没有字符串是其他的前缀字符串，因此
            # 可以不加检查的越过分叉处，直接将向目标字符串分叉的那个节点作为前缀字符串
            curr_prefix = word[:i+2]

    return curr_prefix

n = int(input())
dummy_node = TrieTreeNode('')
words = []
for _ in range(n):
    s = input()
    words += [s]
    curr_node = dummy_node
    for c in s:
        curr_node = curr_node.get_or_append_node(c)

for word in words:
    print(prefix(dummy_node, word))
```

## 三色小球组合问题
问题给出三个数，分别是红球、绿球、蓝球的数量。将这些球排成一列，求满足相同颜色的球不相邻的排列的数量。

好吧，想了很久没想出来。在查找的时候，突然看见有一个哥们说用四维DP。一想数据量确实不大，可以用DP的。

假设$DP(s, i, j, k)$表示以$s$为结尾的，分别包含$i,j,k$个红球、绿球与蓝球。$s={0, 1,2}$。递归式如下：

$$
\\begin{eqnarray}
DP(0, i, j, k) = DP(1, i-1, j, k) + DP(2, i-1, j, k) \\\\
DP(1, i, j, k) = DP(0, i, j-1, k) + DP(1, i, j-1, k) \\\\
DP(2, i, j, k) = DP(0, i, j, k-1) + DP(1, i, j, k-1)
\\end{eqnarray}
$$

边界情况为：
$$
\\begin{eqnarray}
DP(0, 1, 0, 0) = 1 \\\\
DP(0, 0, \*, \*) = 0 \\\\
DP(1, 0, 1, 0) = 1 \\\\
DP(1, \*, 0, \*) = 0 \\\\
DP(2, 0, 0, 1) = 1 \\\\
DP(2, \*, \*, 0) = 0
\\end{eqnarray}
$$
其中$\*$表示任取。

最终结果就是$\\sum\_{i=0}^2 DP(i, n\_r, n\_g, n\_b)$。

使用带记忆的递归DP实现如下，应该是可以再优化的。明天起床后试试。

```Python
def dp_with_memorize(dp_table, s, i, j, k):
    if dp_table[s][i][j][k] != -1:
        return dp_table[s][i][j][k]
    else:
        if s == 0:
            if i == 1 and j == 0 and k == 0:
                return 1
            if i == 0:
                return 0
            else:
                dp_table[s][i][j][k] = dp_with_memorize(dp_table, 1, i-1, j, k) + \
                     dp_with_memorize(dp_table, 2, i-1, j, k)
                return dp_table[s][i][j][k]
        elif s == 1:
            if j == 1 and i == 0 and k == 0:
                return 1
            if j == 0:
                return 0
            else:
                dp_table[s][i][j][k] = dp_with_memorize(dp_table, 0, i, j-1, k) + \
                     dp_with_memorize(dp_table, 2, i, j-1, k)
                return dp_table[s][i][j][k]
        elif s == 2:
            if k == 1 and i == 0 and j == 0:
                return 1
            if k == 0:
                return 0
            else:
                dp_table[s][i][j][k] = dp_with_memorize(dp_table, 0, i, j, k-1) + \
                     dp_with_memorize(dp_table, 1, i, j, k-1)
                return dp_table[s][i][j][k]

def tri_color_balls(n_r, n_g, n_b):
    dp_table = [[[[-1 for _ in range(n_b + 1)] for _ in range(n_g + 1)] for _ in range(n_r + 1)] for _ in range(3)]
    return sum([dp_with_memorize(dp_table, i, n_r, n_g, n_b) for i in range(3)])
```

## 幸运数字问题：
定义数字为$n = \\{d\_n, d\_{n-1}, \ldots, d\_1\\}$。定义幸运数字为对于$i，d\_{n-i+1} \neq d\_i$。给定一个范围，求该范围内的幸运数字个数。