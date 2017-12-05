---
title: 从零开始的JavaScript的OOP世界：对象创建
tags:
  - JavaScript
---
面向对象编程（Object-oriented Programming, OOP）是广为使用一种编程范式。许多编程语言支持这一编程范式，当然JavaScript也不例外。然而JavaScript并没有从语言层面上直接给出例如定义类，继承等专门的语法。所以，人们通过这给来自一些像C++、Java等编程语言背景的开发者来说，JavaScript的OOP技巧看起来非常奇怪，这中间也包括我。本文将以一个只明白类C++方式OOP的JavaScript初学者去学习JavaScript方式的OOP。
<!--more-->

## JavaScript世界的公理——一切都是Object
> JavaScript本来没有OOP，用的人多了，OOP就出来了。

JavaScript世界中并没有像其他编程语言中许多的条条框框。从语言层面上看，JavaScript的规则极为简单。但是简单的规则不能够覆盖方方面面的需求。这使得人们需要“创造性的”从简单的规则中寻找构建复杂事物的方法。这一整套的OOP技巧即是一种。

### 从最原始开始——直接操作对象

在JavaScript的世界中，所有的故事都要从“一切都是Object”开始。我们用到的所有东西都是`Object`（对象），函数也好，变量也好。我们可以非常方便地为一个对象添加属性，属性也是对某个对象的引用。这个原则，使得我们在运行时像拥有上帝的能力一般，能够自由操纵整个世界。我们可以在运行时从零开始构建我们的世界。

  var baby = new Object();
  baby.name = "James";
  baby.gender = "male";
  baby.growUp = function () {
    console.log(this.name + " grows up".);
  };

这里，我们构建了一个`baby`对象，它有它自身的`name`、`gender`。我们也为这个`baby`对象提供了一个`growUp`属性，它的值是一个函数。这个demo虽小，但是已经是OOP意义中对象了。因为，我们已经将相关的属性（attribute）与方法（method）组合在一个个体中了。这位`baby`将是我们OOP世界中第一位住民。

*this指针：*

这个原则赋予了JavaScript高度的灵活性。但是，如果我们在工作中构建对象都要使用上面的方法不免就会有一些繁杂了。于是，有一些聪明的人就用*工厂函数*封装上面的代码。

## 参考资料
　1. Professional JavaScript for Web Developers 3 ed., Nicholas C. Zakas
