---
title: React元素渲染
date: '2017-11-13'
description: React元素渲染
categories: react
tags: ["react", "js"]

---

# React元素渲染

**元素是React应用的最小构成单元。**

一个元素就是用来描述你想在屏幕上看到的内容：

```js
const element = <h1>Hello, world</h1>;
```

与浏览器的DOM元素不同，React元素是普通的对象，创建起来代价很小。React DOM负责更新DOM以匹配React元素。

> **注意：**
> 人们可能会将元素与更广为人知的“组件”概念混为一谈。元素是由组件组合而成的。

