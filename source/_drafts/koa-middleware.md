---
title: Koa的中间件洋葱模型
categories: Technology
tags:
    - Koa
    - Middleware Pattern
---

之前在改造博客项目后端的时候使用到了[Koa](https://koajs.com/)这一款web框架。它的设计极为简单，基本上只剩下使用中间件模式处理请求的逻辑。Koa的中间件模式与一些线性的中间件模式有一些不同。它的执行流类似洋葱，从第一个中间件进入，依次执行到最后一个中间件，然后从最后一个中间件再返回到第一个中间件。这样，每一个中间件会有两次处理`ctx`的机会。那么，Koa是如何实现这一模式的呢？

<!--more-->

## 中间件洋葱模型

![中间件洋葱模型](MiddlewareOnion.png)
<p style="text-align:center"><p>