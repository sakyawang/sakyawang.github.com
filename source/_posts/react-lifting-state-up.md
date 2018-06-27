---
title: React提升状态
date: '2017-11-15'
description: 提升状态
categories:
- react
tags:
- react
- state
---

通常，几个组件需要反映相同的变化数据。我们建议将共享状态提升到最接近的共同祖先。让我们看看这是如何工作的。

在本节中，我们将创建一个温度计算器，计算在给定温度下水是否会沸腾。

我们从`BoilingVerdict`组件开始。它接受一个`celsius`温度属性，打印这个温度是否可以把把水烧开：

```js
function BoilingVerdict(props) {
  if (props.celsius >= 100) {
    return <p>The water would boil.</p>;
  }
  return <p>The water would not boil.</p>;
}
```

接下来，我们创建一个`Calculator`组件。它渲染一个`<inpu/>`输入框让用户输入温度，并把它存储在`this.state.temperature`。

另外，会渲染输入参数的`BoilingVerdict`判定结果。

```js
class Calculator extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {temperature: ''};
  }

  handleChange(e) {
    this.setState({temperature: e.target.value});
  }

  render() {
    const temperature = this.state.temperature;
    return (
      <fieldset>
        <legend>Enter temperature in Celsius:</legend>
        <input
          value={temperature}
          onChange={this.handleChange} />

        <BoilingVerdict
          celsius={parseFloat(temperature)} />

      </fieldset>
    );
  }
}
```

