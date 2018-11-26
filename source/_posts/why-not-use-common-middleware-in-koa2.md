---
title: Koa2中为什么不要使用普通中间件
tags:
  - Koa
categories: Technology
date: 2018-11-26 20:21:25
---


Koa2里支持两种中间件的写法，一种是使用ES7`async/await`语法的**异步函数**，这里我们称为**异步中间件**。一种是使用普通函数语法的**普通中间件**。尽管说官方支持两种写法，但是在实际应用中，我们可能不大常见普通中间件的写法。为什么呢？

<!--more-->

> tl;dr
> 1. 普通中间件可以有，但没必要；
> 2. 错误的普通中间件写法可能破坏洋葱模型；
> 3. 正确的普通中间件的写法可能和想象中的有点不太一样。

## Koa2的中间件洋葱模型

大家都知道Koa2的中间件的执行流类似洋葱，即请求会从第一个中间件开始然后暂停在`next`，接着执行第二个中间件，一直到最后一个中间件暂停在`next`。接着最后一个中间件从`next`恢复执行，执行完之后倒数第二个中间件从`next`的地方恢复执行，一直到第一个中间件从`next`恢复执行到完毕。

```javascript
const Application = require('koa')

const middleware_1 = async (ctx, next) => {
    console.log('middleware_1 before next')
    await next()
    console.log('middleware_1 after next')
}

const middleware_2 = async (ctx, next) => {
    console.log('middleware_2 before next')
    await next()
    console.log('middleware_2 after next')
}

const app = new Application()
app.use(middleware_1)
app.use(middleware_2)

app.listen(3000)

// 当有请求经过的时候，console会打印
// middleware_1 before next
// middleware_2 before next
// middleware_2 after next
// middleware_1 after next
```

## Koa2中间件洋葱模型的实现

这一部分我们简单介绍一下Koa2中间件洋葱模型的实现。当我们使用`app.use(middleware)`时，其实内部会将`middleware`推入一个数组保存:

```javascript
// koa/lib/application.js

module.exports = class Application extends Emitter {
    //...
  use(fn) {
    // if (typeof fn !== 'function') throw new TypeError('middleware must be   a function!');
    // if (isGeneratorFunction(fn)) {
    //   deprecate('Support for generators will be removed in v3. ' +
    //             'See the documentation for examples of how to convert old middleware ' +
    //             'https://github.com/koajs/koa/blob/master/docs/migration.md');
    //   fn = convert(fn);
    // }
    // debug('use %s', fn._name || fn.name || '-');
    this.middleware.push(fn);
    return this;
  }
    //...
}
```

忽略掉大量检查代码，`use`就做了一件事情，`this.middleware.push(fn)`。`this.middleware`是一个数组。

接下来，在`app.listen(3000)`的时候，`app.listen`会调用原生的`http`模块，然后使用`this.callback()`产生一个回调函数在请求到来的时候调用。接下来就是重头戏了，

```javascript
  callback() {
    const fn = compose(this.middleware);

    // if (!this.listenerCount('error')) this.on('error', this.onerror);

    const handleRequest = (req, res) => {
      // const ctx = this.createContext(req, res);
      return this.handleRequest(ctx, fn);
    };

    return handleRequest;
  }
```

注意到`compose(this.middleware)`，它将中间件数组组合起来，然后返回一个执行函数`fn`，当调用执行函数时，如`fn(ctx)`时，`ctx`就会以洋葱模型被各个中间件所处理。`this.handleRequest`干的就是这么一件事，可以看源码:

```javascript
  handleRequest(ctx, fnMiddleware) {
    // const res = ctx.res;
    // res.statusCode = 404;
    // const onerror = err => ctx.onerror(err);
    // const handleResponse = () => respond(ctx);
    // onFinished(res, onerror);
    return fnMiddleware(ctx).then(handleResponse).catch(onerror);
  }
```

这里我们重点来看一下`compose`函数，它是实现洋葱模型的核心函数。

### compose函数

