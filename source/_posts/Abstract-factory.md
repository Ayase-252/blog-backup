---
title: 原笔记：Abstract Factory
date: 2016-10-06 18:16:29
categories: "Note"
tags:
  - Design Patterns
  - Abstract Factory
  - Raw Note
---
# 意图
  Abstract Factory给一类class提供了一个统一接口。Client调用接口时会得到实际起作用的那一个class的实例。
而决定哪一个class起作用的是abstract factory。这样client便与class的具体实现解耦。
<!-- more  -->

# 应用领域
  -系统的实现不关心product是如何创建，组成，表示的；（系统只需要知道有这样一个product。）
  -系统可以被一类product中的任意一个所配置；
  -一类product的接口是相同的；（所有的类都是Abstract factory的子类）
  -提供一个含有相同接口的class的库，你只想提供其接口，而不想提供它的具体实现。

# 结构
![](/images/2016/10/Abstract Factory.png)

# 参与者
  - AbstractFactory
  - ConcreteFactory
  - AbstractProduct
  - ConcreteProduct
  - Client

# 作用过程
  `client`调用`AbstractFactory`，而实际上，`AbstractFactory`会使用`ConcreteFactory`去实例化真正起作用的`class`。
  `client`通过`AbstractProduct`去处理`ConcreteFactory`返回的`ConcreteProduct`。通过继承和多态，这个设计是可以实现的。

# 优势与劣势
  - 实现了`client`与`concrete class`的隔离。
  通常意义上，`concrete class`才是实际承担工作的`class`。如何使他们在一起工作？一个本能的想法就是，`client`直接调用`concrete class`。
  但是当软件需求发生变化时，比如需要一个新的实现。我们将会被迫去修改所有`client`中对于上一个`concrete class`的调用。当使用`Abstract Factory`
  模式时，`client`不会关心实际的`concrete class`是什么。这样，当需求变化时，只需要修改`abstract factory`去返回新的`concrete class`，
  而不需要改动`client`。而后者这种修改显然是更为便捷的。

  - 使得**运行时改变**生效的`concrete class`更为容易
  `abstract factory`模式下，改变其返回的`concrete class`是十分简单的。甚至可以在运行时完成转换。

  - 要求`product`具有一致性
  `client`通过`AbstractProduct`去操作实际的`ConcreteProduct`，显然，需要`ConcreteProduct`的接口一致。

  - 在添加新需求时**缺乏灵活性**
  `ConcreteFactory`需要与`AbstractFactory`的接口保持一致，但是如果在开发过程中出现了新的需求，将不可避免的要更改`AbstractFactory`，
  然后,`ConcreteFactory`要保持一致的话也需要更改。在一个大型系统中，这个工作量是非常大的。

# 参考文献
  - [1]	Gamma E, Helm R, Johnson R. Design Patterns: Elements of Reusable Object-Oriented Software [M]. Boston: Addison-Wesley, 1994.
