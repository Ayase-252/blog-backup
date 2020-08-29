---
title: React Fiber架构
categories:
  - Tech Inside
tags:
  - React
---

在我们使用 React 的时候，我们通常不需要关心 React 底层是如何响应组件的变化的。
例如当一个组件的 `states` 发生改变，
该组件会通过 Diffing 算法进行高效的更新操作。
[React 的文档](https://zh-hans.reactjs.org/docs/reconciliation.html#the-diffing-algorithm)对 Diffing 算法有清楚的描述。
但是，React 如何做到用户体验友好的组件更新，而没有一丝卡顿。
这一切的秘密就在 React 的 Fiber 架构。

<!--more-->

## Reconciliation

在讨论 Fiber 之前，我们需要一个新的概念—— _Reconciliation_（中文文档中翻译为 协调）。
所谓 Reconciliation 是 diff 两颗不同的树找出需要改变的部分的一个算法 [^rfa]。
在 React 中，某个组件的改变不会引起整棵组件树重新渲染。React 会对比树的根节点，当：

1. 如果根节点在改变前后是不同类型的元素，则卸载当前节点的子树，用新的子树替换；
2. 如果根节点在改变前后仍然是相同的类型，React 会保留 DOM 节点，只对变化的部分进行更新，
3. 当改变后，子树中节点 `key` 能够在改变前的树中找到，那么该 DOM 节点会被保留，但是位置可能会发生变化。

通过这个算法，React 能够使用较小的计算开销来改变 DOM 以匹配新的组件状态。

Reconciliation 算法的实现被称为 reconciler（名词形式），
所谓 React Fiber 就是 [React 16](https://zh-hans.reactjs.org/blog/2017/09/26/react-v16.0.html)
新实现的一个 reconciler。

## 从一个计数器说起

Fiber 的概念比较晦涩，为了更好的说明 React Fiber 架构，我们构建一个麻雀虽小，五脏俱全的小应用——计数器。

```jsx
import React from "react";

class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState((state) => {
      return { count: state.count + 1 };
    });
  }

  render() {
    return (
      <>
        <button key="1" onClick={this.handleClick}>
          Add
        </button>
        <span key="2">{this.state.count}</span>
      </>
    );
  }
}
```

渲染 `Counter` 组件会出现一个 Add 按钮，以及一个计数器。当点击 Add 按钮时，
计数器会自增 1。

在这里我们使用了 JSX 语法，但是本质上，JSX 会被编译成一系列由 `React.createElement` 组成的函数语句。
如上面的 `render` 方法经过编译之后会形成

```javascript

  render() {
    return _react.default.createElement(_react.default.Fragment, null, _react.default.createElement("button", {
      key: "1",
      onClick: this.handleClick
    }, "Add"), _react.default.createElement("span", {
      key: "2"
    }, this.state.count));
  }

```

## 任务

当点击 Add 按钮时，React 会启动 Reconciliation。在 Reconciliation 的过程中，

1. `Counter` 组件状态中的 `counter`字段会被更新；
2. `Counter` 子组件的 `props` 会被比较；
3. `span` 元素的 `props` 会被更新（`children` 是一类特殊的 `props`）

除此之外，一些生命周期方法，钩子方法也会被调用。例如被更新元素的 `shouldComponentUpdate`， `componentDidUpdate`。
被卸载元素的 `componentWillUnmount`。我们把这些所有需要执行的活动抽象为一个新的概念 _work_（可以认为是*任务*）。

当调用 `ReactDOM.render()` 方法挂载 React 应用时。从概念上来说，
React 会生成一颗元素树。同时，React 会为元素树中的每一个节点生成一个对应的 `FiberNode`。
这些 `FiberNode` 会自然地组成一颗树，我们称为 Fiber 树。

## 元素

从例子可以看到，`Counter`的 `render` 元素返回了一个由 `React.createElement` API 创建的 _React 元素_。
如果我们从根组件开始渲染，我们最终会得到一颗 _React 元素树_。（这也是 React 组件要求有唯一的父节点的原因）。
我们来看一下 React 元素的数据结构。

```javascript
{
  $$typeof: Symbol(react.element)
  key: null
  props: {}
  ref: null
  type: class Counter
  _owner: FiberNode {tag: 0, key: null, stateNode: null, elementType: ƒ, type: ƒ, …}
  _store: {validated: false}
  _self: null
  _source: null
}
```

可以看到，React 元素就是一个对象，包含 `key`, `props`, `ref`, `type` 等字段。
其中 `type` 字段反向的引用 `Counter` 组件的构造函数。如果是函数组件的话，`type`
会应用组件的函数。当然还有一些其他的字段，但是与 Fiber 的主题无关，就不在这里展开了。

注意到，React 元素的 `_owner` 字段引用了一个 `FiberNode` 类型的实例。这个 `FiberNode`
就是 Fiber 架构的核心数据类型。

## Fiber Node

在 Reconciliation 的过程中，每次渲染组件的结果——元素树，都会被整合到一颗由 Fiber Node 组成的树中，
这里我们称为 Fiber 树。Fiber Node 是一个**维护组件状态与对应的 DOM 元素**的数据结构。与元素树不同，
Fiber Node 是**可变的**，在更新时，React 会**重用** Fiber Node 对象，
而不是拷贝一份新的实例出来。它的生命周期贯穿于整个应用的生命周期。

### Fiber 树的根元素

一个 React 应用可以有一个或者多个容器元素（取决于 `ReactDOM.render` 的调用）。
每一个容器元素会生成一颗 Fiber 树。这颗 Fiber 树的根元素就被存储在容器元素的 DOM 节点上。

React 首先会为每一个容器元素生成一个 `FiberRootNode` 对象。这个对象可以通过
容器 DOM 节点的`._reactRootContainer._internalRoot`属性访问到，
假如我们挂载在一个`id`为 `root`的 `div` 节点上，我们可以通过下列语句访问到 Fiber 树的根节点。

```javascript
const fiberRootNode = document.getElementById("root")._reactRootContainer
  ._internalRoot;
```

Fiber 树的根节点就被存储在 `FiberRootNode` 对象的 `current` 属性中。

```javascript
const hostRootFiberNode = fiberRoot.current;
```

## Fiber 的运行机制

## 总结

[^rfa]: Andrew Clark et al. React Fiber Architecture. https://github.com/acdlite/react-fiber-architecture#react-fiber-architecture
[^in-depth]: Max Koretskyi. Inside Fiber: in-depth overview of the new reconciliation algorithm in React. https://indepth.dev/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react/
