---
title: react-lifting-state-up
date: '2017-11-15'
description:提升状态
categories:
-react

tags:

-react
-state

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

这两个函数是用来转换数字类型的。