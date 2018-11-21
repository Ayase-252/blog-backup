---
title: Step by Step —— 解密原型链
tags:
  - Web Development
  - JavaScript
categories: Programming
date: 2018-11-22 00:00:58
---


原型链是JavaScript中的一个核心特性，会在前端面试中经常问道。原型链原理十分简单，但是在面试题里看起来似乎不是那么简单。通过一些不常有的操作，原型链的面试题很容易就可以将人陷入一个迷宫之中无法脱出。只有掌握原型链的基本原理的勇士才能够找出通往出口的道路。

但是正如我说的，原型链的原理是简单的。本文会通过三个步骤，来看这些面试题是怎么将简单的原型链弄复杂的。

<!--more-->

## 入门：prototype与[[prototype]]

原型链的原理其实很简单：**如果在某个对象中查找一个属性而在当前属性中未命中时，JavaScript引擎会转向该对象`[[prototype]]`指向的对象继续查找这个属性**。在一般浏览器里`[[prototype]]`被实现为`__proto__`属性，因此下面用`__proto__`替换`[[prototype]]`的写法。

因此执行以下代码：

```javascript
const obj = {a: 1, b: 2}
obj.__proto__ = {c: 2}

console.log(obj.c)  // 2
```

`obj.c`为2，因为`c`属性不在`obj`的属性中，因此JS引擎向原型链的上一级，即`obj.__proto__`查找`c`，此时命中了`c`。所有原型链的终极思路都是这个。

接下来，我们将问题变的复杂一点点：

```javascript
function Person (name) {
    this.name = name
}

Person.prototype.sayName = function () {
    console.log(this.name)
}

const jim = new Person('Jim')
jim.sayName() // 'Jim'
```

这是一个很典型地使用原型链实现类的形式。这里`jim`搜寻到了`.sayName`方法。为什么呢？这个问题的关键在于`new Person(Jim)`干了什么？

`new`操作符可以看作以下函数：

```javascript
function _new(F, ...args) {
    const obj = {}
    obj.__proto__ = F.prototype
    const retVal = F.apply(obj, args)
    return retVal || obj
}
```

总而言之，`new`就干了三件事情：
1. 生成一个继承自`F.prototype`的新对象`obj`；
2. 在新对象`obj`上调用构造函数`F`；
3. 如果构造函数有返回值，返回返回值，如果没有返回值，返回构造的对象`obj`。

构造函数一般没有返回值。但是在有返回值的情况下注意区别，可能这里要考。

到这里，原型链的初级部分就差不多了。用这些知识可以做一个小练习：

```javascript
function Person (name) {
    this.name = name
}

Person.prototype.sayName = function () {
    console.log(this.name)
}

function Student (name, school) {
    Person.call(this, name)
    this.school = school
}

Student.prototype = new Person()
Student.prototype.saySchool = function () {
    console.log(this.school)
}

const jack = new Student('Jack', 'MIT')
jack.sayName()
jack.saySchool()
```

上面`jack.sayName()`、`jack.saySchool`会输出什么呢?

## 进阶:constructor

当原型链与`constructor`混合起来，问题就又复杂了一层。复杂的原因在于，从属性名看起来它指向的是对象的构造函数，但是可能因为一堆骚操作之后，`constructor`并不是指向对象的构造函数。

要解决`constructor`是什么的问题，我们还是从最简单的例子开始。

### constructor在哪里？

```javascript
function Person (name) {
    this.name = name
}

const jim = new Person('jim')
console.log(jim.constructor)
```

在浏览器上是可以打印`jim.constructor`属性的，它指向的确实是`Person`函数。但是能够打印属性那么它一定在`jim`上吗？事实上不是的，`jim.constructor`其实是`jim.__proto__.constructor`。在浏览器上执行`jim.constructor === jim.__proto__.constructor`会返回`true`证明了这一结果。

从`new`操作符的执行过程来看，我们并没有向新构造的元素添加`constructor`属性，我们有的只是通过`__proto__`方法继承构造函数的`prototype`，`constructor`就是在这一步被加入对象的原型链中。

在明白了`constructor`在哪里之后，诶？说了那么久，`constructor`到底是什么东西啊?

### `constructor`是什么？

相信大家都用过`instanceof`运算符。这个运算符的功能就是沿着原型链查找`constructor`，如果在原型链上找到了运算符右边所指的构造函数，那么`instanceof`就会返回`true`。

如：
```javascript
console.log(jim instanceof Person) // true
console.log(jim instanceof Object) // true
```

大家可能会疑问为什么`jim instanceof Object`是`true`呢？这个答案在`jim.__proto__.__proto__.constructor === Object`。这就是沿着原型链查找`constructor`属性的意思。这样也就是说`Person.prototype.__proto__.constructor === Object`。emmmm，越来越混乱了吧，这个思路我们先停在这里，不要细想。接下来我们会讲到这一问题。

