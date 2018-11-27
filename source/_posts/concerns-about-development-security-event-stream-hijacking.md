---
title: 重新审视Web开发安全：event-stream注入恶意代码
tags:
  - Web Development Security
  - Cyber Security
  - JavaScript
categories: Technology
date: 2018-11-27 22:19:51
---

在开发Web应用时，如果我们的需求缺少一个不那么简单而又不想亲自动手写的工具时，我们会向Google老师求助，看到一个`npm`包恰好能够满足，然后`npm install package`一把梭，`require`解决问题。完美的工作流，体现了JavaScript社区的开放与强大。但是，最近一个月下载量达百万的包`event-stream`被注入了恶意代码。恶意代码会尝试劫持另外一个包中的`bitcore-wallet-client.getKeyFunc`方法。如果一个项目同时依赖了`event-stream`与`copay-dash`，当`bitcore-wallent-client`中的`getKeyFunc`运行时，恶意代码会检查钱包的id，持有比特币BTC的数量以及比特币现金BCH的数量。如果BTC数量大于100或者BCH数量大于1000，就将钱包的公钥发给一个地址。

当我看到这个新闻的时候，我心里想，我没写过写区块链相关的对象，应该不会被影响到吧。但是当我没事跑一下检查方法时，结果却傻眼了：

![检查结果](vulnerability-check.png)

我中招了！

<!--more-->

## 安全建议

我中招是因为`vue-cli`间接依赖了受影响`event-stream@3.3.6`。目前`vue-cli`已经更新了版本将`event-stream`锁定在了未受影响的`3.3.4`。如果有向我一样中招的朋友，请尽快升级`vue-cli`。

```sh
# 全局安装时
npm update vue-cli -g
```

如果没有安装过`vue-cli`，也要检查一下是否有项目简介依赖了`event-stream@3.3.6`。方法如下：

```sh
# 全局层面
npm ls event-stream flatmap-stream -g

# 项目层面
npm ls event-stream flatmap-stream
```

## 攻击原理

太简单了，假如我们有一个正常的包`good.js`是这么写的：

```javascript
module.exports.goodFunc = () => 'good'
```

然后我们有一个包含恶意的包`malicious.js`：

```javascript
const good = require('./good')

good.goodFunc = () => 'hijacked'
module.exports.usefulFunc = () => 'a useful function in malicious code'
```

当我们的项目同时依赖两个包的时候：

```javascript
// index.js
const good = require('./good')
const malicious = require('./malicious')

console.log(good.goodFunc())  // hijacked
```

简单到不可思议吧，比起别的程序深入汇编挖漏洞。JavaScript只需要，从一个不知道干什么的包里`require`被攻击的包，然后修改一下就行。这里`event-stream`干了很容易被发现的通过网络发送数据，如果其他攻击不是那么容易被发现呢？我们项目里动辄上千的依赖，我们能够保证它们都是善良的天使吗？I don't know....

## 重新审视开发安全

Web开发者整天与不安全的网络、不可信任的输入等打交道。Web安全对于开发者而言，似乎就是保证用户的信息安全。我们会使用很多技术来保证用户的安全，像HTTPS，CSRF防御等等。

但是...对于开发者本身呢？Node.js的模块机制提供的保护是0。它只能够保证第一次`require`的时候代码是包作者提供的。接下来轮到我们的代码`require`的时候，说实话，我们不能保证我们调用的函数是李逵还是“李鬼”。JavaScript语言层面提供的保护...近乎为0。能够阻止修改我们`exports`出去的对象的方法只有一个，`Object.defineProperty(module.exports, 'myMethod', { writable: false })`。那么长，还只能锁住一个方法，一般库的作者是不会写的。

现在我们使用这些包，就像使用C++的未定义行为一样。有时它确实有用，但是可能不是像你想象中的一样作用的。也许哪一天，一个包给`Object.create`注入新行为，当执行这个函数时删除项目文件夹，也说不定。。

目前JavaScript社区是靠人来解决这一事情的，默认信任大牛、高Star的包。但是这些“高信任度”的包引入的依赖呢？巨量的基础包依赖，像`is-odd`（判断一个数是不是奇数。很简单是吧？这个包每周下载量达1百万次哦）这些，使得我们根本不可能去检查项目中的每一个依赖。当其中一个依赖出现问题的时候，我们每个人都是潜在的受害者。

综上所述，`Node.js`不严格的模块机制、过于灵活的JavaScript以及项目中巨量的包依赖已经对开发安全形成了威胁。我们需要一个解决方案**保证**我们调用的函数就是作者写的函数。

## 一个可能的解决方案

既然现有机制无法保证包在`export`出去之后其**公共接口**不受改变。那么包的开发者应该承担起这一责任，无论是通过`Object.defineProperty`去设定公共接口不可写或什么其他的方法。这里我写了一个[exports-lock](https://github.com/Ayase-252/exports-lock)，能够通过递归地设置`module.exports`的属性为不可写(`writable: false`)、不可配置(`configurable: false`)。保证公共接口在`export`之后不受篡改。

（可能暴力了点，可以改良。。。）

## 参考文章

1.[I don't know what to say.](https://github.com/dominictarr/event-stream/issues/116)
