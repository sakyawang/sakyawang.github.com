---
title: React元素渲染
date: '2017-11-13'
description: React元素渲染
categories: 
- react
tags:
- react
- js

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

## 渲染元素到DOM中

假设你的html文件中有一个`<div>`元素：

```html
<div id="root"></div>
```

我们称之为“根”DOM节点，因为它内部的一切都将由React DOM进行管理。

使用React构建的应用程序通常只有一个根DOM节点。如果你将React集成到现有的应用程序中，则可以拥有尽可能多的隔离根DOM节点。

要将React元素渲染到根DOM节点，要把根节点和React元素都传递给`ReactDOM.render()`：

```js
const element = <h1>Hello, world</h1>;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```

## 更新已渲染元素

React元素是不可变的。一旦创建了一个元素，就不能更改其子元素或属性。一个元素就像电影中的某一帧：它代表了某个时间点的UI。

更新UI的唯一方法是创建一个新元素，并将其传递给`ReactDOM.render()`。

思考下面这个时钟的例子：

```js
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

它每秒从一个`setInterval()`回调中调用`ReactDOM.render()`。

## React只更新必要的内容

React DOM将元素及其子元素与元素之前的状态进行比较，仅更新必要的DOM到指定的状态。

可以通过chrome控制台查看[例子](http://codepen.io/gaearon/pen/gwoJZk?editors=0010)。

![demo](https://reactjs.org/c158617ed7cc0eac8f58330e49e48224.gif)

尽管我们创建了一个描述整个UI树的元素，但是只有内容改变的文本节点被React DOM更新了。

> 
> In our experience, thinking about how the UI should look at any given moment rather than how to change it over time eliminates a whole class of bugs.

我们应该关注在某个时间点UI该是什么样子，而不是随着时间改变UI。
