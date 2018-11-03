---
title: 从零开始的Promise
categories: "Technology"
tags:
    - Promise
    - JavaScript
---

`Promise`是JavaScript中一种包装异步调用的方法。`Promise`初步解决了多重异步调用下的回调地狱的问题。因此成为了主流的进行异步调用的方法。本文将从标准出发，手写一个简单的`Promise`类，深入理解`Promise`。

<!--more-->

本文参考文档：[Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)。

## Promise核心

个人认为`Promise`有两个核心概念，即（1）执行函数接受`resolve`与`reject`函数用于判定调用哪个回调函数。（2）链式调用，`Promise`的`then`与`catch`都返回一个新的`Promise`对象，这是解决回调地狱问题的核心。

在写之前，我们先来看一下使用`Promise`的方式：

```JavaScript
const promise = new Promise(function(resolve, reject) {...})
```

这里我们新构造了一个`Promise`对象，构造函数中传入了一个匿名函数，也可称为`executor`。`executor`接受两个参数，第一个参数是`resolve`，第二个参数是`reject`，分别用于完成或者失败时使用。它们都是函数。调用的时候传入的参数就是传入对应回调函数的参数。

回调函数可以使用`then`与`catch`方法设置：

```JavaScript
const promise = new Promise(function(resolve, reject) {
  setTimeout(() => { resolve('OK') }, 100)
}).then(onFulfilled).catch(onRejected)
```
当`Promise`被`resolve`的时候，`onFulfilled`函数被调用，当`Promise`被`reject`的时候`OnRejected`被调用。

## 实现

首先我们先写一个构造函数，为了方便，这里使用`ES6`的`class`语法。

```JavaScript
class SimplePromise {
  constructor(executor) {
    this._status = 'pending'
    let retVal;
    try {
      retVal = executor(this.resolve, this.reject)
    } catch (e) {
      this.reject(e)
    }
    if (this._status === 'pending') {
      this.resolve(retVal)
    }
  }
}
```