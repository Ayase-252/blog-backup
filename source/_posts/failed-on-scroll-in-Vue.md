---
title: Vue中无效的`scroll`事件handle
tags:
  - Web Development
  - Debug
  - Event
  - JavaScript
  - DOM
categories: Note
date: 2018-10-24 21:06:10
---


最近在实现文章列表懒加载的时候，遇到了监听组件内的`scroll`事件失效的情况。这个时候就有点诡异。其实问题在于`scroll`事件到底发生在什么地方。

<!--more-->

按照`scroll`事件的说法，该事件会被具备滑条的元素在滑动时触发。那么真的是我们绑定元素触发了`scroll`事件吗？

有的时候不是这样的。比如说在父组件在子组件的外边包了一层，比如说`div`。

```JavaScript
// Parent.vue
<template>
<div>
<child></child>
</div>
</template>

// Child.vue
<template>
<div v-on:scroll="onScroll">
</div>
</template>
```

此时如果滑动滑条的的话。不一定是`Child.vue`中的`div`的触发`scroll`，而是`Parent.vue`中的`div`触发滑条事件。由于外层的`div`会被`child`撑开而形成滑条。而触发`scroll`事件。此时，由于事件冒泡的机制，内部元素无法捕捉到事件。这种调试可以使用Chrome的performance调试工具查看`scroll`事件的root元素。
