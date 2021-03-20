---
title: Node.js 调试手记：使用 VS Code 调试 Node.js JavaScript 侧代码
tags:
  - Node.js
categories: Technology
date: 2021-03-20 22:36:26
---


在{% post_link annual-reflection-2020 去年的年终总结 %}中，
我给自己立下了一个 flag——「参与开源项目，向知名开源项目提交至少一个功能性或者修复性 PR」。
我选择的是大名鼎鼎的[Node.js](https://github.com/nodejs/node)。应该符合
知名开源项目。事不宜迟，赶快行动。

> tl;dr; 本文介绍的是调试 Node.js **核心**中 JS 侧的代码方法。
  请注意，并**不**是 Node.js 应用的调试哦。

<!--more-->

## Node.js 的结构

Node.js 由两部分代码组成，一部分代码是 C++ 代码，位于项目的 `src` 目录中，
另外一部分是 JavaScript 代码，位于项目的 `lib` 目录中。

![Node 项目架构](./node-js.png)

C++ 部分负责把用 C 或者 C++ 编写的一些提供底层能力的库粘在一起，
如异步 I/O 库 libuv（C）、JS引擎 V8（C++）、http解析器 llhttp (TS 编译到 C)等。
C++ 部分也提供了程序的入口点，如 `node xxx.js` 实际上最先到达的就是 C++ 部分的
[main 函数](https://github.com/nodejs/node/blob/e427c487fe3f7da465c372ade3d65bd55b057e30/src/node_main.cc#L105)
另外，C++ 部分也提供了一些 Node.js 内部才能够使用到的模块，
这些模块可以被 JavaScript 部分调用来实现逻辑。

JavaScript 部分提供了用户侧可使用的API。
这些 API 在文档中分为了[若干模块](https://nodejs.org/dist/latest-v15.x/docs/api/async_hooks.html)，
基本上每一个模块对应 `lib` 中的一个 JS 文件。根据模块的名字能容易找到对应的文件，
如 `Events` 模块，对应的就是 `lib/events.js`。

## 提升效率——外置 JS

Node.js 严格上算是一个 C++ 程序。对程序的修改要走编译流程生效。
对于习惯于热重载等技术带来的即时反应的工程师来说，这套流程是非常慢。
好在，如果只修改 JavaScript 侧的代码的话，
我们可以运行下列命令：

```bash
./configure --node-builtin-modules-path $(pwd)
```

此时，如果运行 `make -j4` 会构建在 `out/Release` 下出来一个特殊的 `node` 二进制。
在这个二进制中，JS 层的文件不会被编译进去，
而是会使用 `--node-builtin-modules-path` 指定的外置 JS
path 下的 `lib` 中的 JS 文件运行时解释。在这个例子中，就是 Node.js 的项目文件夹。
对 JS 文件的修改会即时地反应到这个特殊二进制当中。

例如，为了测试，我们在 `EventEmitter` 的 `init` 方法中抛出一个异常:

```js
// lib/events.js
EventEmitter.init = function(opts) {
  throw new Error('error');
  if (this._events === undefined ||
      this._events === ObjectGetPrototypeOf(this)._events) {
    this._events = ObjectCreate(null);
    this._eventsCount = 0;
  }
  //..
}
```

保存后，执行 `./out/Release/node`，就可以看到 node 确实抛出了异常

```bash
node:events:186
  throw new Error('error');
  ^

Error: error
    at process.EventEmitter.init (node:events:186:9)
    at process.EventEmitter (node:events:79:3)
    at setupProcessObject (node:internal/bootstrap/node:377:3)
    at node:internal/bootstrap/node:55:1
```

## DEBUG JS

使用外置 JS 的可以很大的提升开发效率。但是，
调试时如果我想看到一些属性或者变量又怎么办呢？`console.log` 大法吗？

很不幸的是，`console.log` 在一部分模块里是行不通的。因为 `console`
本身也是 Node.js 公共模块的一部分。使用 `Stream` 模块实现。如果在调试
`Stream` 模块相关的模块时，就容易出现爆栈无法打印的情况。emmmm🤔，
难道我们就束手无策了吗？其实，是有解决方案的——调试器。

调试器就是那个可以单步调试，在每一步都能够打出来当前环境的中的变量的情况的程序。
也许 `console` 过于方便已经让我们忘记了有这把瑞士军刀了。对于 Node.js 应用，
VS Code 的调试器原生支持。但是对于 Node.js 项目本身呢？
我们有办法在模块源码中打断点吗？这也是有的。

mmomtchev 向 VS Code JS 调试器提交了 debug Node.js 内部代码的功能[2]。
结合上一节的外置 JS，我们可以方便地在代码中打断点，然后更改调试了。
在 `.vscode/launch.json` 中添加下面的调试配置：

```json
{
  "version": "0.2.0",
  "configurations": [
    ...,
    {
      "name": "Launch current file",
      "runtimeExecutable": "${workspaceFolder}/out/Release/node",
      "args": [
        "--expose-internals",
        "--nolazy",
        "${workspaceFolder}/${relativeFile}",
      ],
      "request": "launch",
      "type": "pwa-node"
    }
  ]
}
```

然后把需要调试的**外部** JS 文件，复制到 Node.js 项目中。打开 VS Code 调试器，
选择 `Launch current file` 启动，就可以愉快调试了。

## 参考资料

[1] Node.js [BUILDING](https://github.com/nodejs/node/blob/master/BUILDING.md#speeding-up-frequent-rebuilds-when-developing)
[2] [Add support for debugging Node internals when they are externally loaded](https://github.com/microsoft/vscode-js-debug/issues/823)