[demo](https://codepen.io/gaearon/pen/ZXeOBm?editors=0010)

## 添加第二个输入

我们有一个新需求，除了摄氏温度输入外，我们需要一个华氏温度输入，他们俩保持同步。

我们可以从`Calculator`组件中抽取出一个`TemperatureInput`组件。我们添加一个新的`scale`属性，值可以是`c`或者`f`：

```js
const scaleNames = {
  c: 'Celsius',
  f: 'Fahrenheit'
};

class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {temperature: ''};
  }

  handleChange(e) {
    this.setState({temperature: e.target.value});
  }

  render() {
    const temperature = this.state.temperature;
    const scale = this.props.scale;
    return (
      <fieldset>
        <legend>Enter temperature in {scaleNames[scale]}:</legend>
        <input value={temperature}
               onChange={this.handleChange} />
      </fieldset>
    );
  }
}
```

我们把`Calculator`渲染为两个独立的温度输入：

```js
class Calculator extends React.Component {
  render() {
    return (
      <div>
        <TemperatureInput scale="c" />
        <TemperatureInput scale="f" />
      </div>
    );
  }
}
```

[demo](https://codepen.io/gaearon/pen/jGBryx?editors=0010)

我们现在有两个输入框，但是当在一个输入框输入温度的时候另一个输入框没有联动。这不符合我们的需求:两个输入框信息同步。

我们也不能在`Calculator`中显示`BoilingVerdict`。因为温度存储在`Calculator`中，`Calculator`不知道当前的温度。

## 编写转换函数

首先，我们编写两个函数用来转换摄氏温度和华氏温度：

```js
function toCelsius(fahrenheit) {
  return (fahrenheit - 32) * 5 / 9;
}

function toFahrenheit(celsius) {
  return (celsius * 9 / 5) + 32;
}
```

这两个函数是用来转换数字类型的。我们再编写一个方法接收一个字符串温度参数和一个转换函数，返回一个字符串。我们会用这个函数根据一个输入框的温度计算另一个输入框要显示的信息。

对于无效的参数返回空字符串，保持四舍五入到小数点后三位：

```js
function tryConvert(temperature, convert) {
  const input = parseFloat(temperature);
  if (Number.isNaN(input)) {
    return '';
  }
  const output = convert(input);
  const rounded = Math.round(output * 1000) / 1000;
  return rounded.toString();
}
```

不如，执行`tryConvert('abc', toCelsius)`返回空字符串，执行`tryConvert('10.22', toFahrenheit)`返回`50.396`。

## 提升状态

截止到目前，两个温度输入框组件在自己的state中维护温度值：

```js
class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {temperature: ''};
  }

  handleChange(e) {
    this.setState({temperature: e.target.value});
  }

  render() {
    const temperature = this.state.temperature;
    // ...  
```

然而，我们希望这两个输入框的数据是互相级联的。当我们修改摄氏温度值，华氏温度随之变化，反之亦然。

在React中，共享状态是通过将需要共享的状态数据移动到相关组件的最近的共同祖先中来完成的。这就是所谓的“提升状态”。我们会把`TemperatureInput`组件的状态数据移动到`Calculator`组件中。

如果`Calculator`组件拥有共享状态，则它将成为两个`TemperatureInput`组件中温度的“真实数据源”。它会让两个输入框的数据相互级联同步更新。
由于两个`TemperatureInput`组件的属性来自相同的父组件`Calculator`，所以两个输入将始终保持同步.

让我们分步看一下他的工作流程。

首先，我们把`TemperatureInput`组件中的`this.state.temperature`替换为`this.props.temperature`。现在，让我们假装`this.props.temperature`已经存在，虽然我们将来需要从计算器中传递它：

```js
render() {
    // Before: const temperature = this.state.temperature;
    const temperature = this.props.temperature;
    // ...
```

我们知道属性是不可修改的。当`temperature`属性是`TemperatureInput`组件的state中的值时，可以通过`this.setState()`来修改它。然而，现在`temperature`属性父元素通过props传递的，`TemperatureInput`组件不能操作他。

在React中通常使用一个受控组件来解决。就像`<input/>`DOM元素接收`value`和`onChange`属性，`TemperatureInput`组件可以从父组件`Calculator`中获取`temperature`和`onTemperatureChange`属性。

现在，当`TemperatureInput`组件要更新温度的时候，只需要调用`this.props.onTemperatureChange`：

```js
handleChange(e) {
    // Before: this.setState({temperature: e.target.value});
    this.props.onTemperatureChange(e.target.value);
    // ...
```

> 注意：
> 在自定义组件中不论是`temperature`属性还是`onTemperatureChange`属性都没有特殊涵义。我们可以随意命名，比如把常见约定是把它们命名为value和onChange。

`temperature`和`onTemperatureChange`由父组件一起提供。它将通过修改自己的本地状态来处理数据变更，从而使用新值重新渲染两个输入框。在深入修改`Calculator`组件之前，让我们总结一下对`TemperatureInput`组件的变更。我们把`temperature`从他自身状态移除，通过props获取。调用`Calculator`提供的`this.props.onTemperatureChange()`替代`this.setState()`来改变状态：

```js
class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange(e) {
    this.props.onTemperatureChange(e.target.value);
  }

  render() {
    const temperature = this.props.temperature;
    const scale = this.props.scale;
    return (
      <fieldset>
        <legend>Enter temperature in {scaleNames[scale]}:</legend>
        <input value={temperature}
               onChange={this.handleChange} />
      </fieldset>
    );
  }
}
```

现在让我们来处理下`Calculator`组件。

我们把输入的`temperature`和`scale`存储在组件的state中。这是从子组件提升上来的状态，讲作为两个输入框的真实数据源。这是为了渲染两个输入框需要准备的最小化数据。

比如，我们输入37到摄氏温度输入框中，`Calculator`组件的状态是：

```js
{
  temperature: '37',
  scale: 'c'
}
```

如果我们把华氏温度修改为212，`Calculator`组件的状态变为：

```js
{
  temperature: '212',
  scale: 'f'
}
```

我们可以存储这两个输入的值，但事实证明是不必要的。我们只需要存储最近的温度值和他的单位就可以了。我们可以根据温度值和单位计算出另一个单位的温度值。

由于输入框的值是根据同一个状态值计算出来的所以两个输入框会保持同步：

```js
class Calculator extends React.Component {
  constructor(props) {
    super(props);
    this.handleCelsiusChange = this.handleCelsiusChange.bind(this);
    this.handleFahrenheitChange = this.handleFahrenheitChange.bind(this);
    this.state = {temperature: '', scale: 'c'};
  }

  handleCelsiusChange(temperature) {
    this.setState({scale: 'c', temperature});
  }

  handleFahrenheitChange(temperature) {
    this.setState({scale: 'f', temperature});
  }

  render() {
    const scale = this.state.scale;
    const temperature = this.state.temperature;
    const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
    const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;

    return (
      <div>
        <TemperatureInput
          scale="c"
          temperature={celsius}
          onTemperatureChange={this.handleCelsiusChange} />

        <TemperatureInput
          scale="f"
          temperature={fahrenheit}
          onTemperatureChange={this.handleFahrenheitChange} />

        <BoilingVerdict
          celsius={parseFloat(celsius)} />

      </div>
    );
  }
}
```

[demo](https://codepen.io/gaearon/pen/WZpxpz?editors=0010)

现在不论哪一个输入框变更，`Calculator`组件的`this.state.temperature`和`this.state.scale`都会更新。其中一个输入框按原样得到值，所以任何用户输入都被保留，而另一个输入值总是基于它重新计算。

我们总结一下当修改输入框内容的时候发生了什么：

* React调用DOM元素`<input />`的`onChange`事件绑定的函数。在这里是调用`TemperatureInput`组件的`handleChange`方法。
* `handleChange`方法调用`TemperatureInput`组件的`this.props.onTemperatureChange()`方法处理期望值。他的属性包括`onTemperatureChange`都有父组件`Calculator`提供。
* 如果`Calculator`组件之前的渲染指定了`onTemperatureChange`使用摄氏温度输入框的处理函数`handleCelsiusChange`。根据我们修改的输入框类型来调用不同的函数来同步数据。
* 在这些方法里，`Calculator`组件调用`this.setState()`来根据我们的输入重绘。
* React调用`Calculator`组件的`render`方法告诉UI要显示成什么样。两个输入框的信息根据最后的输入和温度类型计算得来。温度转换在这一步执行。
* React调用每个由父组件`Calculator`更新过属性的`TemperatureInput`组件的`render`方法。告诉他们渲染成什么样子。
* React把DOM渲染为期望的样子。我们编辑的输入框显示输入的值，另一个输入框显示转换之后的值。

每个更新都经过相同的步骤，使输入保持同步。

## 总结

对于在React应用程序中的动态数据，应该只有一个“数据源”。通常，状态是最先加入到组件中并且重绘是需要用到的数据。然后，如果其他组件也需要它，这时需要把这个状态提升到这些组件公共的祖先中。你应该遵循[单向数据流](https://reactjs.org/docs/state-and-lifecycle.html#the-data-flows-down)原则来保持组件的状态同步，而不是在不同组件之间互相同步状态。

提升状态涉及到比双向数据绑定更多的模板代码，但是好处是，花费更少的工作找到和隔离错误。由于任何状态“存在”某个组件中，而且这个组件本身就可以改变它，所以错误的范围大大减小。此外，你可以实现任何自定义逻辑来拒绝或转换用户输入。

如果一些信息可以从属性或状态中推导出来，它可能不应该在状态中。比如，我们可以通过最后编辑的`temperature`值和他的`scale`值来推导出`celsiusValue`和`fahrenheitValue`。一个输入框总是可以根据另一个输入框的值在`render()`方法中计算出来。这可以让我们清除或应用四舍五入到其他字段，而不会丢失用户输入的任何精度。

当UI出现错误时，可以使用[React Developer Tools](https://github.com/facebook/react-devtools)进行排错，可以跟踪数据的流转：

![demo](https://reactjs.org/ef94afc3447d75cdc245c77efb0d63be.gif)
