---
title: 9102年的HTTP/2简介
categories:
  - Technology
tags:
  - Computer Network
  - HTTP
---

都9102年了，怎么还有人提HTTP/2的，HTTP/3都出来了（画外音）。

<!-- more -->

HTTP/2的主要目的就是提升网页的页面加载速度，换句话说就是“快”。
HTTP/2基于Google开发的[SPDY协议](https://zh.wikipedia.org/wiki/SPDY)。
在本文中，我们会重点关注一下HTTP/2是如何在提高网络传输的速度。

## HTTP/1.x的局限性

HTTP是现代互联网的基石之一。但是，
HTTP过于简单的“客户端——服务端”运作模型给目前追求载入速度的互联网应用带来了一些阻碍。
如在HTTP/1.x的模型中，每一个TCP连接只能够同时存在一个请求，
如果要同时载入不同不相关的资源，比如图片的话，就必须要同时开启多个TCP连接。
再比如HTTP标头可能携带很多内容，像`cookies`等，
相较于真正请求或者回应的内容而言，标头的比重可能很高。
换句话说协议的成本很高。当然，HTTP/1.x的局限性不仅限于之前提到这些。

## HTTP/2的主要feature

## 二进制分帧层

## 数据流、消息与帧

## 请求和响应复用

### 服务器端推送

## 标头压缩（HPACK）



为了从协议层面解决这个问题，HTTP/2引入了一个二进制分帧层（Binary Framing Layer）。
