---
title: JavaScript的生成器
categories:
  - Programming
tags:
  - 生成器
  - JavaScript
---

ES6 为 JavaScript 引入了[生成器](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators)特性。
本篇文章将会从生成器的概念出发来讨论一下生成器与其应用。

通过这篇文章，将会了解到：

- 生成器是一种实现[惰性求值](https://en.wikipedia.org/wiki/Lazy_evaluation)
  的一种语言结构；
- 生成器允许我们的函数按需调用，节省在处理大数组情形下的内存需求与提高响应速度；
- 生成器可以用来实现一些很有趣的功能。

## 生成器是什么

生成器是一个特殊的函数，在用途上它与返回一个数组的函数非常像，
可以将生成器看作是产生数组的函数的一种替代方法。

```javascript
function* generator() {
  for (let i = 1; i < 6; i++) {
    yield i
  }
}

function ordinary() {
  return [1, 2, 3, 4, 5]
}

for (let elem of generator()) {
  console.log(elem) // 1,2,3,4,5
}

for (let elem of ordinary()) {
  console.log(elem) // 1,2,3,4,5
}
```

在上面这段代码中，用`function*`声明的`generator`是一个生成器，
`ordinary`是一个普通的返回数组的函数（下面略称数组函数）。
可以看到在`for of`迭代中，两种方式都产生了 1、2、3、4、5。
这说明从调用者的角度来说生成器函数与数组函数是相同的。

生成器与数组函数不同的地方在于，生成器一次只返回数组中的一个元素。
而普通函数需要将数组的元素全部计算出来。这提供了两个好处，
第一是对于空间需求非常大的数组，使用生成器只需要产生数组其中一个元素的内存空间。
第二是对于需要计算时间非常长的数组，使用生成器时，
调用者仅需要计算一个元素的时间就能够恢复控制权。
允许调用者对数组产生这一过程进行更加细粒度的管理。

## JavaScript 中的生成器

## 应用

## 总结

## 参考文献
