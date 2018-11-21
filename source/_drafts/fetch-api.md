---
title: Fetch API
categories: Note
tags:
    - Fetch
---

Fetch API提供了一个用于异步获取资源的JavaScript接口。Fetch API提供的接口比起`XMLHttpRequest`对象对于开发者更加友好。

<!--more-->
## 简单使用Fetch

要发起一个异步请求，只用简单的调用`fetch(resource_url)`即可。`fetch`函数返回一个`Promise`，因此在ES7下可以配合`async/await`关键字使用。下面是使用`fetch`请求一个JSON数据的demo。

```javascript
fetch('https://api.myjson.com/bins/sv0te')
    .then((res) => {
        return res.json()
    })
    .then((res) => {
        console.log(res.message) // hello world
    })
```

在线[demo](https://codepen.io/ayase-252/pen/yQXxjK?editors=1012#)。

### 配置Fetch

`fetch(input, init)`函数接受两个参数，其中`options`是可选参数。通过给`init`传入配置对象可以改变`fetch`的默认行为。比如使用`POST`方法向`input`请求数据。

配置对象的属性以及可选值如下:

| 属性 | 意义 | 可选值 |
|---|--|--|
|`method`| 请求使用的方法|`GET`,`POST`,`PUT`等|
|`headers`|请求的头信息 | 形式为`Headers`的对象或包含`ByteString`值的对象字面值|
|`body`| 请求的body信息 | 可能是一个`Blob`，`BufferSource`，`FormData`,`URLSearchParams`或者`USVString`|
|`mode`| 请求的模式 | `cors`, `no-cors`或者`same-origin` |
|`credentials`| 请求是否带有机密信息（如Cookies） | `omit`, `same-origin`, `include`|
|`cache`| 请求的cache头 | `default`, `no-store`, `reload`, `no-cache`, `force-cache`或者`only-if-cached`|
|`redirect`| 可用的redirect模式| `follow`（自动重定向），`error`（重定向抛出错误），`manual`（手动处理重定向）|
|`referrer`| 一个`USVString`，可以是`no-referrer`，`client`或者一个URL，默认`client`|
|`referrerPolicy`| 处理`referrer`头的策略| `no-referrer`, `no-referrer-downgrade`, `origin`, `origin-when-cross-origin`, `unsafe-url`|
|`intergrity`| 包含请求的子资源的完整性值

## 参考资料
1. [WorkerOrGlobalScope.fetch()](https://developer.mozilla.org/zh-CN/docs/Web/API/WindowOrWorkerGlobalScope/fetch)


