---
title: JSX简单说明
date: '2017-11-13'
description: JSX简单说明
categories: 
- react

tags:

- react
- jsx

---

# JSX使用说明

看下面的代码段：

```js
const element = <h1>Hello, world!</h1>;
```

这就是JSX，它既不是字符串也不是HTML，它是javascript的语言扩展。在React中推荐使用JSX来描述一个组件。虽然它看起来和模板语言有点相似，但是他具有js的所有功能。

下面是JSX的一些基本知识。

## 在JSX中使用表达式

在JSX中可以使用js表达式，需要用大括号把js表达式包起来。

比如，`2+2`、`user.name`、`format(name)`等有效的js表达式：
```js
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = {
  firstName: 'Harper',
  lastName: 'Perez'
};

const element = (
  <h1>
    Hello, {formatName(user)}!
  </h1>
);

ReactDOM.render(
  element,
  document.getElementById('root')
);
```

## JSX是一个js表达式

JSX表达式编译之后是一个普通的js对象。
```js
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stranger.</h1>;
}
```

## 指定JSX对象属性

可以使用字符串直接包起来：
```js
const element = <div tabIndex="0"></div>;
```

也可以用大括号包裹js表达式：
```js
const element = <img src={user.avatarUrl}></img>;
```
这么设置属性的时候不能用双引号把大括号包起来。

> **注意：**
> 由于JSX语法更接近js语法，所以html元素的属性名转化为驼峰命名法表示。
> 
> 比如，`class`变为`className`,`tabindex`变为`tabIndex`.

## 在JSX中指定子元素

对于没有text的标签直接用`/>`结束一个节点。

对于开放的标签元素，JSX可以设置子元素。
```js
const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```

## JSX放xss注入攻击

下面是用户输入安全的写法：
```js
const title = response.potentiallyMaliciousInput;
// This is safe:
const element = <h1>{title}</h1>;
```
React DOM引擎会在渲染把嵌入的数据前进行转义。因此，可以防止注入不确定的XSS数据。所有的数据在渲染之前都会转为字符串。

## JSX对象表示

Babel会把JSX表达式编译为`React.createElement()`格式代码。

下面两段代码表达的意思是一样的：
```js
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```
```js
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```
`React.createElement() `执行的时候会进行一些代码语法检查来减少一些bug，实际上它会生成这么一个对象：
```js
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world'
  }
};
```
这些对象叫“React elements”.可以把它们想象成你在屏幕上看到的显示内容。React读取这些对象然后渲染浏览器并维护对象的状态。

