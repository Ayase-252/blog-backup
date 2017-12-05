---
title: ES6在Babel转译下的循环import问题
date: 2016-10-02 19:34:31
categories: "Techno"
tags:
  - ES6
  - Babel
  - Node.js
---

为了解决ES5缺乏模块化的问题，ES6正式引进了Module的概念。Module允许开发者们将一个很大的
系统拆分为若干个小的Module。模块化程序设计的概念存在于现今流行的大多数编程语言中。

但是，由于ES6刚刚发布，支持ES6的环境非常少。如果要使用ES6的话，一般需要通过`babel`等转译器
将使用ES6的js文件转译为ES5的js文件。

<!-- more -->

关于Module，ES6定义了`import`和`export`两个命令，顾名思义，`import`用于从其他模块中导入东西，
`export`用于从本模块中导出东西到其他模块。

## 问题描述

最近在编写blog后台的时候，遇见了一个奇怪的bug。代码经过`babel`转译后在`Nodejs`环境下运行，
代码示意如下：

    //a.js
    import b from './b'

    const foo = () => {
      b.bar()
    }

    foo()

    export default foo

    //b.js
    import foo from './a'

    const b = {
      bar: () => {
        foo()
      }
    }

    export default b

上述代码本身是存在问题的。`a.js`中的`foo()`调用了`b.js`中的`b.bar()`，而`b.js`中
的`b.bar()`，又反过来调用`a.js`中的`foo()`。然而，Nodejs给出的错误却难以一眼看出问题在哪。

    G:\learn\something\b.js:15
        (0, _a2.default)();
                        ^

    TypeError: (0 , _a2.default) is not a function
        at Object.bar (G:\learn\something\b.js:15:21)
        at foo (G:\learn\something\a.js:14:15)
        at Object.<anonymous> (G:\learn\something\a.js:17:1)
        at Module._compile (module.js:541:32)
        at Object.Module._extensions..js (module.js:550:10)
        at Module.load (module.js:458:32)
        at tryModuleLoad (module.js:417:12)
        at Function.Module._load (module.js:409:3)
        at Function.Module.runMain (module.js:575:10)
        at startup (node.js:160:18)

## 问题分析

从上面的stack trace看,当NodeJS尝试去Load某个模组时，问题出现在了`(0, _a2.default)()`这一句。前者这个表达式不是一个函数。因为代码被转译过，没有办法从原来的代码进行分析，这里先使用相同的转译设定，在REPL上将原代码转译出来。如下：

    //a.js
    'use strict';

    Object.defineProperty(exports, "__esModule", {
      value: true
    });

    var _b = require('./b');

    var _b2 = _interopRequireDefault(_b);

    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }

    var foo = function foo() {
      _b2.default.bar();
    };

    foo();

    exports.default = foo;


    //b.js
    'use strict';

    Object.defineProperty(exports, "__esModule", {
      value: true
    });

    var _a = require('./a');

    var _a2 = _interopRequireDefault(_a);

    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }

    var b = {
      bar: function bar() {
        (0, _a2.default)();
      }
    };

    exports.default = b;

可见，Babel对于`import`和`export`的处理方式，还是转化为`require`和`export.exportedObj`的形式。通过查阅[文档](https://nodejs.org/api/modules.html)的Cycle一节，NodeJS为了在出现circular require的情况下，不使系统陷入无限循环，其内层的`require('./a')`会返回`a`的一个*不完全的copy*。在这个copy中，`foo`还不存在。因此`_a2.default`在这种情况下还是`undefined`，所以出现了`(0, _a2.default)`不是函数的错误。

## 解决方法
解决方法就是想办法将循环的import断开。一个正常的程序是不应该出现circular igmport的情况的。出现这种情况的原因，大部分可能是因为重构时手滑，多添了几个不必要的import。

## 参考资料

  *[Node.js v6.7.0 Documentation - Modules](https://nodejs.org/api/modules.html)
