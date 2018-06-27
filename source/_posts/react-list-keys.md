---
title: React列表和键
date: '2017-11-15'
description: 列表和键
categories: 
- react

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

键只有在数组上下文中才有意义。

比如，如果你抽取一个`ListItem`组件，应该在数组的`<ListItem />`元素上维护键，而不是在`<ListItem />`元素自身的`<li>`元素上维护键。

错误使用键示例：

```js
function ListItem(props) {
  const value = props.value;
  return (
    // Wrong! There is no need to specify the key here:
    <li key={value.toString()}>
      {value}
    </li>
  );
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // Wrong! The key should have been specified here:
    <ListItem value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

正确使用键示例：

```js
function ListItem(props) {
  // Correct! There is no need to specify the key here:
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // Correct! Key should be specified inside the array.
    <ListItem key={number.toString()}
              value={number} />

  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

[demo](https://codepen.io/gaearon/pen/ZXeOGM?editors=0010)

一个好的经验法则是`map()`调用中的元素需要键。

## 键在列表中必须是唯一的

一个元素的键在元素数组中必须是唯一的。但是在全局范围不需要保证唯一性。当我们生成两个不同的数组时，我们可以使用相同的键：

```js
function Blog(props) {
  const sidebar = (
    <ul>
      {props.posts.map((post) =>
        <li key={post.id}>
          {post.title}
        </li>
      )}
    </ul>
  );
  const content = props.posts.map((post) =>
    <div key={post.id}>
      <h3>{post.title}</h3>
      <p>{post.content}</p>
    </div>
  );
  return (
    <div>
      {sidebar}
      <hr />
      {content}
    </div>
  );
}

const posts = [
  {id: 1, title: 'Hello World', content: 'Welcome to learning React!'},
  {id: 2, title: 'Installation', content: 'You can install React from npm.'}
];
ReactDOM.render(
  <Blog posts={posts} />,
  document.getElementById('root')
);
```

[demo](https://codepen.io/gaearon/pen/NRZYGN?editors=0010)

键可以作为React的提示，但不会传递给组件。如果您的组件中需要相同的值，请将其明确传递为具有不同名称的属性：

```js
const content = posts.map((post) =>
  <Post
    key={post.id}
    id={post.id}
    title={post.title} />
);
```

上面的例子中，`Post`组件可以读取`props.id`而不能读取`props.key`。

## 在JSX中嵌入`map()`

在上面的例子中我们声明了一个`listItems`变量,并在JSX中使用：

```js
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <ListItem key={number.toString()}
              value={number} />

  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```

JSX允许在大括号中嵌入任何表达式，所以我们可以内联`map()`返回值：

```js
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {numbers.map((number) =>
        <ListItem key={number.toString()}
                  value={number} />

      )}
    </ul>
  );
}
```

[demo](https://codepen.io/gaearon/pen/BLvYrB?editors=0010)
