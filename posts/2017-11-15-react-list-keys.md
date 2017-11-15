---
title: react-list-keys
date: '2017-11-15'
description: 列表和键
categories: react

tags:

 - react
 - list
 - keys

---

首先，让我们回顾一下如何在JavaScript中转换列表。

下面的代码，我们使用`map()`函数获取一个`numbers`数组并将其值加倍。我们把`map()`返回的新数组指向变量`doubled`并log输出它：

```js
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((number) => number * 2);
console.log(doubled);
```

代码日志的输出`[2, 4, 6, 8, 10]`到控制台。

在React中，将元素数组转换为元素列表和上面几乎是相同的。

## 渲染多个组件

你可以构建元素的集合，在JSX中使用大括号{}括起来使用。

下面的代码中我们用`map()`函数遍历`number`数组，每一个数字返回一个`<li/>`元素。最终，我们把生成的元素数组指向变量`listItems`:

```js
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li>{number}</li>
);
```

我们把整个`listItems`元素数组放入一个`<ul>`元素中，把他渲染到DOM中：

```js
ReactDOM.render(
  <ul>{listItems}</ul>,
  document.getElementById('root')
);
```

上面代码显示1到5之间的数字的项目符号列表。

## 基本列表组件

通常你会在一个组件内渲染列表。

我们可以将前面的例子重构成一个接受一个数组数组并输出一个无序列表的元素。

```js
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li>{number}</li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

当你运行这个代码时，会给你一个警告，让你的列表中的元素提供一个`key`。“key”是创建元素列表时需要包含的特殊字符串属性。我们将在下一节讨论为什么它很重要。

让我们给`numbers.map()`里面的列表项分配一个键，并修复缺少的关键问题。

```js
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

[demo](https://codepen.io/gaearon/pen/jrXYRR?editors=0011)

## 键

键可帮助React识别哪些项目已更改，添加或删除。数组中的元素需要一个键来给这些元素一个稳定的标识：

```js
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
);
```

选择关键字的最好方法是使用一个字符串来唯一标识来区分其兄弟元素。大多数情况下，您可以使用数据中的ID作为关键字：

```js
const todoItems = todos.map((todo) =>
  <li key={todo.id}>
    {todo.text}
  </li>
);
```

如果你要渲染的元素列表没有明确的ID，你可以使用列表项元素的索引作为键：

```js
const todoItems = todos.map((todo, index) =>
  // Only do this if items have no stable IDs
  <li key={index}>
    {todo.text}
  </li>
);
```

如果项目可以重新排序，我们不建议使用索引索引，因为这会很慢。如果您有兴趣，您可以阅读[“深入解释为什么键是必要的”](https://reactjs.org/docs/reconciliation.html#recursing-on-children)。

## 用键提取组件

