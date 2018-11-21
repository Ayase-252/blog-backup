---
title: 从实例学Vue的生存周期
categories: Technology
tags:
    - Vue
---

人生可以用两个字来概括——“生死”。每一个人的人生都是从出生而起，从死亡而终。作为自然的终极定律，没有人可以违背这一定律。在Vue的世界中，每一个组件也有其“生存周期”，描述了这个组件在Vue世界中的“一生”。今天，我们会从屏幕外观察一位名叫`Bob`的Vue组件的一生。

<!--more-->

## Bob自传

`Bob`是一位程序员，与平常人一样，6岁开始上学，18岁进入大学校园。在大四的时候觉得自己的技术水平还不够，于是考了研究生。25岁研究生毕业开始踏入社会。在社会兢兢业业工作几十年，然后离开这个世界。我们把人的一生抽象成一个Vue组件。就像下面这样。

{% raw %}
<iframe height='194' scrolling='no' title='A Life Story of Bob' src='//codepen.io/ayase-252/embed/pQrpzE/?height=194&theme-id=0&default-tab=js,result' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/ayase-252/pen/pQrpzE/'>A Life Story of Bob</a> by Qingyu Deng (<a href='https://codepen.io/ayase-252'>@ayase-252</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
{% endraw %}

我们把Vue的生命周期对应到人的一生，出生之前(`beforeCreate`)，出生之时(`create`)，出母体之前(`beforeMounted`)，出母体之后(`mounted`)，岁数增长时(`beforeUpdate`, `updated`)以及奄奄一息，只有最后一口气之时(`beforeDestroy`，`destroyed`)。

## 胚胎阶段beforeCreate,created

正如人从胚胎细胞开始发育一样，`Vue`组件不是从我们`new Vue.component('xxx', {...})`就如此的全能，它要经过一系列的初始化的过程，才会成为我们希望它成为的那个能抗需求的组件。我把生命钩子`beforeCreate`以及`created`之间的阶段称为组件的胚胎阶段，因为在这个阶段，有一些非常基础的属性是拿不到的。

### beforeCreate

`beforeCreate`是我们能够操作`Vue`组件的最早的一个钩子，在此之前，`Vue`组件完成了**生命周期初始化**、**事件初始化**与**渲染初始化**。源码如下

```typescript
// vue/src/core/instance/init.js
Vue.prototype._init = function(options?: Object) {
    //...
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    //...
}
```

#### 生命周期初始化

生命周期化主要是确定了组件之间的关系，如组件的根组件(`vm.$root`)、父组件(`v,.$parent`)分别是谁？同时初始化了子组件以及`vm.$ref`。一些与生命周期密切的标志变量被初始化（这些变量以`_`开头，表示是私有属性，因此我们就不深究了）

生命周期初始化使得以下API可用：

|API| 解释 |
|--|--|
|`vm.$root`| 当前组件树的根 Vue 实例。如果当前实例没有父实例，此实例将会是其自己。|
|`vm.$parent`| 父实例，如果当前实例有的话。|

就如人的一生开始一样，当精卵细胞结合的一刻，我们的父母就被确定下来。我们的人生属性也被重置为默认值。

#### 事件初始化

事件初始化不会带来新的API可用。其实这段代码以我现在的水平，看不太懂。

```typescript
export function initEvents (vm: Component) {
  vm._events = Object.create(null)
  vm._hasHookEvent = false
  // init parent attached events
  const listeners = vm.$options._parentListeners
  if (listeners) {
    updateComponentListeners(vm, listeners)
  }
}
```

#### 渲染初始化

渲染初始化使得`vm.$slots`， `vm.$scopeSlots`可用。同时`vm.$attrs`，以及`vm.$listeners`被定义为响应式属性。

`vm.$attrs`包含了父作用域中不被`prop`识别的特性绑定。
`vm.$listeners`中包含了父作用域中的`v-on`事件监听器。

### created

随着进一步发育，`Vue`变得越来越强大，在`created()`调用之前，Vue组件完成了**依赖注入初始化**，**状态初始化**，**提供初始化**。源码如下：

```typescript
// vue/src/core/instance/init.js
Vue.prototype._init = function(options?: Object) {
    // ...
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created'
    // ...
}
```