---
title: 你所不知道的scroll事件：为什么scroll事件会失效？
tags:
  - Web Development
  - JavaScript
  - Event
categories: Programming
date: 2018-11-20 23:46:28
---


在Web开发中，`Scroll`事件是一个很重要的事件。通过`Scroll`事件，我们可以通过一些方式获知视口在页面的位置。知道这个信息可以帮助我们判断很多东西，如用户即将浏览到页面底部，是不是该调用API加载一些新的内容等等。

在实际运用中，`scroll`事件常常带来很多迷惑。最近在开发练手的博客项目时就遇到了一个问题，在`Vue`中的组件绑定`scroll`事件，事件处理函数似乎不会触发。这个问题好像困扰过很多人，谷歌`Vue 滚动事件失效`能发现很多相关内容。比如说[这一个问题](https://segmentfault.com/q/1010000009119633)，可以一个个去试他们给出的解决方案，总会发现几个有用的。但是作为开发者，知其然还要知其所以然。那么，本文就来探究一下这个`scroll`事件的背后到底有什么不为人知的东西。
<!--more-->

## JavaScript事件模型

大家都知道，JavaScript事件有两个阶段——（1）**捕获阶段(Capture Phase)**（2）**冒泡阶段(Bubble Phase)**。捕获阶段是事件从`document`到传递到目标元素的过程，而冒泡阶段是事件从目标元素传递到`document`的过程。下面是《高程》上的一张图，很明了地描述了整个过程。

![capture-bubble-phase](capture-bubble-phase.png)

在平时，大家一般是监听事件的冒泡阶段，即`elem.addEventListener('scroll', handler)`。`Vue`中也提供了`v-on`指令监听某个事件，默认用的也是事件的冒泡阶段。那么这里会有什么坑呢？

根据[MDN](https://developer.mozilla.org/zh-CN/docs/Web/Events/scroll)对`scroll`事件的描述，我们发现了惊人的事实：

> 冒泡: `element`的`scroll`事件**不冒泡**, 但是`document`的`defaultView`的`scroll`事件冒泡

这句话的意思就是说，如果`scroll`的目标元素是一个元素的话，比如说是一个`div`元素。那么此时事件只有从`document`到`div`的捕获阶段以及`div`的冒泡阶段。如果尝试在父级监视`scroll`的冒泡阶段监视这一事件是无效的。如果`scroll`是由`document.defaultView`（目前`document`关联的`window`对象）产生的有冒泡阶段。但是由于其本身就是DOM树里最顶级的对象，因此只能在`window`里监视`scroll`的捕获阶段以及冒泡阶段。

注意到在元素为目标元素时，也在目标元素上监视`scroll`事件的捕获阶段以及冒泡阶段。两种情况其实内在是一致的。接下来我们会通过两个Demo证明这一点。

## 当scroll的目标元素为document.defaultView时

这是最常见的情形，比如知乎，可以在console里执行`window.addEventListener('scroll', e => console.log(e.target), true)`，然后滑动滚动条就可以看见效果。

这里我们自己写一个小型的HTML文档。

```html
<body>
  <div id="root">
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
    <p>Hello world</p>
  </div>
  <script src="scroll-in-window.js"></script>
</body>
```

（大段复制代码搞得我像灌水一样，反正是自己博客，灌就灌吧）那么多的`Hello world`是为了让浏览器产生滚动条，所需要的数量由屏幕分辨率而定。。

接下来，我们分别在`window`与`div#root`上监听`scroll`事件的捕获阶段以及冒泡阶段；

```javascript
// scroll-in-window.js

window.addEventListener('scroll', e => console.log('scroll handler is triggered in window during capture phase.'), true)
window.addEventListener('scroll', e => console.log('scroll handler is triggered in window during bubble phase.'))

const root = document.getElementById('root')
root.addEventListener('scroll', e => console.log('scroll handler is triggered in root during capture phase.'), true)
root.addEventListener('scroll', e => console.log('scroll handler is triggered in root during bubble phase.'))

```

在Chrome下运行如下：
![result-in-window](result-in-window.png)

这里我也准备了在线Demo，请戳[这里](https://codepen.io/ayase-252/pen/xQpGKj?editors=1011)

可以看到在全程只有`window`上监听`scroll`的**捕获阶段**以及**冒泡阶段**的回调函数执行了。这验证之前的结论：

> 如果`scroll`是由`document.defaultView`（目前`document`关联的`window`对象）产生的有冒泡阶段。但是由于其本身就是DOM树里最顶级的对象，因此只能在`window`里监视`scroll`的捕获阶段以及冒泡阶段。

## 当scroll的目标元素是元素时

这种情况也常见，我给出一种示例：一个双栏布局，左侧为导航栏，右侧为内容，父容器使用`Flex`布局，要求导航栏不随内容的滑动而滑动。HTML文档如下：
```html
<!DOCTYPE html>
<html>

<head>
  <title>Scroll In Element</title>
  <style>
    body {
      margin: 0;
    }

    #wrapper {
      width: 100%;
      height: 100vh;
      display: flex;
      overflow: hidden;
    }

    #left-col {
      overflow: hidden;
      flex: 0 0 300px;
      background-color: aqua;
    }

    #right-col {
      flex: 1;
      overflow: auto;
    }
  </style>
</head>

<body>
  <div id="wrapper">
    <div id="left-col">
      <nav>
        barabara
      </nav>
    </div>
    <div id="right-col">
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
      <p>Hello world</p>
    </div>
  </div>
  <script src="scroll-in-element.js"></script>
</body>

</html>
```

效果如下：
![demo](demo.png)

此时`scroll`事件的目标元素是`div#right-col`。接下来我们，分别在`window`,`div#wrapper`,`div#right-col`上监听`scroll`的捕获阶段以及冒泡阶段。

```javascript
const log = (elem, phase) => {
  console.log(`scroll handler is trigged in ${elem} during ${phase}`)
}

const bindEvent = (elem, elemName)  => {
  elem.addEventListener('scroll', log.bind(null, elemName, 'capture'), true)
  elem.addEventListener('scroll', log.bind(null, elemName, 'bubble'))
}

bindEvent(window, 'window')
const wrapper = document.querySelector('#wrapper')
bindEvent(wrapper, 'wrapper')
const rightCol = document.querySelector('#right-col')
bindEvent(rightCol, 'rightCol')
```

在Chrome上运行如下：
![result-in-element](result-in-element.png)

这里我也准备了在线Demo，请戳[这里](https://codepen.io/ayase-252/pen/BGJNXR?editors=1011#)

这里的运行结果很清楚的展示了`scroll`事件的传递轨迹，也证明了展示了我们之前说的：

> 如果`scroll`的目标元素是一个元素的话，比如说是一个`div`元素。那么此时事件只有从`document`到`div`的捕获阶段以及`div`的冒泡阶段。

## 总结

还用《高程》里那一副图总结一下`scroll`的事件流向：

![scroll-event-flow](scroll-event-flow.png)

当触发元素为`div`时，事件从`document`（在浏览器的实现上是`window`）开始向下传递，直到目标元素，此时捕获阶段结束。接下来在目标元素上进行一次冒泡阶段（4），然后不再冒泡。

对于`document.defaultView`产生的`scroll`事件一样，由于其本身就是顶层元素，在其本身上冒泡可以视为冒泡阶段结束。

### 建议

综上所述，在debug`scroll`事件失效问题的时候要清楚两个东西：

1. `scroll`事件的目标元素是什么？也可以说谁产生了`scroll`事件。如果实在不清楚的话，可以用Chrome开发者工具的`Performance`模块录制一下滚动动作，然后在`Event Log`里查看`scroll`事件的目标元素。
2. 监听`scroll`事件的元素是否在`scroll`事件的传递路径上，是否监听了正确的阶段？

弄清楚这些基本上就能够判定为什么`scroll`事件的回调触发不了了。

另外有一个万能的方法，就是在`window`上监听`scroll`的捕获阶段，即`window.addEventListener('scroll', handler, true)`。这也是很多类似问题中答主回答的答案。希望在看完本文之后能对其原理有所了解。

## 参考资料
1. Nicholas C. Zakas, Professional JavaScript for Web Developers, Third Edition
2. [scroll - MDN](https://developer.mozilla.org/zh-CN/docs/Web/Events/scroll)