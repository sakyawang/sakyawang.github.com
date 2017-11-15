---
title: react-state-lifecycle
date: '2017-11-14'
description: React状态和生命周期
categories: react

tags:

 - react
 - lifecycle
 - state

---

# React状态和生命周期

考虑之前我们的时钟例子：

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

到目前为止我们只学习了一种更新UI的方法。我们调用`ReactDOM.render()`来改变渲染输出。

在这里我们将学习如何真正的将`Clock`组件封装和可重用。它将设置自己的计时器，并且每秒更新一次。

我们从封装时钟对象开始：

```js
function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  ReactDOM.render(
    <Clock date={new Date()} />,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

但是，它却忽略了一个关键的要求：`Clock`设置一个定时器并每秒更新一次UI的事实应该是`Clock`的实现细节。

理想情况下，我们想生成一次`Clock`，然后`Clock`更新自己的时间：

```js
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

要实现这种功能我们需要添加“state”到`Clock`组件中。

State与props很相似，但是它是组件私有的，完全由组件控制。

我们之前提到，定义为类的组件具有一些额外的功能。本地状态正是这样的：一个只有类组件具有的功能。

## 把函数转换为类

可以通过5步把函数式组件`Clock`转变为类组件：

1. 创建与函数同名的ES6语法的class，然后继承`React.Component`。
2. 添加一个独立的`render()`空方法。
3. 把函数式组件的内容转移到render方法中。
4. 在`render()`方法中把`props`替换为`this.props`。
5. 删除函数式声明组件。

```js
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

`Clock`现在就是类式组件了。

这样我们就可以使用附加功能，例如本地状态和生命周期挂钩。

## 添加本地状态到类组件中

我们将通过三个步骤把`date`从`props`中转移到`state`中。

1. 在`render()`方法中把`this.props.date`替换为`this.state.date`：

```js
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

2. 添加一个类构造器指定初始化的`this.state`:

```js
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

注意我们是如何把`props`传递给基础构造器的：

```js
constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }
```

类组件应该总是调用基础构造器传入`props`。

3. 把`date`属性从`Clock`组件移除：

```js
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

我们稍后将计时器代码添加到组件本身。

完整的代码如下：

```js
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```
[demo](http://codepen.io/gaearon/pen/KgQpJd?editors=0010)

接下来，我们将使时钟设置自己的计时器，并每秒更新一次。

## 添加生命周期方法到类组件中

在多组件应用中，在组件销毁时释放组件占用的资源是很重要的。

当`Clock`组件第一次被渲染到DOM时我们会设置一个定时器。在React中我们称之为“mounting”。

当`Clock`组件渲染的DOM移除时我们要销毁定时器。在React中我们称之为“unmounting”。

在React组件类中我们可以声明特定的方法在mount和unmount的时候运行相关代码：

```js
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {

  }

  componentWillUnmount() {

  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

这些方法称之为“生命周期钩子”。

`componentDidMount()`钩子方法会在React组件输出渲染到DOM的时候触发。这就是设置定时器最佳时机：

```js
componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }
```

注意我们把定时器的ID赋值给`this.timerID`。

虽然`this.props`和`this.state`是由React组件自身设置的具有特殊意义的字段，你同样可以设置自己需要的存储的数据到类中，这些数据不能用于视觉输出。

如果你在`render()`中没有使用一些数据，那么他们就不应该出现在`state`中。

我们会在`componentWillUnmount()`钩子方法中销毁定时器：

```js
componentWillUnmount() {
	clearInterval(this.timerID);
}
```

最后，我们实现一个`tick()`方法供`Clock`组件每秒钟执行一次。

`tick()`方法会调用`this.setSate()`方法定时更新组件的本地状态：

```js
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

