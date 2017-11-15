---
title: react-conditional-render
date: '2017-11-15'
description: React有条件渲染
categories: react

tags:

-react
-render
-conditional

---

> 在React中，你可以创建不同的组件来封装你需要的行为。然后根据应用的状态来渲染需要的内容。

React中的条件渲染和js中的条件表达式一样。使用JavaScript的关键字`if`或者[条件运算符](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)创建表示当前状态的元素，然后React更新UI来匹配元素。

想一下下面两个组件：

```js
function UserGreeting(props) {
  return <h1>Welcome back!</h1>;
}

function GuestGreeting(props) {
  return <h1>Please sign up.</h1>;
}
```

我们创建一个`Greeting`组件，根据用户是否登录来显示以上组件中的某一个：

```js
function Greeting(props) {
  const isLoggedIn = props.isLoggedIn;
  if (isLoggedIn) {
    return <UserGreeting />;
  }
  return <GuestGreeting />;
}

ReactDOM.render(
  // Try changing to isLoggedIn={true}:
  <Greeting isLoggedIn={false} />,
  document.getElementById('root')
);
```

[demo](https://codepen.io/gaearon/pen/ZpVxNq?editors=0011)

此示例根据`isLoggedIn`属性的值显示不同的问候语。

## 元素变量

可以使用变量存储元素。这可以帮助你有条件地显示组件的一部分，而其余的输出不会改变。

考虑这两个代表注销和登录按钮的新组件：

```js
function LoginButton(props) {
  return (
    <button onClick={props.onClick}>
      Login
    </button>
  );
}

function LogoutButton(props) {
  return (
    <button onClick={props.onClick}>
      Logout
    </button>
  );
}
```

在下面的例子中，我们将创建一个有状态组件`LoginControl`。

它将根据当前状态显示`<LoginButton />`或`<LogoutButton />`。它也将显示前面例子中的`<Greeting />`：

```js
class LoginControl extends React.Component {
  constructor(props) {
    super(props);
    this.handleLoginClick = this.handleLoginClick.bind(this);
    this.handleLogoutClick = this.handleLogoutClick.bind(this);
    this.state = {isLoggedIn: false};
  }

  handleLoginClick() {
    this.setState({isLoggedIn: true});
  }

  handleLogoutClick() {
    this.setState({isLoggedIn: false});
  }

  render() {
    const isLoggedIn = this.state.isLoggedIn;

    let button = null;
    if (isLoggedIn) {
      button = <LogoutButton onClick={this.handleLogoutClick} />;
    } else {
      button = <LoginButton onClick={this.handleLoginClick} />;
    }

    return (
      <div>
        <Greeting isLoggedIn={isLoggedIn} />
        {button}
      </div>
    );
  }
}

ReactDOM.render(
  <LoginControl />,
  document.getElementById('root')
);
```

[demo](https://codepen.io/gaearon/pen/QKzAgB?editors=0010)

尽管声明一个变量并使用if语句是条件渲染组件的好方法，但是有时你可能想使用更短的语法。如下所述，在JSX中有几种方法可以内联条件。

### 内联if和&&逻辑运算符

你可以在JSX中[嵌入任何表达式](https://reactjs.org/docs/introducing-jsx.html#embedding-expressions-in-jsx)，用大括号括起来。包括JavaScript逻辑&&运算符。可以很方便有条件地包含一个元素:

```js
function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 &&
        <h2>
          You have {unreadMessages.length} unread messages.
        </h2>
      }
    </div>
  );
}

const messages = ['React', 'Re: React', 'Re:Re: React'];
ReactDOM.render(
  <Mailbox unreadMessages={messages} />,
  document.getElementById('root')
);
```

[demo](https://codepen.io/gaearon/pen/ozJddz?editors=0010)

在JavaScript中，`true && expression`总是等于`expression`,`false && expression`总是等于`false`。

因此，如果条件结果是`true`，`&&`运算符右侧的元素会输出。如果结果为`false`，React会忽略并跳过该元素。

### 内联if-else和三目条件操作符

另一种内联条件渲染元素的方法是使用JavaScript的三目运算符`condition ? true : false`。

在下面的例子中，我们使用它来有条件地渲染一小块文本:

```js
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      The user is <b>{isLoggedIn ? 'currently' : 'not'}</b> logged in.
    </div>
  );
}
```

它也可以用于较大的表达式，虽然不能明显看出逻辑：

```js
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      {isLoggedIn ? (
        <LogoutButton onClick={this.handleLogoutClick} />
      ) : (
        <LoginButton onClick={this.handleLoginClick} />
      )}
    </div>
  );
}
```

就像在JavaScript中一样，根据自己和和团队认为更具可读性来选择合适的代码样式。还要记住，只要条件变得太复杂，可能是提取组件的好时机。

## 防止组件重绘

在极少数情况下，即使由另一个组件呈现，您也可能希望组件隐藏自己。为此，返回`null`而不是其渲染输出内容。

下面的例子，`<WarningBanner />`组件根据属性`warn`值进行渲染。如果属性值是`false`组件则不渲染：

```js
function WarningBanner(props) {
  if (!props.warn) {
    return null;
  }

  return (
    <div className="warning">
      Warning!
    </div>
  );
}

class Page extends React.Component {
  constructor(props) {
    super(props);
    this.state = {showWarning: true}
    this.handleToggleClick = this.handleToggleClick.bind(this);
  }

  handleToggleClick() {
    this.setState(prevState => ({
      showWarning: !prevState.showWarning
    }));
  }

  render() {
    return (
      <div>
        <WarningBanner warn={this.state.showWarning} />
        <button onClick={this.handleToggleClick}>
          {this.state.showWarning ? 'Hide' : 'Show'}
        </button>
      </div>
    );
  }
}

ReactDOM.render(
  <Page />,
  document.getElementById('root')
);
```

[demo](https://codepen.io/gaearon/pen/Xjoqwm?editors=0010)

从组件的`render`方法返回`null`不会影响组件生命周期方法的触发。例如，`componentWillUpdate`和`componentDidUpdate`仍然会被调用。

