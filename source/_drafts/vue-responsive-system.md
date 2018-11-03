---
title: 初探Vue的响应式系统——数据的双向绑定
categories: Technology
tags:
    - Vue
---

作为MVVM框架，Vue的核心系统之一就是其响应式系统，即平时所谓的“Vue如何实现双向绑定这一问题”。本文将会step by step的探究Vue是如何实现响应式系统的。

## 简单的数据双向绑定思路

想到要某一个变量随着另外一个变量而变，一个很自然的想法就是实现观察者模式。一个数据订阅观察者，然后当另外一个数据更新的时候触发通知更改另外一个数据。要实现双向绑定的话，可以实现两个观察者，互相观察数据的变化。

首先，我们构建一个简单的观察者。

```JavaScript
class Observer {
  constructor() {
    this.callbacks = []
  }
  subscribe(callback) {
    this.callbacks.push(callback)
  }
  notify(...args) {
    for(callback of this.callbacks) {
      callback(...args)
    }
  }

}
```

接下来，我们想要绑定`objA.a`与`objB.b`元素。

```JavaScript
let objA = { a: 2 }
let objB = { b: 3 }

const observerA = new Observer()
const observerB = new Observer()

const dataStorage = {
  a: objA.a,
  b: objB.b
}

function changeA (val) {
  if (dataStorage.a !== val) {
    dataStorage.a = val
  }
}

function changeB (val) {
  if (dataStorage.b !== val) {
    dataStorage.b = val
  }
}

observerA.subscribe(changeB)
observerB.subscribe(changeA)

Object.defineProperty(objA, 'a', {
  get () {
    return dataStorage.a
  },
  set (val) {
    if (dataStorage.a !== val) {
      observerA.notify(val)
    }
  }
})

Object.defineProperty(objB, 'b', {
  get () {
    return dataStorage.b
  },
  set (val) {
    if (dataStorage.b !== val) {
      observerB.notify(val)
    }
  }
})

console.log(objA.a) // 2
console.log(objB.b) // 3

objA.a = 6
console.log(objB.b) // 6
objB.b = 1
console.log(objA.a) // 1
```

代码很冗长是吧？而且含有非常多的Copy-and-paste代码，必须干掉它们。但是在优化之前，我们先来看看里面比较奇怪一点，就是`dataStorage`为什么要出现。由于`get`方法里面不能够采用`return this.a`的方法返回数据（递归栈溢出），因此需要一种把数据放在外部的方法。这里我们先简单地采用一个全局变量存储。

接下来，我们来一点一点消灭掉冗余的代码，首先，我们可以把观察者与定义观察者的部分合并成一个类，我们命名为`Watcher`。

```JavaScript
class Watcher {
  constructor(value) {
    this.value = value
    this.observer = new Observer();
  }

  subscribe(watcher) {
    this.observer.subscribe(watcher.onChange.bind(watcher)) //需要bind传入的对象，注意notify调用callback的方式
  }

  onSet(value) {
    this.value = value
    this.observer.notify(value)
  }

  onGet() {
    return this.value
  }

  onChange(value) {
    this.value = value
  }
}
```

我们来实际使用一下我们的新类`Watcher`：
```JavaScript
let objA = { a: 2 }
let objB = { b: 3 }
const watcherA = new Watcher(objA.a)
const watcherB = new Watcher(objB.b)

watcherA.subscribe(watcherB)
watcherB.subscribe(watcherA)

Object.defineProperty(objA, 'a', {
  get () {
    return watcherA.onGetter()
  },
  set (val) {
    watcherA.onSet(val)
  }
})

Object.defineProperty(objB, 'b', {
  get () {
    return watcherB.onGetter()
  },
  set (val) {
    watcherB.onSet(val)
  }
})

console.log(objA.a) // 2
console.log(objB.b) // 3

objA.a = 6
console.log(objB.b) // 6
objB.b = 1
console.log(objA.a) // 1
```

看起来好很多，但是`Object.defineProperty`仍然是Copy-and-paste的。这里再进行一次优化。我们将创建`watcher`的部分与`Object.defineProperty`给包装起来。

```JavaScript
function defineReactive (obj, prop) {
  const watcher = new Watcher(obj[prop])
  Object.defineProperty(obj, prop, {
    get () {
      return watcher.onGet()
    },
    set (val) {
      return watcher.onSet(val)
    }
  })
  return watcher
}
```

我们来使用这一个函数来完成双向绑定任务。

