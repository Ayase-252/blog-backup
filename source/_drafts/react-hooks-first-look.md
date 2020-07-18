---
title: React Hooks：先睹为快
categories:
  - Programming
  - Front-end
  - React
tags:
  - React
  - Hooks
---

去年 10 月，在 React Conf 上，[Hooks API](https://youtu.be/dpw9EHDh2bM)
的出现一石激起千层浪。Hooks 试图解决前端开发中一个重要但是无可奈何的问题——状态难以重用。

<!--more-->

## 现在就可以使用的 React Hook

React Hook 已经被包含在 React 16.7.0 alpha 中。
可以通过安装`react@next`与`react-dom@next`，立即尝鲜。

```bash
# 如果使用npm的话
npm install react@next react-dom@next

# 如果使用yarn的话
yarn install react@next react-dom@next
```

在开发中，组件的状态常常由