`compose`函数并不在`koa`包里，而是在`koa-compose`包中，是一个非常短小精悍的函数。源码地址在[这里](https://github.com/koajs/compose)。我们只保留函数的核心部分如下：

```javascript
module.exports = compose

function compose (middleware) {

  return function (context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```

从上面可以看到我们的中间件是如何调用的，比如上面第一个中间件其实是以`middleware_1(context, dispatch(1))`的形式调用的。函数中使用到了`Promise`来保证各个中间件按照洋葱模型执行。具体原理可以自行推导一下。

## 一种错误的普通中间件写法

下面是一种很流行的**错误的**普通中间件写法：

```javascript
const one = (ctx, next) => {
  console.log('>> one');
  next();
  console.log('<< one');
}

const two = (ctx, next) => {
  console.log('>> two');
  next(); 
  console.log('<< two');
}

const three = (ctx, next) => {
  console.log('>> three');
  next();
  console.log('<< three');
}

app.use(one);
app.use(two);
app.use(three);
```

代码出自阮一峰老师的[Koa 框架教程](http://www.ruanyifeng.com/blog/2017/08/koa.html)。上面的代码是说明中间件栈（和洋葱模型差不多）这个概念。但是，代码成立的条件是**所有中间件都是同步的**。当其中有任意一个中间件是`async`的时候，代码就可能不是按照洋葱模型执行了。阮一峰老师提到**如果有异步操作（比如读取数据库），中间件就必须写成 async 函数。**可能就是因为这一原因。

下面是一个例子，在响应请求的过程中，我们需要记录响应请求所需要的时间，然后对用户传过来的密码进行哈希之后保存。对密码加盐哈希是一个非常耗时的操作，一般使用异步，这里为了模拟耗时，使用`setTimeout`延时1s。

```javascript
const Koa = require('koa')

const logger = (ctx, next) => {
  console.log('logger starts')
  const entryTime = new Date()
  next()
  const msUsed = new Date() - entryTime
  console.log(`response takes ${msUsed}ms.`)
}

const hashPassword = async (ctx, next) => {
  console.log('Hashing user password')
  await new Promise(resolve => {
    setTimeout(() => {
      console.log('password has been hashed')
      resolve()
    }, 1000)
  })
  ctx.body = "password secure"
  await next()
  console.log('Hashing password after next')
}

const app = new Koa()
app.use(logger)
app.use(hashPassword)

app.listen(3000)
```

当在浏览器里请求`localhost:3000`时，console输出如下：

```
logger starts
Hashing user password
response takes 1ms.
password has been hashed
Hashing password after next
```

可以看到`logger`的`next`之下的部分先于`hashPassword`的`next`之下的部分执行，这破坏了洋葱模型。

### 这种写法问题出现在哪？

注意到，`next()`返回的是一个`Promise`，在上面的写法中没有对这个`Promise`做任何处理，直觉告诉我们，这里极有可能会出现异步调用的顺序问题。事实上也是如此，我们来一步一步的分析问题产生的原因。

首先，`logger`函数被调用，传入的`next`参数是`dispatch(1)`。当`logger`执行`next()`，实际上`dispatch(1)`被执行。注意到`dispatch(i)`的返回值：

```javascript
return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
```

这个`Promise.resolve`是什么东西呢？

> Promise.resolve(value)方法返回一个以给定值解析后的Promise 对象。但如果这个值是个thenable（即带有then方法），返回的promise会“跟随”这个thenable的对象，采用它的最终状态（指resolved/rejected/pending/settled）；如果传入的value本身就是promise对象，则该对象作为Promise.resolve方法的返回值返回；否则以该值为成功状态返回promise对象。

很不巧，接下来的`fn`，即`hashPassword`是一个`async`函数，当运行`async`函数时，遇见`await`，函数会暂停执行并立即返回一个状态为`pending`的`Promise`。也就是说，`hashPassword`在第一个`await`的时候就返回了一个`pending`的`Promise`。

```javascript
const hashPassword = async (ctx, next) => {
  console.log('Hashing user password')
  >> await new Promise((resolve) => {
    setTimeout(() => {
      console.log('password has been hashed')
      resolve()
    }, 1000)
  })
  ctx.body = "password secure"
  await next()
  console.log('Hashing password after next')
}
```

然后问题就大了，`Promise.resolve(fn(context, dispatch.bind(null, i + 1)))`一看到`fn`返回了一个`Promise`，接着就把这个`Promise`再返回给上层。此时`dispatch(1)`结束。`logger`看到`next()`返回，然后兴高采烈地执行接下来的语句。此时`hashPassword`仍然在紧张地哈希用户的密码，甚至还没有调用`next()`函数。洋葱模型就此打乱。

### 那么正确的普通中间件的写法是？

既然官方支持普通中间件，那么正确的写法是什么呢？根据[文档](https://github.com/koajs/koa#middleware)一个正确的普通中间件的写法应该是：

```javascript
// Middleware normally takes two parameters (ctx, next), ctx is the context for one request,
// next is a function that is invoked to execute the downstream middleware. It returns a Promise with a then function for running code after completion.

app.use((ctx, next) => {
  const start = Date.now();
  return next().then(() => {
    const ms = Date.now() - start;
    console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
  });
});
```

可以看到正确处理`next()`返回的`Promise`才能够保证洋葱模型的正确性。。。

既然要处理`Promise`的话....为什么不直接使用`async/await`呢？

```javascript
app.use(async (ctx, next) => {
  const start = Date.now();
  await next();
  const ms = Date.now() - start;
  console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
});
```

上面的代码明显要比普通中间件的写法要好看一点。

## 总结

文章可能标题党了一点，Koa里面的中间件非得是`async`函数吗？当然不是。但是普通中间件的写法，第一可能和你想象中的不一样，第二还要略懂中间件实现的原理才能够正确实现普通中间件。如果坚信自己会用普通中间件，just do it!

## 参考资料
1. [Promise.resolve() - MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve)