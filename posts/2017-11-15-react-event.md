---
title: react-event
date: '2017-11-15'
description: React事件处理
categories: react

tags:

 - react
 - event
 - 
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