[demo](http://codepen.io/gaearon/pen/amqdNA?editors=0010)

现在时钟每秒都在走着。

让我们快速回顾一下发生了什么以及调用方法的顺序：

1. 当把`<Clock />`传递给`ReactDOM.render()`时，React调用`Clock`组件的构造方法。由于Clock需要显示当前时间，因此它会使用包含当前时间的对象来初始化`this.state`。我们稍后会更新这个状态。

2. React随后调用`Clock`组件的`render()`方法。这就是React如何知道屏幕上应该显示的内容。然后React更新DOM以匹配时钟的渲染输出。

3. 当`Clock`组件输入插入到DOM元素后，React调用`componentDidMount()`生命周期钩子。在它里面，`Clock`组件要求浏览器设置一个定时器来每秒调用一次组件的`tick()`方法。

4. 浏览器每秒调用`tick()`方法。在它里面，`Clock`组件通过用包含当前时间的对象调用`setState()`来调度UI更新。由于`setState()`调用，React知道状态已经改变，并再次调用`render()`方法来获取屏幕上应该显示的内容。这次，render（）方法中的this.state.date将会不同，所以渲染输出将包含更新的时间。 React会相应地更新DOM。

5. 如果`Clock`组件被从DOM中删除，React调用`componentWillUnmount()`生命周期钩子，停止定时器。

## 正确使用状态

关于`setState()`你该知道的三件事。

### 不要直接修改状态

比如，下面的代码不会重新渲染组件：

```js
// Wrong
this.state.comment = 'Hello';
```

正确的是使用`setState()`：

```js
// Correct
this.setState({comment: 'Hello'});
```

唯一可以分配`this.state`的地方是构造函数。

## 状态更新可能是异步的

React可能会将多个`setState()`调用批量处理为单个更新，以提高性能。

由于`this.props`和`this.state`可能会异步更新，所以你不应该依靠它们的值来计算下一个状态。

比如，下面的代码可能会更新出错：

```js
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

为了解决这个问题，使用接受函数而不是对象的形式使用`setState()`。该函数将接收前一个状态作为第一个参数，并将更新应用时的`props`作为第二个参数：

```js
// Correct
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}));
```

上面使用的是箭头函数表示法，当然普通的函数写法也是可以的：

```js
// Correct
this.setState(function(prevState, props) {
  return {
    counter: prevState.counter + props.increment
  };
});
```

## 状态是合并更新的

当调用`setState()`方法时，React会把传入的参数和原来的`state`进行合并。

比如，你的状态可能包含几个独立变量：

```js
constructor(props) {
  super(props);
  this.state = {
    posts: [],
	comments: []
  };
}
```

然后你可以单独的调用`setState()`来更新它们：

```js
  componentDidMount() {
    fetchPosts().then(response => {
      this.setState({
        posts: response.posts
      });
    });

    fetchComments().then(response => {
      this.setState({
        comments: response.comments
      });
    });
  }
```

合并是浅合并，所以`this.setState（{comments}）`表达式会保持`this.state.posts`不变，但会完全替换`this.state.comments`。

## 数据流向下

不论父组件还是子组件都不应该知道自身是有状态的还是无状态的，他们不应该关心自己是函数是组件还是类组件。

这就是为什么状态经常被称为本地或封装。除了拥有和设置它的组件之外，任何组件都无法访问它。

组件可以选择将其状态作为属性传递给其子组件：

```js
<h2>It is {this.state.date.toLocaleTimeString()}.</h2>
```

着同样适用于自定义组件：

```js
<FormattedDate date={this.state.date} />
```

`FormattedDate`组件会通过`props`接收`date`，但是不会知道`date`是来自于`Clock`组件的`state`或者`props`，还是手动输入的：

```js
function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```

这通常叫做自顶向下或者单向数据流。任何状态总是属于某些特定组件，并且从该状态派生的任何数据或UI只能影响组件树中“在其下面”的组件。

如果你把一个组件树想象为一个属性瀑布，每一个组件的状态就像二外的水源，它在任意点加入瀑布，顺着流下来。

为了显示所有的组件都是独立的，我们创建一个有三个`Clock`组件组成的`App`组件：

```js
function App() {
  return (
    <div>
      <Clock />
      <Clock />
      <Clock />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

[demo](http://codepen.io/gaearon/pen/vXdGmd?editors=0010)

我们可以看到每一个时钟都有自己的定时器并且独立的更新时间。

在React应用程序中，无论组件是有状态的还是无状态的，都被认为是组件的实现细节，可能随时间而改变。您可以在有状态组件内使用无状态组件，反之亦然。