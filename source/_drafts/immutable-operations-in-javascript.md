---
title: JavaScript不可变数组与对象的更新操作
categories:
  - Programming
tags:
  - JavaScript
  - Immutable Data
---

不可变对象是指对象在创建好之后，它的状态就不能够被改变。
一些 Web 开发框架的状态管理方案就基于不可变对象而来。例如
[React 的 Class-based Component 的状态管理](https://reactjs.org/docs/state-and-lifecycle.html#using-state-correctly)。
我们无法简单的通过改变`.state`的属性来触发试图的更新，
而必须通过`.setState`方法来改变`.state`。

为了满足这些框架的需求，我们需要将程序中的数据类型视为是不可变的。
每当数据发生更新，我们需要重新创建一个反映了更新的对象。
然而在 JavaScript 中，有两种类型内在是可变的——*数组*与*对象*，
如果要将它们当作不可变来使用的话，
我们必须从现有的工具中提取出一套专门来处理一些常见的更新操作。

## 不可变数组的更新方法

JavaScript 支持一些函数式的编程方法，在数组中引入了一些不改变原有数组，
而是返回一个新的数组的方法，这些方法包括`map`、`filter`、`concat`与`reduce`。
这些方法将是我们处理不可变数组中工具的一部分。

另外 ES6 引入的[展开语法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Spread_syntax)能够更加方便地从现有数组中构造新的数组。

### 更新一个不可变数组中的值

```javascript
const arr = [1, 2, 3]

//我们想要arr[1] = 2，在不可变数组中可以这样做
arr.map((elem, index) => {
  index === 1 ? 2 : elem
})
```

### 在数组删除中一个值

```javascript
const arr = [1, 2, 3]

// 我们想要删去arr的第2个元素
arr.filter((elem, index) => index !== 2)
```

### 在数组中增加一个值

```javascript
const arr = [1, 2, 3]

// ES6
[...arr, 1]
[1, ...arr]

// ES5
arr.concat([1])
[1].concat(arr)
```

## 不可变对象的更新方法

### 增加一个属性

```javascript
const source = {a: 1, b: 2}

// ES6
{...source, c: 3}

// ES5
Object.assign({}, source, {c : 3})
```

### 更改一个属性

```javascript
const source = {a: 1, b: 2}

// ES6
{...source, b: 3}

// ES5
Object.assign({}, source, {b : 3})

```

### 删除一个属性

```javascript
const source = { a: 1, b: 2 }

// ES6
const { b, ...deleted } = source

// ES5
Object.keys(source)
  .filter(key => key === 'b')
  .reduce((res, curKey) => (res[curKey] = source[curKey]), {})
```

## 总结

使用原生的 JS 特性可以模拟不可变数组与对象，但是实现看起来还是很繁琐，
而且可读性很差。

> 我终于理解了为什么有那么多 Immutable 库的存在了
