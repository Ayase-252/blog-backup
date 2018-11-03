---
title: 探索Vue的实例初始化——生命周期
categories: Technology
tags:
    - Vue
---

## Vue实例的初始化

在Vue应用中，所有的故事都开始于`new Vue(...)`，即Vue的实例化。Vue在实例化过程中到底做了一些什么？本节将会阐明这一问题。

下面先看源代码：

```JavaScript
// Vue/src/core/instance/index.js
// https://github.com/vuejs/vue/blob/dev/src/core/instance/index.js

import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  // Vue对象必须由new Vue创建！
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  // 实际做的
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```

上述代码实在非常的简练。特别是大量`Mixin`使得逻辑复杂的代码看起来非常舒适。当然，本文的重点不是讨论这个。这里`Vue`只做了一件事情`this._init(options)`。我们将会潜入这个函数中，看看它做了什么。

## `this._init`的幕后

`_init`函数由`initMixin(Vue)`引入`Vue`示例中。（再次感叹。。）在我们继续深入之前，我先盗一张Vue示例生命周期（Lifecycle）的图。

![Life cycle of Vue instance](https://cn.vuejs.org/images/lifecycle.png)

过一会，我们在探索的过程中就可以邂逅这些所谓的生命周期钩子(Lifecycle Hook)。

从现在开始，让我们进入`_init`。

```JavaScript
// vue/src/core/instance/init.js
// https://github.com/vuejs/vue/blob/dev/src/core/instance/init.js
/* @flow */

import config from '../config'
import { initProxy } from './proxy'
import { initState } from './state'
import { initRender } from './render'
import { initEvents } from './events'
import { mark, measure } from '../util/perf'
import { initLifecycle, callHook } from './lifecycle'
import { initProvide, initInjections } from './inject'
import { extend, mergeOptions, formatComponentName } from '../util/index'

let uid = 0

export function initMixin (Vue: Class<Component>) {
  // 向Vue示例挂载_init方法
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // a uid
    vm._uid = uid++
    // 性能检查
    let startTag, endTag
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      startTag = `vue-perf-start:${vm._uid}`
      endTag = `vue-perf-end:${vm._uid}`
      mark(startTag)
    }

    // a flag to avoid this being observed
    vm._isVue = true
    // merge options
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options)
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm)
    } else {
      vm._renderProxy = vm
    }
    // expose real self
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    // 调用beforeCreate钩子，在此之前完成了`LifeCycle`，`Event`，`Render`的初始化
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    // 调用created钩子，在此之前完成了`Injections`，`State`，`Provide`的初始化
    callHook(vm, 'created')

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      vm._name = formatComponentName(vm, false)
      mark(endTag)
      measure(`vue ${vm._name} init`, startTag, endTag)
    }

    // 如果指定了挂载对象，执行$mount挂载
    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
}
```

在这段初始化的过程中，我们看见了`Vue`实例的初始化的过程。在`Vue`实例初始化的过程中，`LifeCycle`、`Events`、`Render`、`Injections`、`State`、`Provide`被依次初始化。`beforeCreate`、`created`钩子在初始化的过程中依照生命周期图所述的样子调用。但是问题来了，这些名词被初始化的时候到底发生了什么？作为使用者的我们，在生命周期钩子里到底能够得到或者操作什么东西？这是本文（也可能是一系列文章，太长了orz）想要搞清楚的东西。

作为开始，按初始化的顺序，我们先来看看`InitLifeCycle`干的是什么事情。

## `InitLifeCycle`

说起`LifeCycle`，立即会想到的是前面那张LifeCycle的图。然后会谈起的是：首先，Vue....然后执行了...,balabala。

作为使用者，我们最为关心的是生命周期钩子函数中我们要操作的组件已经“进展”到什么程度了。比如，在`created`里我们可以使用`data`属性了吗？（画外音：可以）。为了解决这一问题，我们再一次潜入源码中。

```JavaScript
// Vue/src/core/instance/lifecycle.js
// https://github.com/vuejs/vue/blob/dev/src/core/instance/lifecycle.js
import config from '../config'
import Watcher from '../observer/watcher'
import { mark, measure } from '../util/perf'
import { createEmptyVNode } from '../vdom/vnode'
import { updateComponentListeners } from './events'
import { resolveSlots } from './render-helpers/resolve-slots'
import { toggleObserving } from '../observer/index'
import { pushTarget, popTarget } from '../observer/dep'

import {
  warn,
  noop,
  remove,
  handleError,
  emptyObject,
  validateProp
} from '../util/index'

export let activeInstance: any = null
export let isUpdatingChildComponent: boolean = false

export function initLifecycle (vm: Component) {
  const options = vm.$options
  
  // locate first non-abstract parent
  let parent = options.parent
  if (parent && !options.abstract) {
    while (parent.$options.abstract && parent.$parent) {
      parent = parent.$parent
    }
    parent.$children.push(vm)
  }

  vm.$parent = parent
  vm.$root = parent ? parent.$root : vm

  vm.$children = []
  vm.$refs = {}
  // watcher，实现数据绑定的关键
  vm._watcher = null
  vm._inactive = null
  vm._directInactive = false
  // 一些关于生命周期的flag
  vm._isMounted = false
  vm._isDestroyed = false
  vm._isBeingDestroyed = false
}
```

可以看到`initLifeCycle`代码就是做了一些属性初始化的工作，也将新的实例合并到了组件树里面。其余的有意思的地方的话，emmmmm，好像没有看到。但是注意到在我们最初的`index.js`中，执行了一句`lifecycleMixin(Vue)`。`lifeCycleMixin`在我们实际初始化实例的时候已经执行过了，所以很有必要看一下里面做了什么。所幸这一函数就在`initLifecycle`的下面。

```JavaScript
// Vue/src/core/instance/lifecycle.js
// https://github.com/vuejs/vue/blob/dev/src/core/instance/lifecycle.js
export function lifecycleMixin (Vue: Class<Component>) {
  // 挂载更新视图的方法_update
  Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    const vm: Component = this
    const prevEl = vm.$el
    const prevVnode = vm._vnode
    const prevActiveInstance = activeInstance
    activeInstance = vm
    vm._vnode = vnode
    // Vue.prototype.__patch__ is injected in entry points
    // based on the rendering backend used.
    if (!prevVnode) {
      // initial render
      vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
    } else {
      // updates
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
    activeInstance = prevActiveInstance
    // update __vue__ reference
    if (prevEl) {
      prevEl.__vue__ = null
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm
    }
    // if parent is an HOC, update its $el as well
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el
    }
    // updated hook is called by the scheduler to ensure that children are
    // updated in a parent's updated hook.
  }

  // 挂载$forceUpdate，强制令vm._watcher执行更新
  Vue.prototype.$forceUpdate = function () {
    const vm: Component = this
    if (vm._watcher) {
      vm._watcher.update()
    }
  }
  
  // 挂载组件销毁的方法$destroy
  Vue.prototype.$destroy = function () {
    const vm: Component = this
    if (vm._isBeingDestroyed) {
      return
    }
    callHook(vm, 'beforeDestroy')
    vm._isBeingDestroyed = true
    // remove self from parent
    const parent = vm.$parent
    if (parent && !parent._isBeingDestroyed && !vm.$options.abstract) {
      remove(parent.$children, vm)
    }
    // teardown watchers
    if (vm._watcher) {
      vm._watcher.teardown()
    }
    let i = vm._watchers.length
    while (i--) {
      vm._watchers[i].teardown()
    }
    // remove reference from data ob
    // frozen object may not have observer.
    if (vm._data.__ob__) {
      vm._data.__ob__.vmCount--
    }
    // call the last hook...
    vm._isDestroyed = true
    // invoke destroy hooks on current rendered tree
    vm.__patch__(vm._vnode, null)
    // fire destroyed hook
    callHook(vm, 'destroyed')
    // turn off all instance listeners.
    vm.$off()
    // remove __vue__ reference
    if (vm.$el) {
      vm.$el.__vue__ = null
    }
    // release circular reference (#6759)
    if (vm.$vnode) {
      vm.$vnode.parent = null
    }
  }
}
```

在`lifeCycleMixin`中，向Vue实例添加了`._update`、`.$forceUpdate`、`.$destroy`方法。要完全理解里面的逻辑需要一点虚拟DOM以及数据绑定的知识。