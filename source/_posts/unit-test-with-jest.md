---
title: 使用Jest进行单元测试
tags:
  - JavaScript
  - Jest
  - 单元测试
categories:
  - Programming
  - Front-end
  - JavaScript
date: 2018-12-09 23:11:30
---


[Jest](https://jestjs.io/)是Facebook开发的一款JavaScript测试框架。我之前做单元测试一直使用的是[Mocha](https://mochajs.org/)测试框架与[Chai](https://www.chaijs.com/)断言库。之前在好几个地方看到了Jest测试框架，心生好奇学习了一下。

<!--more-->

## Jest框架

Jest框架标榜"Delightful JavaScript Testing"，提供零配置、快速反馈与快照测试功能。

零配置方面，Jest本身就是测试框架、断言库、Mock框架与测试覆盖率检查工具的结合体。而Mocha仅仅是测试框架。要在Mocha里配置出与Jest相同的功能的话需要自行安装像Chai断言库、[Sinon.js](https://sinonjs.org/)框架与[istanbul](https://istanbul.js.org/)测试覆盖率检查。所以说Jest可以说是开箱即用的。Jest在零配置的情况下就可以自动检测以`*.spec.js`，`*.test.js`模式命名的文件以及`__tests__`文件夹下的文件，把它们作为测试用例。

快速反馈方面，Jest的监视模式可以只运行与**被改变的文件相关的测试用例**，不需要把所有的测试用例重跑一遍。开发者可以快速知道改变代码的结果。

快照测试方面，Jest可以将某次函数的输出保存为一个快照，之后测试就把实际输出与快照中的输出进行对比，保证函数行为的一致性。

Jest可以分为测试框架、断言以及Mock三个部分。

## 测试框架

关注Jest的测试框架部分，主要是关注在Jest中测试用例如何写、如何组织的问题。Jest中，测试用例的写法与Mocha非常相似。

```javascript
const binaryStringToNumber = binString => {
  if (!/^[01]+$/.test(binString)) {
    throw new CustomError('Not a binary number.');
  }

  return parseInt(binString, 2);
};

describe('binaryStringToNumber', () => {
  describe('given an invalid binary string', () => {
    test('composed of non-numbers throws CustomError', () => {
      expect(() => binaryStringToNumber('abc')).toThrowError(CustomError);
    });

    test('with extra whitespace throws CustomError', () => {
      expect(() => binaryStringToNumber('  100')).toThrowError(CustomError);
    });
  });

  describe('given a valid binary string', () => {
    test('returns the correct number', () => {
      expect(binaryStringToNumber('100')).toBe(4);
    });
  });
});
```

在Jest中，不必将`test`嵌入`describe`中。如果能使测试更简单的话，`test`也可以直接写在顶层。

### skip，only，each修饰符

`describe`与`test`可以连接`skip`，`only`，`each`修饰符。如`describe.skip('something', testFunction)`，会在测试时跳过这一个`describe`。`only`会使测试只运行指定的测试用例，这在某个测试用例出错Debug时非常好用。`each`修饰符可以执行多次参数不同的测试，它接受一个数组`table`和一个测试函数，`table`里的元素会作为参数传入测试函数。具体语法可以参见[文档](https://jestjs.io/docs/zh-Hans/api#describeeachtable-name-fn-timeout)。

### beforeAll，afterAll，beforeEach，afterAll钩子函数

Jest也支持在执行测试用例之前以及之后执行一些代码来做一些工作，像在测试前设置好测试数据、在测试后清理测试数据。这些工作可以作为`beforeAll`、`afterAll`、`beforeEach`、`afterAll`的回调函数。

## 断言

Jest支持`expect`式的断言，像`expect(1).toBe(1)`，其中`toBe`就是断言部分。Jest支持很丰富的断言。

### 相等断言

断言两个基本类型的值相等使用`expect(val1).toBe(val2)`。注意`toBe`断言使用`Object.is()`判断相等。它与`==`以及`===`都有不同。相对`===`，`Object.is()`在`-0`, `+0`与`NaN`的判断上有所不同。

如果要断言数组或者Object相等，使用`toEqual`断言。它会递归地判断每个属性/元素是否是相等的。

### 数字大小断言

大小关系断言有`toBeGreaterThan`、`toBeGreaterThanOrEqual`、`toBeLessThan`、`toBeLessThanOrEqual`。名字很直白，不解释。

对于浮点数，不能使用`toBe`或者`toEqual`进行相等断言。Jest提供了`toBeCloseTo`断言，可以在忽略一定误差的情况下，断言浮点数相等。

```javascript
test('float equality', function () {
  expect(0.2 + 0.1).toBeCloseTo(0.3) //pass
})

```

### 真值断言（Truthiness）

Jest提供`toBeTruthy`与`toBeFalsy`断言被测试函数的返回结果在`if`中是真还是假。像：

```javascript
test('nonempty string should be true', function () {
  expect('it is true').toBeTruthy() //pass
})
```

Jest也提供`toBeNull`、`toBeUndefined`、`toBeDefined`针对性的断言`null`与`undefined`的情况。

### 字符串相关

Jest提供`toMatch`断言被测试的字符串是否匹配给定正则表达式。

```javascript
test('but there is a "stop" in Christoph', () => {
  expect('Christoph').toMatch(/stop/) // pass
});
```

### 数组

要断言数组中包含某个子项可以使用`toContain`断言。

```javascript
const shoppingList = [
  'diapers',
  'kleenex',
  'trash bags',
  'paper towels',
  'beer',
];

test('购物清单（shopping list）里面有啤酒（beer）', () => {
  expect(shoppingList).toContain('beer');
});
```

### 抛出异常

要断言对函数的某些操作会抛出异常可以使用`toThrow`断言。

```javascript
test('throws on octopus', () => {
  expect(() => {
    drinkFlavor('octopus');
  }).toThrow();
});
```

### not修饰符

`not`修饰符可以把所有的断言反向，像`expect(1).not.toBe(2)`。

Jest提供的断言不止上面提到那么多。常用到的还有像断言长度的`toHaveLength`，断言对象有某个属性以及属性的值的`toHaveProperty`。更多断言的可以参见[Expect文档](https://jestjs.io/docs/zh-Hans/expect)。

## Mock

如果目前正在开发的模块存在依赖，比如某个函数需要一个随机产生的结果。假如依赖模块没有开发完成或者结果不可预知，我们是无法测试我们的模块的。为了解开开发模块与依赖模块在测试上的耦合，我们可以使用Mock功能去模拟被依赖模块的行为，比如规定某个函数调用时返回某些值。此时测试我们开发的模块就与被依赖模块的实际逻辑无关，实现了测试上的解耦。

这一部分比较复杂，我会在另外一篇文章来介绍这一部分。

## 踩坑部分

这里是我将一个用Vue编写的项目从Mocha转到Jest时踩的一些坑，算是笔记吧。

### Automatic mock不支持引用路径带有alias

`@`是Vue项目中`src`目录的`alias`（通过`jest.config.js`的`moduleNameMapper`指定）。如果尝试使用`alias`指定Automatic mock的模块：

```javascript
import Module from '@/src/libs/module'
jest.mock('@src/libs/module')
```

上述会报错。目前的解决方案是[jest issue #1290](https://github.com/facebook/jest/issues/1290)提到的用Manual mock。这个方式有用，但是过了那么多年可能有更好的做法。

### vscode-jest与Vue CLI 3脚手架生成的项目整合

`vscode-jest`是一款VS Code的插件，它可以在文件保存时使用Jest的监视模式进行单元测试。但是对于Vue CLI 3生成的项目，`vscode-js`原生的配置无法使用。为了使`vscode-jest`有用我们需要做一些额外的工作。下面假设已经通过`vue add @vue/unit-jest`安装了`@vue/cli-plugin-unit-jest`。

1. 在Workspace设置中将`jest.pathToJest`设置为`npm run test:unit`（使用npm）或者`yarn test:unit`（使用yarn）。
2. 在`jest.config.js`里添加两行
  ```javascript
  process.env.VUE_CLI_BABEL_TARGET_NODE = true
  process.env.VUE_CLI_BABEL_TRANSPILE_MODULES = true
  ```
3. 执行`npx jest --clearCache`

在[vue-cli issue #1879](https://github.com/vuejs/vue-cli/issues/1879)上有关于这个问题的讨论。如果没用可以尝试其他人提出的方法。