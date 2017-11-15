---
title: react-event
date: '2017-11-15'
description: React事件处理
categories: react

tags:

-react
-event 

---


React元素的事件处理和DOM元素的事件处理很相似。有下面的一些语法差异：

* React事件使用驼峰命名法而不是小写字母+“-”命名法
* 在JSX中传入函数表达式绑定事件而不是函数名称字符串

例如，在HTML中：

```html
<button onclick="activateLasers()">
  Activate Lasers
</button>
```

在React中略有不同：

```js
<button onClick={activateLasers}>
  Activate Lasers
</button>
```

另一个区别是你不能返回`false`来阻止React中的默认行为。您必须显式调用事件的`preventDefault`方法。比如，在原生的HTML为了阻止默认的链接行为打开新页面，我们这么写：

```html
<a href="#" onclick="console.log('The link was clicked.'); return false">
  Click me
</a>
```

在React中，应该是：

```js
function ActionLink() {
  function handleClick(e) {
    e.preventDefault();
    console.log('The link was clicked.');
  }

  return (
    <a href="#" onClick={handleClick}>
      Click me
    </a>
  );
}
```

这里，`e`是一个合成事件。React根据[W3C规范](https://www.w3.org/TR/DOM-Level-3-Events/)定义了这些合成事件，所以你不必担心跨浏览器兼容性。请参阅[SyntheticEvent](https://reactjs.org/docs/events.html)参考指南以了解更多信息。

当使用React时，通常不需要调用`addEventListener`来在创建DOM元素之后添加事件监听器。相反，只需在元素初始渲染时提供一个侦听器。

当你定义一个[ES6 class](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes)格式的组件时，一种常见的模式是把事件处理程序作为类中的一个方法。比如，这个`Toggle`组件呈现一个按钮，让用户在“ON”和“OFF”状态之间切换:

```js
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    // This binding is necessary to make `this` work in the callback
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}

ReactDOM.render(
  <Toggle />,
  document.getElementById('root')
);
```

[demo](http://codepen.io/gaearon/pen/xEmzGg?editors=0010)

在JSX回调中，你必须`this`的含义。在JavaScript中类方法默认是没有边界的。如果你在没有绑定`this.handleClick`的情况下传递参数到`onClick`的话，调用该函数时`this`是`undefined`。

这不是React特有行为，这是函数在js中的[运行机制](https://www.smashingmagazine.com/2014/01/understanding-javascript-function-prototype-bind/)。通常如果你在引用一个方法时候方法名没有带`()`，比如`onClick={this.handleClick}`，你应该绑定该方法。

如果你觉得调用`bind`方法来绑定函数麻烦，有两种解决这个问题的方法。如果你使用的是实验中的[公共类字段语法](https://babeljs.io/docs/plugins/transform-class-properties/),你可以使用类字段来正确地绑定回调：

```js
class LoggingButton extends React.Component {
  // This syntax ensures `this` is bound within handleClick.
  // Warning: this is *experimental* syntax.
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```

在[Create React App](https://github.com/facebookincubator/create-react-app)中默认启用该语法。

如果你没有使用公共类字段语法，你可以在回调中使用一个[箭头函数](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions)：

```js
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // This syntax ensures `this` is bound within handleClick
    return (
      <button onClick={(e) => this.handleClick(e)}>
        Click me
      </button>
    );
  }
}
```

这种语法的问题是当`LoggingButton`每次渲染时都要创建一个不同的回调。大多数情况下，这没问题。但是如果回调函数是作为props传递给子组件的话，这些组件可能会做额外的重新渲染。我们通常建议在构造函数中绑定或使用类字段语法，以避免这种性能问题。

## 给事件处理函数传递参数

在循环内部，通常需要将一个额外的参数传递给事件处理程序。比如，如果id是行ID，则以下两种都可以工作：

```js
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

上面两行代码是等价的，区别只是一个使用[箭头函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)一个使用[函数原型链绑定](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind)。

在这两种情况下,代表React事件的参数`e`会作为参数ID之后的第二个参数传递。使用箭头函数时，我们必须明确的传递`e`，但是使用`bind`时会自动转发。
