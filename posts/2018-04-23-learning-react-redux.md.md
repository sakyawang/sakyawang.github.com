---
title: 学习React Redux（译文）
date: '2018-04-23'
description: 学习React Redux（译文）
categories: react

tags:

- react
- redux
---


[原文](https://css-tricks.com/learning-react-redux/)

Redux是一个在js应用中用来管理数据状态和UI状态的工具。对于管理随着时间的推移而变得复杂的单页应用程序（SPA）来说，它非常理想。它是框架无关的，虽然它由React编写，但是它同样可以在Anjular或者jQuery应用中使用。

此外，它是从一个“时间旅行”的实验中构思出来的 - 事实是，我们将在以后做到这一点！

正如我们之前的教程所见到的，React通过组件流转数据。进一步说，称之为“单向数据流”--数据从父组件流向子组件。因为这个特性，两个非父子关系的组件如何在React中进行通信并不明显：
![flow](https://cdn.css-tricks.com/wp-content/uploads/2016/03/redux-article-3-01.svg)

React不建议以这种方式直接进行组件间通信。即使它确实有支持这种方法的能力，但它被认为是很糟糕的做法，因为直接的组件间通信很容易出错，并导致意大利面代码 - 一个难以遵循的旧代码术语。

React提供了一个建议，但他们希望你自己实现它。下面是React文档的一部分节选：

>  对于两个没有父子关系的组件之间的通信，你可以设置自己的全局事件系统。Flux模式是一个可行的方案。

这就是Redux的用武之地。Redux提供了将所有应用程序状态存储在一个地方的解决方案，称为“存储”。组件然后将状态更改“分派”给存储，而不是直接传递给其他组件。需要了解状态更改的组件可以“订阅”存储：
![redux](https://cdn.css-tricks.com/wp-content/uploads/2016/03/redux-article-3-02.svg)

存储可以被认为是应用中所有状态变化的“中间人”。因为Redux的参与，组件之间不直接通信，而是所有的状态变换都要通过存储这个单一数据源。

这与部分应用程序直接相互通信的其他策略有很大不同。有时候，这些策略被认为是容易出错和混淆的原因如下：
![other](https://cdn.css-tricks.com/wp-content/uploads/2016/03/redux-article-3-03.svg)

显而易见，借助Redux，所有组件都可以从商店获取其状态。同样清楚的是，组件应该发送状态更改 - 也就是存储。发起更改的组件只需要关注将更改分派给存储，而不必担心需要状态更改的其他组件。这就是Redux如何使数据流更易于推理。

使用存储来协调应用状态的概念就是Flux模式。这是一种设计模式，可以像React一样支持单向数据流体系结构。Redux很像Flux，但是它们有多相似呢？

## Redux 是类Flux

Flux是一种模式，而不是类似Redux的工具，所以它不是你可以下载的东西。Redux是一个受Flux模式启发的工具，还包括其他的东西比如Elm。有很多比较Redux和Flux的指南。他们中的大多数人会得出结论，Redux是Flux或者是类似Flux的，这取决于我们如何严格定义Flux的规则。最终，这并不重要。Facebook非常喜欢并支持Redux，因此他们聘请了其主要开发人员Dan Abramov。

本文假定您不熟悉Flux模式。但如果你是，你会发现一些小的差异，特别是Redux的三大指导原则：

### 1. 真相的唯一来源

Redux仅为其所有应用程序状态使用一个存储。由于所有状态都驻留在一个地方，Redux称这是真相的唯一来源。

存储的数据结构最终取决于您，但在实际应用程序中它通常是一个深层嵌套对象。

Redux的这种单存储方法是它与Flux的多存储方法之间的主要区别之一。

### 2. 状态是只读的

根据Redux的文档，“改变状态的唯一途径就是发出一个描述是发生了何事的动作对象”。

这就意味着应用不能直接修改状态。相反，“动作”是被分发用来改变存储中的状态。

存储对象自身有一个包含四个方法的API：

* store.dispatch(action)
* store.subscribe(listener)
* store.getState()
* replaceReducer(nextReducer)

如你所见，没有设置状态的方法。因此，分发一个动作是应用代码表示状态变换的唯一途径：

```JavaScript
var action = {
  type: 'ADD_USER',
  user: {name: 'Dan'}
};

// Assuming a store object has been created already
store.dispatch(action);
```

`dispatch()`方法发送一个动作对象给Redux。这个动作可以被描述为一个“有效载荷”，它携带一个`type`和用来更新状态的所有其他数据 - 上面例子中式用户信息。请记住，在`type`属性之后，操作对象的设计取决于您。

### 3. 使用纯函数进行更改状态

如上所述，Redux不允许应用程序直接更改状态。相反，分发的动作“描述”状态变化和改变状态的意图。Reducers是您编写的处理调度动作并可以实际更改状态的函数。

一个reducer将当前状态作为参数，并且只能通过返回新状态来修改状态：

```js
// Reducer Function
var someReducer = function(state, action) {
  ...
  return state;
}
```

Reducers应该是“纯函数”，“纯函数”是具有以下特征的功能的函数：

* 它不会进行外部网络或数据库调用
* 其返回值完全取决于其参数的值
* 参数应该被认为是“不可变的”，这意味着它们不应该改变。
* 使用同一组参数调用纯函数将始终返回相同的值。

他们之所以被称为“纯粹”是因为它们什么都不做只根据参数返回基于参数的结果。他们对系统的任何其他部分没有副作用。

## 我们第一个Redux存储

首先，使用`Redux.createStore()`创建一个存储，并将所有reducer作为参数传递。我们来看一下只有一个reducer的简单例子：

```js
// Note that using .push() in this way isn't the
// best approach. It's just the easiest to show
// for this example. We'll explain why in the next section.

// The Reducer Function
var userReducer = function(state, action) {
  if (state === undefined) {
    state = [];
  }
  if (action.type === 'ADD_USER') {
    state.push(action.user);
  }
  return state;
}

// Create a store by passing in the reducer
var store = Redux.createStore(userReducer);

// Dispatch our first action to express an intent to change the state
store.dispatch({
  type: 'ADD_USER',
  user: {name: 'Dan'}
});
```
下面是代码的简要说明：

1. 存储创建附带一个reducer
2. reducer初始化存储为一个空数组。
3. 动作分发创建一个用户
4. reducer添加用户用到state更新存储并返回。

**reducer被调用了两次**--一次是在存储创建的时候，然后分发之后又调用了执行了。

当存储被创建，Redux立即调用reducers并使用它们的返回值作为初始化状态。第一次调用reducer返回`undefined`给状态。Reducer代码预计到这一点，并返回一个空数组来初始化存储的初始状态。每次调度动作时也会调用reducer。由于从reducer返回的状态将成为我们在存储中的新状态，**Redux总是期望reducer返回状态。**

在这个例子中，在分发之后第二次调用reducer。请记住，分派的动作描述了更改状态的意图，并且通常会携带新状态的数据。这一次，Redux将当前状态（仍是一个空数组）与action对象一起传递给reducer。动作对象现在具有`ADD_USER`类型属性，让reducer知道如何更改状态。

很容易将reducer想象为允许状态通过的漏斗。这是因为reducers总是收到并返回状态以更新存储：
![reducers](https://cdn.css-tricks.com/wp-content/uploads/2016/03/redux-article-3-04.svg)

基于该示例，我们的存储现在将成为一个包含一个用户对象的数组：

```js
store.getState();   // => [{name: 'Dan'}]
```

## 不要变动状态，拷贝它

虽然我们例子中的reducer在技术上起作用，但它改变状态这种做法很差的方式。尽管reducer负责改变状态，但不应该直接改变“当前状态”参数。这就是为什么我们不应该在reducer的状态参数中使用`.push（）`这种突变方法。

传递给reducer的参数应该被认为是不可变的。换句话说，他们不能被直接修改。相比直接修改，我们可以使用`.concat()`这样的非变异方法来克隆一个数组，然后我们改变克隆的数组：

```js
var userReducer = function(state = [], action) {
  if (action.type === 'ADD_USER') {
    var newState = state.concat([action.user]);
    return newState;
  }
  return state;
}
```

通过对reducer的更新，添加新用户会更改并返回状态参数的副本。不添加新用户时，请注意返回原始状态而不是创建副本。

下面有关于不可变数据结构的一节介绍了这些类型的最佳实践。

您可能还注意到，现在初始状态来自ES2015默认参数。到目前为止，在本系列中，我们避免了ES2015让您专注于主要主题。但是，Redux在ES2015上更加出色。因此，我们会在本文中开始使用ES2015。不要担心，每次使用新的ES2015功能时，都会指出并解释。

## 多Reducer

最后一个例子是一个很好的入门，但是大多数应用程序在整个应用程序中需要更复杂的状态。由于Redux只使用一个存储，因此我们需要使用嵌套对象将状态组织到不同的部分。让我们想象我们希望我们的存储类似于这个对象：

```js
{
  userState: { ... },
  widgetState: { ... }
}
```

对于整个应用程序来说，它仍然是“一个存储=一个对象”，但它具有可以包含各种数据的`userState`和`widgetState`的嵌套对象。这可能看起来过于简单化，但实际上它与真正的Redux存储相去不远。为了使用嵌套对象创建存储，我们需要用reducer定义每个部分：

```js
import { createStore, combineReducers } from 'redux';

// The User Reducer
const userReducer = function(state = {}, action) {
  return state;
}

// The Widget Reducer
const widgetReducer = function(state = {}, action) {
  return state;
}

// Combine Reducers
const reducers = combineReducers({
  userState: userReducer,
  widgetState: widgetReducer
});

const store = createStore(reducers);
```

`combineReducers（）`允许我们根据不同的逻辑部分来描述我们的存储，并将reducer分配给每个部分。现在，当每个reducer返回初始状态时，该状态将进入其各自的`storeState`或`widgetState`部分。

值得注意的是，现在，每个reducer都会通过其整体状态的各个子部分，而不是整个存储的状态值，就像one-reducer示例一样。然后从每个reducer返回的状态适用于其子部分。

## 分发行动之后调用哪个reducer

所有的reducers。如果我们认为每次发布操作时都会将reducer与漏斗进行比较，那么所有reducer都将被调用，并且将有机会更新其各自的状态：
![dispatch](https://cdn.css-tricks.com/wp-content/uploads/2016/03/redux-article-3-05.svg)

我仔细地说“他们的”状态，因为reducer的“当前状态”参数及其返回的“更新后的”状态仅影响存储的reducer的部分。请记住，正如前一节所述，每个reducer只能通过其各自的状态，而不是整个状态。

## 动作策略

实际上有很多创建和管理动作和动作类型的策略。虽然他们很好，但并不像本文中的其他信息那么重要。为了保持文章更短，我们已经记录了你该知道的基本策略在[github仓库](https://github.com/bradwestfall/CSS-Tricks-React-Series/blob/master/guide-3-redux/docs/action-strategies.md)。

## 不可变数据结构

> 状态的结构取决于你：它可以是一个原始结构，一个数组，一个对象或者甚至是一个Immutable.js 数据结构。唯一重要的部分是“你不能改变状态对象，而是返回一个新的状态变更后的状态对象”--[Redux docs](https://github.com/reactjs/redux#the-gist)

该声明说了很多,同时我们已经在本教程中提到了这一点。我只会强调一些要点。

首先：

* js原始数据类型（Number, String, Boolean, Undefined, Null）是不可变的
* Objects, arrays,  functions 是可变的

有人说过，数据结构上的可变性很容易出现错误。由于我们的存储将由状态对象和数组组成，因此我们需要实施一个策略来保持状态不变。

让我们想象一个我们需要改变属性的状态对象。这里有三种方法：

```js
// Example One
state.foo = '123';

// Example Two
Object.assign(state, { foo: 123 });

// Example Three
var newState = Object.assign({}, state, { foo: 123 });
```

第一个和第二个例子改变了状态对象。第二个例子改变是因为`Object.assign()`方法合并所有的参数到第一个参数中。但这也是为什么第三个例子不会改变状态的原因。

第三个例子合并`state`和`{foo: 123}`到一个新的空对象。这是一个常见的技巧，它使我们能够在不影响原始状态的情况下创建状态副本并对副本进行变更。

对象“传播运算符”是保持状态不变的另一种方法：

```js
const newState = { ...state, foo: 123 };
```

## 初始化状态和时间旅行

如果你阅读了[文档](http://redux.js.org/docs/api/createStore.html),你可能意识到`createStore()`的第二个参数，是为了`initial state`。这看起来像是reducer创建初始状态的替代方案。但是，这个初始状态只能用于“state hydration”。

想象一下，用户会对SPA进行刷新，并且存储的状态将重置为reducer初始状态。这可能不是用户所期望的。

相反，想象一下，您可以使用策略来固化存储，然后您可以在刷新时将其重新水合到Redux中。这是将初始状态发送到`createStore（）`的原因。

## Redux with React

正如我们已经讨论过的，Redux是框架无关的。先了解Redux的核心概念，然后再考虑它如何与React协同工作。但是现在我们准备从上一篇文章中获取一个容器组件，并将Redux应用到它。

首先，这里是没有Redux的原始组件：

```js
import React from 'react';
import axios from 'axios';
import UserList from '../views/list-user';

const UserListContainer = React.createClass({
  getInitialState: function() {
    return {
      users: []
    };
  },

  componentDidMount: function() {
    axios.get('/path/to/user-api').then(response => {
      this.setState({users: response.data});
    });
  },

  render: function() {
    return <UserList users={this.state.users} />;
  }
});

export default UserListContainer;
```

当然，它会执行它的Ajax请求并更新它自己的本地状态。但如果应用程序中的其他区域需要根据新获得的用户列表进行更改，则此策略是不够的。

使用Redux策略，我们可以在Ajax请求返回时调度一个动作，而不是执行`this.setState（）`。然后这个组件和其他人可以订阅状态更改。但是这实际上给我们带来了一个问题，我们如何设置`store.subscribe（）`来更新组件的状态？

我想我可以提供几个手动将组件连接到Redux存储的示例。你甚至可以想象如何用你自己的方法来处理。但最终，在这些例子的最后，我会解释说有更好的方法，忘记手动例子。然后我将介绍名为r[eact-redux](http://redux.js.org/docs/basics/UsageWithReact.html)的官方React / Redux绑定模块。所以让我们直接跳到这一点。

## Connecting with react-redux

为了清楚起见，`react`，`redux`和`react-redux`是npm上的三个独立模块。`react-redux`模块使我们能够以更方便的方式将`React`组件连接到`Redux`。

如下：

```js
import React from 'react';
import { connect } from 'react-redux';
import store from '../path/to/store';
import axios from 'axios';
import UserList from '../views/list-user';

const UserListContainer = React.createClass({
  componentDidMount: function() {
    axios.get('/path/to/user-api').then(response => {
      store.dispatch({
        type: 'USER_LIST_SUCCESS',
        users: response.data
      });
    });
  },

  render: function() {
    return <UserList users={this.props.users} />;
  }
});

const mapStateToProps = function(store) {
  return {
    users: store.userState.users
  };
}

export default connect(mapStateToProps)(UserListContainer);
```

上面代码做了以下事情：

1. 从`react-redux`导入`connect`方法
2. 从下向上看代码。`connect`方法其实有两个参数，但是我们值提供了一个`mapStateToProps()`。
3. `connect()`方法的第一个参数是一个函数返回一个对象。对象的属性会变为组件的`props`。可以在状态中看到他们。现在，我希望函数名“mapStateToProps”更有意义。另请注意，`mapStateToProps（）`将收到一个参数，它是整个Redux存储的参数。`mapStateToProps（）`的主要思想是将组件需要的整体状态的哪些部分作为其`props`。
4. 由于第三条的原因我们不在需要`getInitialState()`。我们使用`this.props.users`替代`this.state.users`，因为`users `现在是一个道具而不是本地组件状态。
5. Ajax返回现在调度一个操作，而不是更新本地组件状态。

示例代码假设用户reducer的工作原理可能并不明显。注意存储如何拥有userState属性。但是这个名字来自哪里？

```js
const mapStateToProps = function(store) {
  return {
    users: store.userState.users
  };
}
```

这个名字来自我们结合我们的reducer的时候:

```js
const reducers = combineReducers({
  userState: userReducer,
  widgetState: widgetReducer
});
```

虽然我们没有为示例显示实际的reducer（因为它将在另一个文件中），但它是确定其各自状态的子属性的reducer。为了确保`.users`是`userState`的属性，这些示例的缩减器可能如下所示：

```js
const initialUserState = {
  users: []
}

const userReducer = function(state = initialUserState, action) {
  switch(action.type) {
  case 'USER_LIST_SUCCESS':
    return Object.assign({}, state, { users: action.users });
  }
  return state;
}
```

## Ajax Lifecycle Dispatches

在我们的Ajax示例中，我们只调度了一个动作。它故意被称为“USER_LIST_SUCCESS”，因为我们可能还希望在Ajax启动前发送'USER_LIST_REQUEST'，在Ajax发生故障时发送'USER_LIST_FAILED'。请务必阅读有关[异步操作](http://redux.js.org/docs/advanced/AsyncActions.html)的文档。

## Dispatching from Events

在之前的文章中，我们看到事件应该从容器组件传递到展示组件。事实证明，在事件只需发送一个动作的情况下，react-redux同样有效：

```js
const mapDispatchToProps = function(dispatch, ownProps) {
  return {
    toggleActive: function() {
      dispatch({ ... });
    }
  }
}

export default connect(
  mapStateToProps,
  mapDispatchToProps
)(UserListContainer);
```

在展示组件中，我们可以像以前那样做`onClick = {this.props.toggleActive}`，但是这次我们不必自己编写事件。

## Container Component Omission

有时，容器组件只需要订阅存储，并且不需要像`componentDidMount（）`这样的方法来启动Ajax请求。它可能只需要一个`render（）`方法将状态传递给展示组件。在这种情况下，我们可以通过以下方式创建一个容器组件：

```js
import React from 'react';
import { connect } from 'react-redux';
import UserList from '../views/list-user';

const mapStateToProps = function(store) {
  return {
    users: store.userState.users
  };
}

export default connect(mapStateToProps)(UserList);
```

是的，这是我们新的容器组件的整个文件。但是请等待，容器组件在哪里？我们为什么不在这里使用`React.createClass（）`？

事实证明，`connect（）`为我们创建了一个容器组件。注意这次我们直接传递了展示组件，而不是传入自己创建的容器组件。如果您真的想到容器组件所做的事情，请记住它们存在以允许演示组件只关注视图而不关注状态。他们也以属性形式将状态传递给子视图。这正是`connect（）`所做的 - 它将状态（通过props）传递给我们的展示组件，并实际返回一个包装容器组件的React组件。实质上，该包装器是一个容器组件。

那么这是否意味着之前的例子实际上是两个包装展示性的容器组件？当然，你可以这样想。但这不是问题，只有当我们的容器组件需要除`render（）`之外的更多React方法时才有必要。

将这两个容器组件看作服务于不同但相关的角色:
![presentational](https://cdn.css-tricks.com/wp-content/uploads/2016/03/redux-article-3-06.svg)

嗯，也许这就是为什么React标志看起来像一个原子！

## Provider

为了使这个`react-redux`代码正常工作，您需要让应用程序知道如何将`react-redux`与`<Provider />`组件一起使用。这个组件包装你的整个React应用程序。如果您使用的是React Router，它将如下所示：

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import store from './store';
import router from './router';

ReactDOM.render(
  <Provider store={store}>{router}</Provider>,
  document.getElementById('root')
);
```

被附加在`Provider`上的存储是通过``react-redux``真正“连接”React和Redux。

## Redux with React Router

这不是必需的，但还有另一个名为`react-router-redux`的`npm`项目。由于路由在技术上属于UI状态的一部分，并且React Router不知道Redux，因此该项目有助于将两者联系起来。

你看到我在那里做了什么？我们走了一圈，然后回到第一篇文章！# 

## Final Project

本系列的[最终项目指南](https://github.com/bradwestfall/CSS-Tricks-React-Series/tree/master/guide-3-redux)允许您创建一个小的“用户和小部件”单页应用程序：
![final](https://raw.githubusercontent.com/bradwestfall/CSS-Tricks-React-Series/master/guide-3-redux/docs/preview.gif)

与本系列中的其他文章一样，每个文章都附带了一份指南，该指南更详细地介绍了指南在GitHub中的工作方式。

## Summary

我真的很希望你喜欢这个系列，就像我写的一样。我意识到有很多关于React的话题我们没有涉及（表格为一），但我试图保持真实的前提，即我想让新用户了解如何通过基本知识以及制作单页应用程序的感受。

## 系列文章

**第一部分：[React Router](https://css-tricks.com/learning-react-router/)

**第二部分：[Container Components](https://css-tricks.com/learning-react-container-components/)

**第三部分：Redux