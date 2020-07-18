---
title: 程序设计：抽象的艺术
categories: Thinking
tags:
  - Programming
---

## 从买可乐说起

程序设计实际上是一种“抽象”的艺术。
这里的抽象指的是从一些低级的操作中抽象出一些高级的操作。
比如，我们知道怎么做加法，像`add(1, 1) = 2`，
在后来我们发现，我们常常累加一些重复的数字，
例如，我们去超市买了 2 瓶可乐，我们想知道 2 瓶可乐多少钱。
假如可乐的价格是 3.5 元，我们可以很容易用加法算出来结果`add(3.5, 3.5) = 7`。
下一次，又多了一个朋友一起去超市，我们买了 3 瓶可乐，
同样我们可以用加法算出 3 瓶可乐的价格`add(add(3.5, 3.5), 3.5) = 10.5`。
假如我们买 N 瓶可乐，我们可以这样算出结果。

```typescript
let totalPriceOfCola = 0
for (let i = 0; i < N; i++) {
  totalPriceOfCola = add(totalPriceOfCola, 3.5)
}
```

更好一点，我们可以给这个过程一个名字，像`computeTotalPriceOfCola`：

```typescript
function computeTotalPriceOfCola(numberOfCola: number) {
  let totalPriceOfCola = 0
  for (let i = 0; i < N; i++) {
    totalPriceOfCola = add(totalPriceOfCola, 3.5)
  }
  return totalPriceOfCola
}
```

似乎，我们的能力增加了一点，
我们只要使用一条指令`computerTotalPriceOfCola(N)`就能够知道`N`瓶可乐的价格。
这样我们在计算可乐总价格的时候，就不需要去想它的实现细节。一条指令就能够解决问题。
我们对掌握了计算可乐总价格的方法很满意。
好景不长，我们连续喝了几天可乐，喝腻了想换一种口味。
这一次去超市的时候，我们买了 2 瓶咖啡。咖啡一瓶 5 元。
我们还不会如何计算多瓶咖啡的总价，但是我们会加法，我们能够解出这个问题。
2 瓶咖啡的总价是`add(5, 5) = 10`元。3 瓶咖啡的总价是`add(add(5, 5), 5) = 15`元。
那么，我们同样可以定义出一个计算`N`瓶咖啡总价的方法。

```typescript
function computeTotalPriceOfCoffee(numberOfCoffee) {
  let totalPriceOfCoffee = 0
  for (let i = 0; i < N; i++) {
    totalPriceOfCoffee = add(totalPriceOfCoffee, 5)
  }
  return totalPriceOfCoffee
```

接下来，我们要计算咖啡的总价时，就可以同样使用一条指令计算出来了
`computeTotalPriceOfCoffee(N)`。那么下次，我们要买茶，假设茶的价格是 3 块。
我们可以同样的定义出一个方法，现在我们的知识库里有：

```typescript
function computeTotalPriceOfCola(numberOfCola) {
  let totalPriceOfCola = 0
  for (let i = 0; i < N; i++) {
    totalPriceOfCola = add(totalPriceOfCola, 3.5)
  }
  return totalPriceOfCola
}

function computeTotalPriceOfCoffee(numberOfCoffee) {
  let totalPriceOfCoffee = 0
  for (let i = 0; i < N; i++) {
    totalPriceOfCoffee = add(totalPriceOfCoffee, 5)
  }
  return totalPriceOfCoffee

function computeTotalPriceOfTea(numberOfTea) {
  let totalPriceOfTea= 0
  for (let i = 0; i < N; i++) {
    totalPriceOfTea= add(totalPriceOfTea, 3)
  }
  return totalPriceOfTea
```

观察上面的方法，我们可以看到三种方法基本上是相同的，区别只有商品的单价。
如果我们把商品的单价也参数化，那么我们不就够计算出 N 件相同任意商品的单价了吗？
Let's do it.

```typescript
function computeTotalPriceOfGood(number, price) {
  let totalPrice = 0
  for (let i = 0; i < N; i++) {
    totalPrice = add(totalPrice, price)
  }
  return totalPrice
}
```

