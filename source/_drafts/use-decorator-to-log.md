---
title: 使用Decorator进行数据埋点
categories:
  - Programming
  - JavaScript
tags:
  - Decorator
  - Proxy
  - Reflect
  - JavaScript
---

数据埋点是前端开发中的常见需求。但是，埋点处数据的采集与软件主要业务逻辑常常是没有关系的。
如何在不改动软件主要业务逻辑的同时，完成埋点的工作呢？

<!--more-->

## 装饰器模式

一说到要在不侵入的情况下扩展一个方法，
我就想起了[装饰器模式](https://en.wikipedia.org/wiki/Decorator_pattern)（咳咳）。
使用装饰器模式可以在运行时扩展一个类的行为。其具体原理很简单，
返回一个包裹着被装饰类逻辑与扩充逻辑的类。这个类与被装饰类的接口一模一样。
因此在运行时可以相互替换。

## 小需求：组件 mount 时打一个 log

假设我们有这样一个小需求，当一个 React 组件，比如`Article`组件`mount`后，
在控制台打一个`log`，表示这个组件已经`mount`了。

最简单的实现莫过于：

```jsx
class Article extends React.Component {
  constructor(props) {
    super(props)
  }

  componentDidMount() {
    console.log('I have mounted. Hohoho')
    // Do some other things
  }
}
```

这样，在每一个`Article`组件`mount`的时候，控制台上都会出现一条
`I have mounted. Hohoho`。 今天的需求完成，可以收工回家了~皆大欢喜。

---

第二天，我们又接到了一个需求。要不我们把用户的在页面上的浏览时间记下来？
然后传到某个 API 上。我们一想，challenge accept。

```jsx
class Article extends React.Component {
  constructor(props) {
    super(props)
    this.states.openTime = 0
  }

  componentDidMount() {
    console.log('I have mounted. Hohoho')
    // Do some other things
    this.setState({
      entryTime: new Date()
    })
  }

  componentWillUnmount() {
    const timeOnPage = new Date() - this.setState.openTime
    fetch({
      url: '/someAPI',
      method: 'POST',
      data: {
        timeOnPage
      }
    })
  }
}
```

emmm，现在`Article`已经耦合进有点复杂的数据采集逻辑了（`mount`时记录打开时间，
`Unmount`时计算页面停留时间）。心想着如果只是一个页面还好，收工下班吧。

---

第三天，最怕的事情还是来了。
老大让将把另外一个组件`Comment`的用户页面停留时间也采集上来。
我们是将数据采集的逻辑 copy&paste 一遍呢？cp 是编程中最应该避免的事情，
我们可以用什么更优雅的方式实现呢？

## 装饰器模式

由于数据采集与主要的业务逻辑没有什么关系，如果我们有一种方式能够在
**不侵入组件的原有逻辑的基础上将数据采集的逻辑附加在正常逻辑上**，
这种方式应该就是我们寻找的最佳方法。

有一种设计模式————
[装饰器模式](https://design-patterns.readthedocs.io/zh_CN/latest/structural_patterns/decorator.html)
就是专门处理这样的问题的。通过装饰器，我们可以在运行时向类添加新的行为。
装饰器是对用户代码透明的，即在用户在使用被装饰的类的时候，
不会注意到装饰器的存在。

### 简单的装饰器

我们可以使用 Vanilla JavaScript 实现一个简单的前后打 log 的装饰器：

```javascript
const logger = fn => (...args) => {
  console.log('before function call')
  const retVal = fn(...args)
  console.log('after function call')
  return retVal
}

let doSomething = () => {
  console.log('eat eat!')
}
doSomething = logger(doSomething)

doSomething()
```

此时，console 会输出:

```plain
before function call
eat eat!
after function call
```

通过这样做，我们实现了`logger`逻辑与正常业务逻辑的分离。但是存在一点点小问题，
我们要包装一个函数需要：

```javascript
let originFunc = () => {
  // do something
}
originFunc = decorator(originFunc)
```

这里`originFunc = decorator(originFunc)`感觉上有一些不好。

来看一下另外一门著名动态语言 Python 对装饰器的原生实现。

```python
def logger(fn):
    def wrapper(*args, **kargs):
        print('before function call')
        fn(*args, **kargs)
        print('after function call')
    return wrapper

@logger
def do_something():
    print('oopps')

do_something()
```

`@logger`是一个语法糖。上面`def do_something`的函数定义部分与下面代码等价。

```python
def do_something():
    print('oopps')
do_something = logger(do_something)

```

## 用 Decorator 重构
