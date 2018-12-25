---
title: JavaScript中的Proxy与Reflect
categories:
    - Programming
    - Front-end
tags:
    - JavaScript
    - 元编程
    - Proxy
    - Reflect
---

在尤大发布的[Vue 3.0](https://medium.com/vue-mastery/evan-you-previews-vue-js-3-0-ab063dec3547)计划中，Vue最核心的响应式属性系统会用`Proxy`实现。预计到这会成为明年面试的常考题，我们来看看`Proxy`，和它的好兄弟`Reflect`是个什么东西。

## Proxy

`Proxy`看起来好像是新东西，但其实它在ES6（ES2015）就提出来了。在平时的开发中，这个特性用得不多。

简单来说，`Proxy`可以自定义对目标对象的一些**原生操作**，比如属性赋值操作、获取属性操作等等。

`Proxy`支持自定义13种操作。

- `apply`：拦截函数调用操作，如`proxy()`；
- `constructor`：拦截`new`操作符，如`new proxy()`；
- `defineProperty`：拦截对对象的`Object.defineProperty()`操作；
- `deleteProperty`：拦截删除对象属性的操作；
- `get`：拦截对对象的读取属性操作，如`proxy.propName`；
- `getOwnPropertyDescriptor`，拦截对对象的`Object.getOwnPropertyDescriptor()`操作；
- `getPrototypeOf`：拦截读取代理对象原型的操作；
- `has`：拦截`prop in proxy`操作；
- `isExtensible`：拦截`Object.isExtensible()`操作；
- `ownKeys`：拦截`Object.keys()`、 `Object.getOwnPropertyNames()`、`Object.getOwnPropetySymbols`；
- `preventExtensions`：拦截`Object.preventExtensions()`操作；
- `set`：拦截设置对象属性值的操作，如`proxy.prop = 1`
- `setPrototypeOf()`：拦截`Object.setPrototypeOf()`操作。


1. [MDN - Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)