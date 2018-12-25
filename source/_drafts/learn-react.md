---
title: 从零开始的React笔记——基础篇
categories:
  - Programming
  - Front-end
  - React
tags:
  - React
---

2018 年，基于前端框架的开发已经成为了前端开发者的必备技能。
其中[React](https://reactjs.org/)、[Vue](https://vuejs.org/index.html)、
[Angular](https://angular.io/)成为了其中最为主流的三个框架。另外，
框架上的框架也如雨后春笋一般，比如基于 React 的跨平台原生开发框架
[React Native](https://facebook.github.io/react-native/)、
国内特色的跨各种小程序的开发框架[Taro](https://taro.js.org/)，
基于 Vue 的跨小程序开发框架[Mpx](https://didi.github.io/mpx/)等等。
这些框架可以让前端开发者不仅能够在浏览器端大展身手，
而且能够在客户端、各种封闭的小程序端有所作为。最近由于工作的需要，我开始学习 React。
在这之前我只有 Vue 的背景。让我们从现在开始，从零开始学习 React 吧。

<!--more-->

## Hello world

在学习一门新的语言的时候，打印 Hello world 就像是一个仪式一样。
现在让我们在 React 里完成这一个仪式吧~

```html
<html>
  <body>
    <div id="app"></div>
  </body>
  <script
    src="https://unpkg.com/react@16/umd/react.development.js"
    crossorigin
  ></script>
  <script
    src="https://unpkg.com/react-dom@16/umd/react-dom.development.js"
    crossorigin
  ></script>
  <script src="hello-world.js"></script>
</html>
```

```javascript
// hello-world.js
ReactDOM.render('hello world', document.querySelector('app'))
```

复制、粘贴、打开一把梭之后，屏幕上会显示`hello world`。这个仪式我们就算完成了。
在我们离开之前，注意到，明明我们要学的是 React，但是我们在 JavaScript 脚本里调用了
`ReactDOM`这是怎么回事呢？在早期，React 还是纯浏览器端的框架的时候是不分`React`与
`ReactDOM`的。后来，在 V0.14 版本中，React 被分为了两个独立的包`React`与`ReactDOM`。
在 Change log 中，他们提到

> As we look at packages like react-native, react-art, react-canvas, and
> react-three, it has become clear that the beauty and essence of React has
> nothing to do with browsers or the DOM.
> To make this more clear and to make it easier to build more environments
> that React can render to, we’re splitting the main react package into
> two: react and react-dom. This paves the way to writing components that
> can be shared between the web version of React and React Native. ...
>
> -- [react v0.14](https://reactjs.org/blog/2015/10/07/react-v0.14.html#changelog)

我们可以从中看出`React`实际上是更高一层的抽象。我们可以用`React`编写我们的程序，
然后用`ReactDOM`将其“编译”到浏览器端或者用`React Native`“编译”到移动客户端。因此，
学完`React`之后就可以同时搞三端的开发（Web、iOS、Android），想想是不是有一些兴奋呢？
（当然，不同端还有一些差异，需要进一步学习）

## 元素

前面提到，React 是更高一层的抽象，是平台无关的。
在 HTML 中，我们写很多标签（tag），它们组成了 HTML 文档。
在 React 中，对应标签的概念是什么呢？

答案是[_元素_](https://reactjs.org/docs/rendering-elements.html)。
元素是 React App 的最小的构件块。
如果熟悉 Vue 的话，React 中的元素类似于 Vue 中的 VDOM Node。
在浏览器中，由 React DOM 负责将 React 元素渲染成真正的 DOM。

### 如何创建一个元素

如果使用原生 JavaScript 语法的话，可以使用`React.createElement`函数：

```javascript
const helloWorld = React.createElement('h1', {}, 'hello world')
```

这样我们就编写了一个`helloWorld`元素。它经过 React DOM 渲染会变成

```html
<h1>hello world</h1>
```

请戳这里看[在线 demo](https://codepen.io/ayase-252/pen/QzpXRz)

当然，这种写法不仅不方便。而且元素一复杂就很难写。因此，
React 提供了一种叫[JSX](https://reactjs.org/docs/introducing-jsx.html)的语法。
通过 JSX 语法，我们的元素可以改写成

```jsx
const helloWorld = <h1>hello world</h1>
```

更进一步，我们在元素中用到原生 JavaScript 表达式，只需要将表达式用`{}`括起来：

```jsx
const weather = 'sunny'
const weatherReport = <p>it's {weather} today</p>
```

上面`weatherReport`会被渲染成：

```html
<p>it's sunny today</p>
```

我们还可以在标签内指定嵌套的标签：

```jsx
const nestedElem = (
  <h1>
    <span>hello world</span>
    <span>published in 22th Dec, 2018</span>
  </h1>
)
```

由于元素跨行，用圆括号`()`包裹是必要的。
[在线 demo](https://codepen.io/ayase-252/pen/KbWOLV)。

我们也可以在标签属性中使用 JavaScript 表达式，这里就不展开说了。
JSX 语法的灵活性可以让 JavaScript 的强大功能与标签结合起来。
如果对 JSX 有进一步的兴趣，
我推荐看一下[官方教程](https://reactjs.org/docs/introducing-jsx.html)。

有同学可能会好奇 JSX 不符合 JavaScript 的语法啊，它是怎么作用的呢？
JSX 会通过 Babel 的一个插件
[babel-plugin-transform-react-jsx](https://github.com/babel/babel/tree/master/packages/babel-plugin-transform-react-jsx)
将 JSX 内的标签变换为用`React.createElement`创建的效果等同的元素。

## 初始化项目

React 也有非常好用的 CLI 脚手架工具。现在让我们创建一个新的 Web 项目吧。

```bash
npx create-react-app learn-react
```

嗯，用到了 npm，会有很多包需要下载，请耐心等待~。

生成完毕之后，项目的目录结构是这样。

```plain text
├── package.json
├── public
│   ├── favicon.ico
│   ├── index.html
│   └── manifest.json
├── README.md
├── src
│   ├── App.css
│   ├── App.js      // App组件
│   ├── App.test.js
│   ├── index.css
│   ├── index.js    // 入口文件
│   ├── logo.svg
│   └── serviceWorker.js
└── yarn.lock
```

目前，我们主要关注的是入口文件`index.js`、以及组件`App.js`。
其中`index.js`、`App.js`已经使用到了 JSX 语法，
这意味着我们也可以在之后的学习中使用到 JSX 语法。

在终端中执行下面的命令开启开发服务器：

```bash
npm run start
```

如果使用`yarn`的话，执行：

```bash
yarn start
```

浏览器会自动打开并且跳转到`localhost:3000`，热重载功能已经启动，
编辑`App.js`之后保存可以立即在浏览器中看到效果。

## 总结

本文章中，我们通过`hello world`程序开启了我们的 React 之旅。React 是一个高级的抽象，
通过 React DOM、React Native 等可以将我们的 React 程序转译到对应平台。
React 中最基本的单位是元素，我们可以通过`React.createElement`创建元素。
为了方便 React 提供了 JSX 语法，通过 Babel 的插件，
Babel 会将 JSX 语法写成的标签转换为用`React.createElement`创建的元素。
我们使用`create-react-app`脚手架创建了一个新的 React Web 项目。

## 下一步

下面一篇文章我们会简述现代前端开发中最重要的一个问题——组件化在 React 中如何实现。
