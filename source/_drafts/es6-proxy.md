---
title: ES6 Proxy操作速查
categories:
    - Programming
    - Front-end development
    - JavaScript
tags:
    - JavaScript
    - Proxy
    - ES6
---

这篇文章不会深入Proxy。我只是在看[文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)的时候觉得它在Proxy的Handler怎么用上组织得不太好，写这一篇文章总结一下Handler基本操作的用法用于日后速查。

<!--more-->

## 基本语法

```javascript
let p = new Proxy(target, handler);
```

参数：
- `target`： 用`Proxy`包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）。
- `handler`：一个对象，其属性是当执行一个操作时定义代理的行为的函数。

## Handler支持的操作

`handler`支持13种可代理的操作，如果`handler`中没有定义某种操作，那么这种操作会被转发到`target`上。

注意，由于Proxy代理了JS在`target`上的原生操作。Proxy代理的效果不能够违背JavaScript的处理原则，如在代理`delete`时将不可配置的属性删除。如果违背原则的话，Proxy会抛出`TypeError`。这些原则在下文中被称为**不变量(Invariant)**。

### .apply()

`handler.apply()`拦截函数调用。

#### 语法

```javascript
var p = new Proxy(target, {
  apply: function(target, thisArg, argumentsList) {
  }
});
```

参数：

- `target`: 目标对象
- `thisArg`： 被调用时的上下文对象
- `argumentsList`： 目标对象被调用时的参数数组

返回值：

**任何值。**

#### 触发代理的操作

- 对代理对象的函数调用：`proxy(...args)`
- 使用代理对象的`apply`, `call`方法
- `Reflect.apply()`

#### 不变量

`target`必须可调用，也就是说`target`必须是函数对象。

### .construct

`handler.constructor()`用于拦截`new`操作符。为了使`new`操作符成立，`new target`必须是有效的。

#### 语法

```javascript
var p = new Proxy(target, {
  construct: function(target, argumentsList, newTarget) {
  }
});
```

参数：

- `target`：目标对象
- `argumentsList`：调用`new`时的参数列表
- `newTarget`：最初被调用的构造函数，就上面的例子而言是`p`。

返回值：

必须返回一个**对象**。

#### 触发代理的操作

- 使用代理对象创建新对象：`new proxy(...args)`
- Reflect.construct()

#### 不变量

`handler.construct`**必须返回一个对象**。

### .defineProperty()

`handler.defineProperty()`用于拦截对对象的`Object.defineProperty()`操作。

#### 语法

```javascript
var p = new Proxy(target, {
  defineProperty: function(target, property, descriptor) {
  }
});
```

参数：

- `target`：目标对象
- `property`： 要定义的属性名
- `descriptor`：待定义或者修改的描述符

返回值：

必须返回一个`Boolean`，表示对该属性的操作成功与否。

#### 触发代理的操作

- `Object.defineProperty(proxy, property, descriptor)`
- `Reflect.defineProperty()`

#### 不变量

- 如果**目标对象**不可扩展，则不能添加属性；
- 如果它不是目标对象的一个不可配置属性，则不能添加或者修改这个属性为不可配置；
- 如果目标对象存在一个对应的可配置属性，则该属性可以不是不可配置的（可以是可配置的）（原文：A property may not be non-configurable, if a corresponding configurable property of the target object exists.）；
- 如果目标对象存在一个对应的属性，`Object.defineProperty(target, prop, descriptor)`不会抛出异常；
- 在**严格模式**下，`handler.defineProperty`不能返回`false`

## 参考资料
1. [Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
2. [handler.apply()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/apply)
3. [handler.construct()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy/handler/construct)