---
title: React组件和属性

date: '2017-11-14'

description: React组件和属性

categories:
- react

tags:

- react
- component
- props
---

> 组件让你把用户界面分成独立的，可重复使用的部分，并且将每个部分分开考虑。

概念上组件就像js的function。它们接受任意的输入（props），然后返回描述屏幕上要展现内容的React元素。

## 函数式组件和类组件

最简单定义组件的方法就是写一个js的function：

```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

这个函数是一个有效的React组件，因为它接受一个单独的“props”参数（属性的汇集对象）并返回一个React组件。我们称呼这些组件为函数式组件，因为他们就是javascript的函数。

也可以使用ES6的class来定义一个组件：

```js
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

以上两种组件表示方法在React看来是等价的。

## 渲染组件

之前的例子中我们只看到了React元素通过html标签表示:

```js
const element = <div />;
```

然而React元素同样可以使用Reat组件表示：

```js
const element = <Welcome name="Sara" />;
```

当React遇到元素由用户自定义组件表示时，会把JSX的属性作为一个单独的对象传递给组件。这个单独的对象就是“props”。

比如，下面的代码会渲染“Hello, Sara”到浏览器屏幕：

```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```

我们来看下上面代码都发生了什么：

1. 我们调用`ReactDOM.render()`使用`<Welcome name="Sara" />`元素作为参数。
2. React调用`Welcome`组件时传入`{name: 'Sara'}`作为props参数。
3. 我们的`Welcome`组件返回一个`<h1>Hello, Sara</h1>`元素。
4. React DOM有效更新DOM元素匹配`<h1>Hello, Sara</h1>`。

> 注意：
> 自定义组件作为标签使用时首字母要大写。
> 
> 比如，`<div />`表示一个DOM标签，但是`<Welcome/>`表示一个React组件。

## 组件结构

组件可以在其输出中引用其他组件。这让我们可以使用相同的组件抽象来实现任何细节层次。一个按钮，一个表单，一个弹窗，一个页面，这些在React应用中通常都表示为一个组件。

比如，我们可以创建一个呈现多次`Welcome`的`App`组件：

```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

通常情况下，一个React应用在最上层只有一个`App`组件。然而如果你把React整合到已有的应用中时，可以自底向上的使用小组件然后慢慢的重构到顶层。

## 提取组件

不要害怕将组件分解成更小的组件。

比如，思考下面的`Comment`组件：

```js
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img className="Avatar"
          src={props.author.avatarUrl}
          alt={props.author.name}
        />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

`Comment`组件接收`author`（对象）、`text`（字符串）、`data`（日期）作为“props”，然后在页面上展示一个评论列表。

由于所有的嵌套，这个组件可能会很难改变，并且很难重用它的各个部分。我们可以从中提取一些组件。

第一步，我们抽取`Avatar`：

```js
function Avatar(props) {
  return (
    <img className="Avatar"
      src={props.user.avatarUrl}
      alt={props.user.name}
    />

  );
}
```

`Avatar`组件不关心它是否会在`Comment`中渲染。这就是为什么我们把它的属性命名为更通用的`user`而不是`author`。

在这里建议命名组件属性的时候尽量使用符合组件自身的特性的属性名字，而不是和上下文相关的名字。

现在我们可以简化一点`Comment`：

```js
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <Avatar user={props.author} />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

接来下我们抽取一个`UserInfo`组件用来展示`Avatar`和用户名：

```js
function UserInfo(props) {
  return (
    <div className="UserInfo">
      <Avatar user={props.user} />
      <div className="UserInfo-name">
        {props.user.name}
      </div>
    </div>
  );
}
```

这样我们可以更进一步简化`Comment`组件：

```js
function Comment(props) {
  return (
    <div className="Comment">
      <UserInfo user={props.author} />
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```

一开始抽取组件可能很麻烦，但是可重用的组件在大型应用中是很有价值的。

> 一个经验原则：
> 
> 如果一个UI的一部分多次重复使用（`Button`、`Panel`、`Avatar`）,或者自身足够复杂（`App`、`FeedStory`、`Comment`），这时候就该把它们改写为可重用的组件。


## Props是只读的

不论你用函数表示法还是类表示法声明一个组件，运行中绝对不能修改组件的“props”。

考虑下面的`sum`函数：

```js
function sum(a, b) {
  return a + b;
}
```

这样的函数叫纯函数，因为函数内不会去修改函数的传入参数，并且同样的输入返回同样的结果。

作为对比，下面的函数不是纯函数，因为它修改了自身的参数：

```js
function withdraw(account, amount) {
  account.total -= amount;
}
```

React非常灵活，但是他有一个严格的准则：

**所有的React组件必须像纯函数一样操作它的props**

当然应用的UI是动态的随时变换的，下一节我们介绍一个新的概念“state”。State允许React组件随着时间的推移改变他们的输出，以响应用户操作，网​​络响应和其他任何事情，而不违反这个规则。
