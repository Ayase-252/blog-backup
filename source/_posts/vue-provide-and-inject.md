---
title: Vue的Provide与Inject机制
tags:
  - Vue
categories: Technology
date: 2018-11-25 00:38:35
---


`Vue`中父组件到子组件的通信主要由子组件的`props`属性实现。但是在一些情况下，父组件无法直接向子组件的`props`传值。比如子组件通过父组件的`slot`进入父组件，父组件根本不知道子组件是谁，更不用说用子组件的`props`了。这时应该怎么办呢？`Vue`在`2.2.0`版本引入了`provide`与`inject`，正好适合处理这一情况。

<!--more-->

## 什么是provide与inject

用[文档](https://cn.vuejs.org/v2/api/#provide-inject)的话说：

> `provide`/`inject`需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。如果你熟悉 React，这与 React 的上下文特性很相似。

这就是说从父组件的`provide`属性传入一个对象，子组件（或者是孙组件，只要是子级组件）可以用`inject`属性接收父组件的`provide`属性。比如

```html
// main.vue
<template>
    <c1 message="hello world">
        <c2></c2>
    </c1>
</template>

// c1.vue
<template>
  <div id="c1">
    <slot></slot>
  </div>
</template>

<script>
export default {
  props: ['message'],
  provides () {
    return {
      message: this.message
    }
  }
}
</script>

// c2.vue
<template>
  <div id="c2">
      {{ message }}
  </div>
</template>

<script>
export default {
  inject: ['message']
}
</script>
```

上面的`main`组件会被渲染为:

```html
<div id="c1">
  <div id= "c2">hello world</div>
</div>
```

可以看到，`c1`组件在不清楚子组件是什么的情况下，将它的`props`中的`message`传给了`c2`组件。在这里`c1`组件就像是一个数据源一样，为子组件提供数据。但是，`c1`组件提供的数据仅在`c1`的子孙组件中可见，因此可以算作是有作用域限定的数据源。

## 父到子孙组件方向的数据流

父到子孙组件方向是`provide/inject`机制设计时的数据流方向。我们可能会猜想，在父组件中更改`provide`的值，子组件会响应式的发生改变。但是注意到文档中话。

> 提示：`provide`和`inject`绑定并不是可响应的。这是刻意为之的。然而，如果你传入了一个可监听的对象，那么其对象的属性还是可响应的。

这意味着，如果`provide`的值不是可监听对象时，在父组件中更改`provide`的值，子组件不会发生任何变化。比如模板仍然为上面那个例子的模板，`message`的值是一个`props`属性，不是可监听对象，如果我们在`c1`的`mounted`钩子函数里改变`message`的值。如:

```html
// c1.vue
<script>
export default {
  //...
  mounted () {
    setTimeout( () => {
      this.message = 'Opps, it would not be rendered'
    }, 1000)
  }
}
</script>
```

子组件不会响应修改后的值。

但是如果`provide`的值是一个可监听对象呢？请看一下例子：

```html
<script>
// c1.vue
export default {
  data () {
    return {
      message: 'hello world'
    }
  },
  provide () {
    messageData: this.$data
  },
  mounted () {
    setTimeout(() => {
      this.message = 'I can show in c2.'
    }, 10000)
  }
}
</script>

// c2.vue
<template>
  <div id="c2">
    {{ messageData.message }}
  </div>
</template>

<script>
export default {
  inject: ['messageData']
}
</script>
```

此时在`c1`挂载10s后，子组件将会显示`I can show in c2`。为什么呢？`c2`中`messageData`实际上就是`c1`实例的`this.$data`。而`this.$data`上有`message`的响应式`getter`与`setter`。所以`c2`的视图会被`message`的`dep`收集，因此在`c1`中更新`message`，`c2`的视图也会更新。如果对此处不熟悉，可以看一看关于`Vue`数据绑定的文章。

## Vue的源码实现

首先我们来看`provide`与`inject`的初始化，在`src/core/instance/init.js`里，我们能够看到它们初始化的过程：

```javascript
//src/core/instance/init.js
Vue.prototype._init = function (options?: Object) {
  // ...
  callHook(vm, 'beforeCreate')
  initInjections(vm) // resolve injections before data/props
  initState(vm)
  initProvide(vm) // resolve provide after data/props
  callHook(vm, 'created')
  // ...
}
```

可以看到在一个组件内，`initInjections`是先于`initProvide`调用的，但是从整个组件树的初始化顺序来看，父组件的`initProvide`的调用要先于子组件的`initInjections`。为了理解上的方便，我们先来看`initProvide`。

### initProvide

`initProvide`定义在`src/core/instance/inject.js`中：

```javascript
// src/core/instance/inject.js
export function initProvide (vm: Component) {
  const provide = vm.$options.provide
  if (provide) {
    vm._provided = typeof provide === 'function'
      ? provide.call(vm) // 如果provide是函数，在本组件上调用
      : provide
  }
}
```

`initProvide`函数很简单，就是将组件的`provide`对象放到`vm._provided`中。这里兼顾了`provide`为函数与为对象两种情况。与文档中所述的一致：

> 类型：
> provide：`Object | () => Object`

注意到，`initProvide`方法中只是进行了简单的复制。在大多数情况下，如果要把父级的响应式属性作为`provide`，此时只有值被复制进去。Vue并没有对`_provided`属性做响应式处理（熟悉源码的同学应该知道`defineReactive`方法）。因此，`provide`是非响应式的。

### initInjections

`initInjections`定义同一个文件中：

```javascript
// src/core/instance/inject.js
export function initInjections (vm: Component) {
  const result = resolveInject(vm.$options.inject, vm)
  if (result) {
    // 不会观察vm.key
    toggleObserving(false)
    Object.keys(result).forEach(key => {
      /* istanbul ignore else */
      if (process.env.NODE_ENV !== 'production') {
        // 如果不是`production`环境中，当试图更改inject的值时会报错。
        defineReactive(vm, key, result[key], () => {
          warn(
            `Avoid mutating an injected value directly since the changes will be ` +
            `overwritten whenever the provided component re-renders. ` +
            `injection being mutated: "${key}"`,
            vm
          )
        })
      } else {
        defineReactive(vm, key, result[key])
      }
    })
    toggleObserving(true)
  }
}
```

这里出现了一个`resolveInject`函数，还是在这个文件里:

```javascript
// src/core/instance/inject.js
export function resolveInject (inject: any, vm: Component): ?Object {
  if (inject) {
    // inject is :any because flow is not smart enough to figure out cached
    const result = Object.create(null)
    // 如果支持Symbol
    const keys = hasSymbol
      ? Reflect.ownKeys(inject).filter(key => {
        /* istanbul ignore next */
        return Object.getOwnPropertyDescriptor(inject, key).enumerable
      })
      : Object.keys(inject)

    // 遍历所有inject中的key
    for (let i = 0; i < keys.length; i++) {
      const key = keys[i]
      const provideKey = inject[key].from
      let source = vm
      // 从本级的._provided中查找（注意到inject先于provide初始化，因此事实上是
      // 从父组件开始查找），如果本级没找到，就到父级的._provided中查找
      while (source) {

        if (source._provided && hasOwn(source._provided, provideKey)) {
          result[key] = source._provided[provideKey]
          break
        }
        source = source.$parent
      }
      // 如果整条链都没有找到，尝试使用`default`属性构建fallback值
      if (!source) {
        if ('default' in inject[key]) {
          const provideDefault = inject[key].default
          result[key] = typeof provideDefault === 'function'
            ? provideDefault.call(vm)
            : provideDefault
        } else if (process.env.NODE_ENV !== 'production') {
          warn(`Injection "${key}" not found`, vm)
        }
      }
    }
    // 返回result对象，它的key是inject中指定的key
    return result
  }
}
```

实际上，`resolveInject`是实现`provide/inject`的核心函数。它从父/祖父组件中把`provide`的值捕捉下来，之后`initInjections`利用这一结果，在自身组件上定义响应式属性。注意到在定义响应式属性之前，`toggleObserving(false)`。这意味着`inject`的值里面是没有`__ob__`的。也就是说，当**更改`inject`的值会触发视图的改变**，而**更改`inject`对象的属性不会触发视图改变**。当然，最佳实践是不要在子组件里更改`inject`。

纵观整个过程，`provide/inject`机制是非响应式的，即`provide`与`inject`之间没有绑定。具体的值是在**子组件初始化**过程中决定的。

## 总结

`provide/inject`提供了一种新的组件间通信的方法。它允许父组件向子孙组件间进行跨层级的数据分发。但是`provide/inject`是非响应式的，如果要子孙组件根据父组件的值进行改变，`provide/inject`机制不是一个好的选择。此时可以使用`Vuex`来管理状态。

## 参考资料

1. [API - Vue.js](https://cn.vuejs.org/v2/api/#provide-inject)
2. [Vue源码分析之Observer](https://segmentfault.com/a/1190000009054946)
