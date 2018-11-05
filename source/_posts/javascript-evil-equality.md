---
title: 混乱邪恶：JavaScript中的相等操作符（==）
tags:
  - JavaScript
categories: Programming
date: 2018-11-05 20:59:59
---


昨天睡觉前刷到一道面试题，这个问题萦绕在我的脑海里，使得我失眠到凌晨3点。题干很简单：
`[] == ![]`的结果是什么？

所以我们今天来介绍一下问题的主角——相等操作符`==`，看完今天的文章，也许技术上没有什么提升，因为这玩意坑太多在开发中是极力避免使用的。但是这是一个很好的JavaScript式的问题。即如何在混乱中走出一条秩序的道路。

<!--more-->

现在请回答以下表达式的返回值，以及`if`语句会不会执行。

```javascript
0 == false
'0' == false
if ('0') {
  console.log('Yes, believe me, it will be executed')
}
```

答案是：

```javascript
0 == false // true
'0' == false //true
if ('0') { // true, it will be executed
  console.log('Yes, believe me, it will be executed')
}
```

明明`'0' == false`成立，为什么`'0'`在`if`中被认为是`true`呢？（现在不要深究这个问题，否则大脑会stack overflow的，相信我）这个结果混乱的相等操作符`==`是无数BUG的根源。有人总结过各种类型的值使用`==`比较后的结果，有兴趣的话可以看看[JS Comparision Table](https://dorey.github.io/JavaScript-Equality-Table/)。Reddit上一位网友对此的评论，我认为很精髓：

![reddit comment](reddit-comment.png)

## 万恶之源——隐式类型转换

`==`之所以变得那么不讲道理，是因为在使用`==`时，对两边表达式进行**隐式类型转换**。这里，我推荐一篇文章“[从\[\]==!\[\]为true来剖析JavaScript各种蛋疼的类型转换](https://segmentfault.com/a/1190000008432611)”。这篇文章把整个`==`涉及的表达式值的转换过程讲的非常清楚。字比较多，大家慢慢看哈wwww。


好，看完回来，相信大家对隐式转换的过程有一定的了解了。但是这篇文章对`toPrimitive`介绍的不够清晰，这里我再推荐一篇文章[Object to primitive conversion](https://javascript.info/object-toprimitive)。

让我们总结一下从上面学到的一些东西：

> 在相等操作符中：
> 1. `undefine == null`，`undefine`和`null`与其他任何类型比较均为`false`；
> 2. 若两边类型相同，按照各自类型的抽象相等规则比较：
>     2.1 `number`比较大小；
>     2.2 `string`只有在长度和对应位置字符都相等才相等；
>     2.3 `boolean`，不用说了吧...
>     2.4 `object`，比较类在内存中的地址（即两个类指向的是同一个类）；
> 3. 基本类型(`boolean`, `string`)转化为`number`后相互比较；
> 4. `Object`使用`ToPrmitive`算法转化为基本类型之后按2比较。

> ToPrimitive算法：
> 在相等操作符中，内置的`Object`（除了`Date`）均以`default`为hint转换为基本类型：
> 1. 如果有`.valueOf()`，尝试使用`.valueOf()`返回**基本类型**；
> 2. 如果没有`.valueOf()`，或者上一步尝试失败，但是有`.toString()`，尝试使用`.toString()`返回**基本类型**；
> 3. 如果都没有，或者上一步尝试失败，报错`TypeError`；
>
> `Date`以`string`为hint转化为基本类型：
> 1. 如果有`.toString()`，尝试使用`.toString()`返回**基本类型**；
> 2. 如果没有`.toString()`，或者上一步尝试失败，但是有`.valueOf()`，尝试使用`.valueOf()`返回基本类型；
> 3. 如果都没有，或者上一步尝试失败，报错`TypeError`

如果对`Date`的转换规则有疑虑的话，可以运行下面的代码验证：

```javascript
const date = new Date()
console.log(date == date.valueOf())  // false
console.log(date == date.toString()) // true
```

## 基本`Object`的类型转换

下面看一些常用的基本类型的`.valueOf()`以及`toString()`是什么，列表总结如下：

| 类型     | 具体值                              | `.valueOf()`   | `.toString()`                                        |
| -------- | ----------------------------------- | -------------- | ---------------------------------------------------- |
| `Object` | `{a: 1, b: 2}`                      | `{a: 1, b: 2}` | `"[object Object]"`                                  |
| `Array`  | [1, 2, 3]                           | `[1, 2, 3]`    | `"1, 2, 3"`                                          |
| `Date`   | `new Date('1970/1/1 00:00:00 GMT')` | `0`            | `"Thu Jan 01 1970 08:00:00 GMT+0800 (中国标准时间)"` |



可以看到`Object`与`Array`的`ToPrimitive`的首选方法`.valueOf()`返回的均不是基本类型之一，因此会使用`toString()`。。。Oh my god。。

## 那么我们开篇问题的答案是

是`true`。先写出整个表达式`[] == ![]`，右边`![]`是`false`（对于非操作符，是这么操作的`!toBoolean(GetValue(expr))`），而对于所有`Object`，`toBoolean`的结果都是`true`，所以右边整个式子是`false`，然后这个布尔值转化为数字是`0`。接下来我们来看左边，首先`[]`用`toPrimitive`转化为基本类型是`""`，然后这个字符串转化为数字，结果也是`0`。最后`0 == 0`，当然是`true`啦。

## 总结

我一直认为相等操作符是JavaScript的设计失误之一。要弄清楚一个相等操作符的结果，与做一道数学证明题相似，从题目到中间结果到中间结果....到结果。所以，`==`仅适用于题目，而对于开发项目而言可能是万恶之源。好在JavaScript提供了正常一些的严格相等操作符`===`。所以总结就是：

> Always use 3 equals unless you have a good reason to use 2.
>
> 节选自[JS Comparision Table](https://dorey.github.io/JavaScript-Equality-Table/)

