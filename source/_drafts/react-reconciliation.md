---
title: React DOM的Reconciliation
categories:
  - Programming
  - Front-end
  - React
tags:
  - React
  - 性能优化
mathjax: true
---

React 允许我们声明式地创建组件。即我们只需要将注意力集中在返回正确的元素树上。
当组件的`props`与`state`（通过`setState`方法）发生变化的时候。
React 会自动尝试以该组件作为根组件`render`出一颗新的元素树。
React 会对新旧元素树进行对比，以期找出更新元素树最佳的方法，
React 给这个过程起了个非常高级的名字——_Reconciliation_。

<!--more-->

在[剑桥英语词典](https://dictionary.cambridge.org/dictionary/english/reconciliation)中，
Reconciliation 被定义为：

> [U] the process of making two opposite beliefs, ideas, or situations agree.

翻译成中文的话，有人翻译成[协调](https://react.docschina.org/docs/reconciliation.html)，
也有人翻译成[一致性比较](https://react.css88.com/docs/reconciliation.html)。
我觉得它们都差那么点意思，因此本文中直接使用英文原文**Reconciliation**表示这个概念。

## 面临的问题——用最少的操作将一棵树变成另外一棵树

在小时候，我们都做过一些智力题，比如说一个用火柴棍摆成的图案，要怎么移动才能变成另外一种图案。
在 React 中，当元素树由于组件的`props`与`state`更新变成了另外一颗元素树。
React 需要搞清楚如何操作能高效地把旧元素树更新成新的元素树。为了表述方便，
接下来我们把这种算法称为 diff 算法，借鉴 git 中比较新旧代码文件不同的`git diff`操作。

元素树与 DOM 结构一样都具有树的结构，
目前 SOTA(State of the art)的
[diff 算法](http://grfia.dlsi.ua.es/ml/algorithms/references/editsurvey_bille.pdf)
的时间复杂度为$O(n^3)$。这个时间复杂度对于目标 60FPS 的 UI 更新是不可接受的。

## $O(n)$的启发式 diff 算法

为了降低 diff 算法的时间复杂度，React 做出了两个假设：

1. 两个*不同类型的元素*会产生不同树；
2. 开发者可以通过`key`属性暗示 React 哪些子元素会在不同的`render`过程中保持稳定；

基于这两个假设，React 实现了时间复杂度为$O(n)$的 diff 算法。

diff 算法总是起始于新旧两颗元素树的根节点，根据根节点的类型不同，diff 算法会产生不同的行动。

### 当元素的类型不同时

元素有类型，它可以是`<a>`、`<img>`之类的 DOM 元素，或者是`<Button>`之类的组件元素。
当被比较的新旧两个元素的类型不同时，根据假设 1，
React 会**完全丢弃旧树、从零开始构建新树**。

比如有以下情形：

```html
<!-- Old Tree -->
<div>
  <p>1</p>
  <p>2</p>
  <p>3</p>
</div>

<!-- New Tree -->
<article>
  <p>1</p>
  <p>2</p>
  <p>3</p>
</article>
```

React 在比较`<div>`与`<article>`元素的时候，发现元素的类型不同，
尽管似乎内部的`<p>`元素完全一致，但是 React 不会尝试重用它们。
而是重新从零开始构造`<article>`元素及其子元素。

### 当元素是同类型的 DOM 元素时

当新旧元素是相同类型的 DOM 元素时，React 会重用这个 DOM 元素。
它会检查新旧元素中属性值（如`class`、`style`），然后将 DOM 元素的属性更新成新的属性。
在更新完这个 DOM 节点之后，React 会在递归地在其子节点上执行 diff 算法。

### 当元素是同类型的组件元素时

当组件更新时，组件实例不会改变。

## 参考文献

1. [Reconciliation](https://reactjs.org/docs/reconciliation.html)