现在，我们不仅能够计算可乐、咖啡、茶的总价格，我们能够计算电影票、耳机、自行车
等等的总价格。似乎我们离掌控这个世界又近了一步。等等，我们去买东西，
不一定只买一种东西啊。如果我们想买 2 瓶可乐与 3 瓶茶呢？emmmmm.
我们似乎一步无法解决，但是我们会加法。2 瓶可乐与 4 瓶茶的价格是
`add(computePriceOfGood(2, 3.5), computePriceOfGood(4, 3)) = 19`元。
现在，假如我们有一张购买清单，上面列明了购买商品的单价与件数，
那么我们就可以定义一个新的过程来计算这张购买清单上商品的总价格。

```typescript
type Goods = {
  number: number
  price: number
}
function computeTotalPriceOfGoodsOnList(goods: Goods) {
  let totalPrice = 0
  for (let good of goods) {
    totalPrice = add(
      totalPrice,
      computeTotalPriceOfGood(good.number, good.price)
    )
  }
  return totalPrice
}
```

现在我们可以横行超市随意购买东西了，不用担心计算商品总价格的问题。
不仅是超市，我们也可以去电影院、火车站以及任意需要结账支付的地方使用这个方法。
我们只要在购买的时候记下来买下东西的单价与件数。在这个例子中，
我们从只会加法到会计算购买商品的总价格，是一个很大的进步吧。

## 程序背后的魔法

我们可以使用一条命令`ssh user@hostname`就能够访问远在异国的服务器，
在上面进行任何操作。我们的身体不需要亲自前往服务器所在的国家。
我们可以在上面设置一个 Web 服务。然后去注册一个域名，DNS 上指向这个地址。
弄好之后，用户就可以通过这个域名，在浏览器上使用我们的 Web 服务，
也就是网站。对于普通人而言，这简直与魔法一样。但是如果我们来结构这个魔法，
我们可以发现，它只有几个要素：

1. 会用`ssh`登录服务器进行一些操作，这可能需要会一些 Linux 的使用方法；
2. 会写一个 Web 服务器，不是说会写一个 Web 服务器的*框架*，
   会利用成熟的框架如`Flask`来写服务就行，也许会需要一点关于 HTTP 的知识。
3. 会注册一个域名，然后设置它的 DNS 记录。这个...有钱就行了。

如果理解上面的抽象，网站的工作流程并不是一个魔法。然而，
我们确实在这个例子中用到了比较高级的抽象。像 SSH 协议、Web 服务器框架等等。
也许其中的每一个软件都是用成千上万行的代码堆出来的。
为了使用它们，我们并不需要去理解其中的细节，我们只要去理解它们提供的抽象。
然后利用它们的抽象，去制作更加高层的产品。

### 最高级的抽象决定了能力范围

程序使用的最高级的抽象决定了程序的能力范围。从网站的例子来说，
我们使用了一些框架，就能够开设起一个网站。但是，
仅仅基于 Web 框架提供的功能，也许我们只能够开发出一个显示 hello world 网站，

```tsx
function hello_world() {
  return <span>hello world</span>
}
```

假如我们会如何去调用一个支付接口，
这样，我们网站就可以提供支付服务。假如，
我们会如何使用数据库去管理某件商品的库存，例如图书。这样结合起来，
我们的网站就可以开始销售书籍了（前提是有书卖的情况下）。
也许我们的网站还应该记录用户的订单历史，我们也可以用数据库去做。
这样，就有了一个成型的图书销售网站。Perfect，我们也许利用这个网站盈利了呢。

可以看到，我们利用低级的抽象去构建越来越高级的抽象。从简单的 hello world 网站
到能够提供支付功能的网站到图书销售网站，最后我们跳出程序本身，
我们发现我们的能力范围已经扩张到可以盈利了（相当于调用方法`make_money()`）。
不是很棒吗？

但是，要实现越高级的抽象，也要求掌握越高级的低层抽象（注意不是底层抽象）。
人类是不擅长思考跨级别的抽象，一般要实现越高级的抽象，人类只会想到比它低一层的抽象。
比如，在做一道求微分的题目时，我们一般不会从微分的定义去推导（底层抽象），
而是会用一些法则与规律去推导（低层抽象）。

### 最低级的抽象决定了完美程度
