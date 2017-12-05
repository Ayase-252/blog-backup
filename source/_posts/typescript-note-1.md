---
title: TypeScript初探（I）
tags:
  - TypeScript
date: 2017-01-29 18:42:25
categories: "Techno"
---


TypeScript是微软开发的一种开源的编程语言。它脱胎于JavaScript，为了解决JavaScript用于大型工程时出现的问题。最近几年，由于移动互联网的爆发，web开发越来越复杂，新技术层出不穷。但是JavaScript却迟迟没有进步，其自身的语言短板使得开发web app变成一件非常有挑战性的事情。TypeScript于2012年发布，正如其官方网站所言，目标是成为“JavaScript that scales”。TypeScript推出了类型系统、类等一些特性，其中一些特性已经被引入到下一代JavaScript标准ES6。TypeScript也是知名web应用框架AngularJs 2的推荐编程语言。
<!--more-->

## 安装
TypeScript可以从`npm`中安装。下面的命令将会在全局范围内安装`typescript`。

    npm install -g typescript

## 编译TypeScript
网页中浏览器实际运行的是JavaScript代码，我们的TypeScript代码需要通过编译器编译到对应的JavaScript代码。假设我们要编译`a.ts`文件，下面的命令将会将`a.ts`编译为对应的JavaScript版本`a.js`。

    tsc a.ts

TypeScript编译的js文件兼容ES3标准。

## 类型注记(Type Annotation)——let hello:string = "world"
类型系统是TypeScript的杀手级特性。我们可以给各种变量，函数的返回值打上类型注记，限定这些变量或返回值的类型。编译器会在编译时对这些变量的类型进行检查。如果实际传入的参数类型不符合注记，编译器会产生错误警告。值得注意的是，尽管有错误警告，但是编译器仍然会将ts文件编译成js文件。只是程序的行为可能不会和想要的一样。

## 接口（Interface）——interface Yakusoku{youShouldHaveThis:any}
接口是代码与用户代码的约定，即用户代码应该拥有或者实现接口所要求的功能。在TypeScript中，接口作为约定，在编译时，编译器会对接口进行复杂的类型检查，例如检查传入的参数是否拥有接口所指示的属性；如果以object literal作为参数直接传入，编译器会检查是否有过多的属性在参数中。接口的使用可以为函数的正确调用提供一定的保证。至少，传入参数的属性名的正确性是可以保证的。

## 类（class）——class OOP {...}
ES6标准正式引入了类的概念。在一些传统的编程语言中如`C++`，OOP往往是基于类的（class-based）。而在JavaScript中，OOP是基于原型的（prototype-based）。两种方法那种更优尚存争议，但是基于类的OOP对于一些来自于`C++`、`Java`等编程语言的程序员来说更为友好。而且基于类的OOP也有许多的工程实践。为了兼容ES6的标准，TypeScript也引入了类。