```JavaScript
let objA = { a: 2 }
let objB = { b: 3 }

const watcherA = defineReactive(objA, 'a')
const watcherB = defineReactive(objB, 'b')

watcherA.subscribe(watcherB)
watcherB.subscribe(watcherA)

console.log(objA.a) // 2
console.log(objB.b) // 3

objA.a = 6
console.log(objB.b) // 6
objB.b = 1
console.log(objA.a) // 1
```

是不是感觉明晰了不少。只要返回的`watcher`互相订阅就可以建立起双向绑定的关系。我们甚至可以遍历一个对象的所有属性（正如`component`实例中的`data`）属性一样。

## Vue的实现

`defineReactive`函数定义在`vue/src/core/observer/index.js`。

```JavaScript
/**
 * Define a reactive property on an Object.
 */
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get // 获取getter回调，在我们实现的版本中相当与watcher.onGet
  const setter = property && property.set // 获取setter回调，在我们实现的版本中相当于watcher.onSet
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key] //如果没有getter或者定义了setter（需要值做脏值检测）的情况
  }

  // let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val // 回调getter
      // if (Dep.target) {
      //   dep.depend()
      //   if (childOb) {
      //     childOb.dep.depend()
      //     if (Array.isArray(value)) {
      //       dependArray(value)
      //     }
      //   }
      // }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      // 若新值与旧值相同
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      if (setter) {
        setter.call(obj, newVal) //回调setter
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
```

这里我们注释掉依赖收集的代码以突出双向绑定的实现。基本原理看起来差不多。但是Vue的实现里面并没有`Watcher`，并且多出了`Dep`。以及我们并不知道所谓的`getter`和`setter`是什么。

大家都知道组件中的`data`属性内的属性是响应式的，我们从其初始化过程出发，去搞清楚这些问题。

```JavaScript
function initData (vm: Component) {
  let data = vm.$options.data // 获取data属性
  data = vm._data = typeof data === 'function'   //判断data是否是函数...函数的效果想必大家以及知道了
    ? getData(data, vm)
    : data || {}
  if (!isPlainObject(data)) {
    data = {}
    // process.env.NODE_ENV !== 'production' && warn(
    //   'data functions should return an object:\n' +
    //   'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
    //   vm
    // )
  }
  // proxy data on instance // Proxy..好像很有趣的样子
  // const keys = Object.keys(data)
  // const props = vm.$options.props
  // const methods = vm.$options.methods
  // let i = keys.length
  // while (i--) {
  //   const key = keys[i]
  //   if (process.env.NODE_ENV !== 'production') {
  //     if (methods && hasOwn(methods, key)) {
  //       warn(
  //         `Method "${key}" has already been defined as a data property.`,
  //         vm
  //       )
  //     }
  //   }
  //   if (props && hasOwn(props, key)) {
  //     process.env.NODE_ENV !== 'production' && warn(
  //       `The data property "${key}" is already declared as a prop. ` +
  //       `Use prop default value instead.`,
  //       vm
  //     )
  //   } else if (!isReserved(key)) {
  //     proxy(vm, `_data`, key)  
  //   }
  // }
  // observe data
  observe(data, true /* asRootData */)
}
```
注释掉检查的部分，代码已经不剩下多少了，我们只用关注`observe`干了什么。

```JavaScript
export function observe (value: any, asRootData: ?boolean): Observer | void {
  // if (!isObject(value) || value instanceof VNode) {
  //   return
  // }
  let ob: Observer | void
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) { //已经存在observer
    ob = value.__ob__
  } else if (
    // shouldObserve &&
    // !isServerRendering() &&
    // (Array.isArray(value) || isPlainObject(value)) &&
    // Object.isExtensible(value) &&
    // !value._isVue
  ) {
    ob = new Observer(value)  //创建一个新的Observer
  }
  if (asRootData && ob) {
    ob.vmCount++
  }
  return ob
}
```

注释掉一串复杂的条件之后。嗯，`Observer`又是什么东西呢？

```JavaScript
/**
 * Observer class that is attached to each observed
 * object. Once attached, the observer converts the target
 * object's property keys into getter/setters that
 * collect dependencies and dispatch updates.
 */
export class Observer {
  value: any;
  dep: Dep;
  vmCount: number; // number of vms that has this object as root $data

  constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    this.vmCount = 0
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
      const augment = hasProto
        ? protoAugment
        : copyAugment
      augment(value, arrayMethods, arrayKeys)
      this.observeArray(value) //如果传入值是数组
    } else {
      this.walk(value) // 遍历所有的可枚举的key，并且defineReactive(obj, keys[i])，见下。
    }
  }

  /**
   * Walk through each property and convert them into
   * getter/setters. This method should only be called when
   * value type is Object.
   */
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }

  /**
   * Observe a list of Array items.
   */
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}
```

