---
title: CSS定位元素
categories: "Technology"
tags:
    - CSS
    - Web Development
---

CSS一个很重要的功能就是确定元素的位置，让元素出现在它应当出现的地方。本文将会梳理浏览器是如何定位元素以及我们可以怎么利用这些机制定位我们的元素。

<!--more-->

在CSS 3 Working Draft中，提到了一个元素可以由3种方式进行定位：
1. 常规流（Normal Flow）: 通过块级元素的块级格式化（Block Formatting）与行内元素的行内格式化（Inline Formatting）以及相对或粘性定位；
2. 浮动：元素首先通过常规流的方式定位，然后从常规流里拿出来并定位。典型情况下，浮动向左边或右边。内容可以围绕浮动元素；
3. 绝对定位：元素被完全地从常规流里拿出来，并且根据其父元素的未知进行定位。

## 常规流（Normal Flow）
常规流是最常用的定位方法。根据元素的不同，在常规流中，元素可能参加块级格式化或者行内格式化。

### 块级格式化上下文（Block Formatting Context, BFC）
首先我们来看块级元素在常规流里是如何布局的。块级元素参加BFC的布局。以下元素会触发一个新的BFC：

* 根元素
* 浮动元素(`float`)
* 绝对定位元素(`position: absolute | fix`)
* 行内块元素(`display: inline-block`)
* 表格单元格(`display: table-cell`)
* 表格标题(`display: table-caption`)
* 匿名表格单元格元素(`display: table | table-row | table-row-group | table-header-group | table-footer-group | inline-table`)
* `overflow`不为`visible`的块元素
* `display`为`flow-root`的元素
* `contain`为`layout | content | strict`的元素
* 弹性元素(`display: flex | inline-flex`)
* 网格元素(`display: grid | inline-grid`)
* 多列容器(`column-count`或`column-width`不为`auto`)
* `column-span`为`all`的元素

参见[块格式化上下文](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)

BFC中的元素会**垂直排列**。默认情况下，元素会继承父元素的宽度。元素在垂直排列中的间隔由元素的`margin`属性决定。在垂直方向上，元素之间的`margin`会发生塌陷。

### 