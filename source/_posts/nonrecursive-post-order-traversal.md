---
title: 非递归的二叉树后序遍历
date: 2016-10-28 11:21:03
categories: "Techno"
tags:
  - Algorithm
  - Binary Tree
---


后序遍历是遍历二叉树的一种方式。其遍历方式是先遍历一个结点的所有左子结点，再遍历其所有右
子结点，最后访问自身。后序遍历有非常简单的递归解法，但是如果要把它转化为非递归的解法的话
会出现一定的困难。

<!-- more -->

## 后序遍历的递归解法
后序遍历可以归结于以下步骤：
1. 后序遍历一个结点的左子结点；
2. 后序遍历一个结点的右子结点；
3. 访问自己

因此可以使用以下递归算法来解决这一问题（使用伪代码描述）：

    Post-Order-Traversal(node):
      if node.left != NIL
        Post-Order-Traversal(node.left)
      if node.right != NIL
        Post-Order-Traversal(node.right)
      Manipulate(node)

## 非递归的后序遍历算法
众所周知，程序中函数的调用一般是通过栈实现的。因此，对于一个递归函数，我们总能够通过模仿
系统管理栈的做法把它转化为非递归的实现。
但是，在后序遍历里，只使用一个栈记录结点是不足的。当先前一个结点，如一个结点的父结点，当
它出栈的时候，算法不能够记录它的状态，如它的子结点是否已经处理过？显然处理过和没处理过两
种状态，之后要执行的步骤是不一样的。基于这种想法，在非递归的实现中，我们可以增加另外一个
栈，专门记录状态。在一些语言中允许使用`tuple`或`pair`这样的有序数组，可以使用这些结构作
为栈的元素（因为结点和状态是一一对应的），这样就可以不用另外开一个栈。

非递归的后序遍历算法可以写成如下伪代码：

    //Sn: New stack of node
    //Ss: New stack of status
    //Node: root of tree
    Post-Order-Traversal(Sn, Ss, node):
      Sn.push(node)
      Ss.push('uncheck')
      while Sn is not empty:
        presentNode = Sn.pop()
        presentStatus = Sn.pop()
        if presentStatus is 'uncheck':
          Sn.push(presentNode)
          Ss.push('checked')
          if presentNode.right != NIL:
            Sn.push(presentNode.right)
            Ss.push('uncheck')
          if presentNode.left != NIL:
            Sn.push(presentNode.left)
            Ss.push('uncheck')
        else  // Case for checked node
          Manipulate(presentNode)