### 练习题

明白了这一原理，我们来面试中常出现的骚操作：

```javascript
function Person (name) {
    this.name = name
}

Person.prototype = {}

const jim = new Person('jim')
console.log(jim.constructor)
```

现在`jim.constructor`该打印出什么呢？

按照我们刚才的思路，`jim.constructor === jim.__proto__.constructor === Person.prototype.constructor === {}.constructor`。注意到`{}`本身又是一个对象，它也有`__proto__`属性，我们继续推导`{}.constructor === {}.__proto__.constructor === Object.prototype.constructor === Object`。推导结束，答案为`Object`。大家可以自行在浏览器里验证这一答案。

看完上面的推导过程是不是感觉在做数学题一样呢？但是这种骚操作仍然是较简单的操作，更加混乱的还在后面。

## 专家：`Function.prototype`

既然，对象的`__proto__`属性与构造函数的`prototype`属性的关系是如此密切。那么在默认情况下，构造函数的`prototype`又是什么呢？

通过控制台查看`Person.prototype`可以看到，在默认情况，`Person.prototype`是：

```javascript
constructor: ƒ Person(name)
__proto__:
    constructor: ƒ Object()
    hasOwnProperty: ƒ hasOwnProperty()
    isPrototypeOf: ƒ isPrototypeOf()
    propertyIsEnumerable: ƒ propertyIsEnumerable()
    toLocaleString: ƒ toLocaleString()
    toString: ƒ toString()
    valueOf: ƒ valueOf()
    __defineGetter__: ƒ __defineGetter__()
    __defineSetter__: ƒ __defineSetter__()
    __lookupGetter__: ƒ __lookupGetter__()
    __lookupSetter__: ƒ __lookupSetter__()
    get __proto__: ƒ __proto__()
    set __proto__: ƒ __proto__()
```

即一个指向自身的`constructor`属性，以及指向`Object.prototype`的`__proto__`。构造函数的`prototype`属性是**唯一影响通过该构造函数构造的对象的原型链的因素。**

但是，在面试中为了增加混乱度，有一些单位会在`Function.prototype`上动手脚，这又是怎么一回事呢？

### 构造函数与Function.prototype

在JavaScript中，函数本身也是对象，因此函数本身也有`__proto__`属性指向`Function.prototype`对象。可以通过`Person.__proto__ === Function.prototype`验证这一结论。因为这条原型链的关系，我们可以使用诸如`Person.call(this, args)`这些函数专有的方法。但是注意，这条原型链上与构建出来的对象的原型链是没有任何关系的，因为`new`操作符的关系，构造出来的对象的`__proto__`只和函数的`prototype`有关。

### 终极测验

下面是本文的终极测验，类似真实面试中的题，可以花几分钟时间做一下...

```javascript
function o1 (a) {
    this.a = a
}
o1.prototype.b = 2
Function.prototype.d = 4
const ob1 = new o1(1)

console.log(ob1.a)
console.log(ob1.b)
console.log(ob1.constructor)
console.log(ob1.__proto__.constructor)
console.log(ob1.constructor.prototype.constructor)
console.log(ob1.constructor.__proto__)
console.log(o1.d)
console.log(o1.prototype.__proto__)
console.log(o1.prototype.__proto__.constructor)
```

答案密封线

==========

答案揭晓，不知道各位同学答对没有

```javascript
console.log(ob1.a)  // 1
console.log(ob1.b)  // 2
console.log(ob1.constructor) // o1
console.log(ob1.__proto__.constructor) // o1
console.log(ob1.constructor.prototype.constructor) // o1
console.log(ob1.constructor.__proto__) // Function.prototype
console.log(o1.d) //4
console.log(o1.prototype.__proto__) // Object.prototype
console.log(o1.prototype.__proto__.constructor) // Object
```

这种题我还能随手写很多，可见其变化。。。

## 总结

在遇见复杂的原型链问题时，特别是像上面那种代码非常杂乱的，首先先冷静，问题都是从一个对象查找一个属性，用我们的原型链原理：

> 如果在某个对象中查找一个属性而在当前属性中未命中时，JavaScript引擎会转向该对象`[[prototype]]`指向的对象继续查找这个属性。

此时只要明白一件事情，当前对象里有什么属性。如果当前对象里没有所要的属性，就搞清楚当前对象的`__proto__`里有什么属性。根据对象的不同，如普通对象（由字面值构建出来的对象）、构造对象（由`new`操作符构建的对象）、函数对象，一定弄清楚`__proto__`是什么。一直查找到不存在`__proto__`或者`__proto__`为`null`为止。

下面是基于本文例子的一张关系图，用于参考。
![relation among objects](relation-among-objects.png)

## 参考资料
1. [new运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)
