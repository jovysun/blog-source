---
title: React基础知识
permalink: react-basic
tags:
  - React
categories:
  - 框架与库
date: 2019-07-03 08:36:40
updated: 2019-07-03 08:36:40
---

# 概述

对 React 的基础知识作一个总结。

<!-- more -->

# 详述

## Hello World

```js
ReactDOM.render(<h1>Hello, world!</h1>, document.getElementById('root'));
```

## JSX

不是新事物，而是一种语法糖，Babel 转译后调用 React.createElement() ，因此具有 JavaScript 的全部功能。

类似一些前端模板引擎的语法，JSX 中用大括号`{}`来包裹 JavaScript 变量和表达式。

属性用 camelCase 命名法，例如 class-className、tabindex-tabIndex。属性值如果是字符串值直接用引号，如果是表达式用大括号。

```js
// 引用变量
const name = 'Josh Perez';
const element = <h1>Hello, {name}</h1>;

// 引用表达式
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = {
  firstName: 'Harper',
  lastName: 'Perez'
};

const element = <h1>Hello, {formatName(user)}!</h1>;

ReactDOM.render(element, document.getElementById('root'));

// 属性值为字符串
const element = <div tabIndex="0" />;

// 属性值为表达式
const element = <img src={user.avatarUrl} />;

// 需要把 CSS 属性变成一个对象再传给元素
const element = <h1 style={{ fontSize: '12px', color: 'red' }}>hello react</h1>;
```

## 元素

元素是构成 React 应用的最小砖块。React 元素是不可变对象。一旦被创建，你就无法更改它的子元素或者属性。一个元素就像电影的单帧：它代表了某个特定时刻的 UI。更新 UI 唯一的方式是创建一个全新的元素，并将其传入 ReactDOM.render()。

## 组件

组件是独立可复用的代码片段，由一个或多个元素组成，是根据 UI 需要对元素的进一步抽象封装，因此可以自由的拆分组合使用，就像使用 HTML 元素那样。形式上分为函数组件和 class 组件，一般不需要维护私有状态 state 的，推荐用函数组件。

```js
// 函数组件
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
// class组件
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
// 组件的组合使用
function Avatar(props) {
  return (
    <img className="Avatar" src={props.user.avatarUrl} alt={props.user.name} />
  );
}
function UserInfo(props) {
  return (
    <div className="UserInfo">
      <Avatar user={props.user} />
      <div className="UserInfo-name">{props.user.name}</div>
    </div>
  );
}
function Comment(props) {
  return (
    <div className="Comment">
      <UserInfo user={props.author} />
      <div className="Comment-text">{props.text}</div>
      <div className="Comment-date">{formatDate(props.date)}</div>
    </div>
  );
}
```

## Props 与 State

组件可以把它的 state 作为 props 向下传递到它的子组件中，而子组件不能直接修改 props，只能通过事件通知父级修改；state 是记录自身状态的私有属性，可以修改。这种数据流向是“自上而下”的，一般称为单向数据流。

props.children 可以理解为插槽。

state 修改需要注意三点：

1. 不要直接修改 State，而是调用 `setState()`方法；
2. State 的更新可能是异步的，因此不能依赖当前状态去更新下一个状态；
3. State 的更新会被合并。

## 生命周期

组件的生命周期图谱如下：
![生命周期图谱](life.jpg)
通常在 constructor 中进行 state 等初始化工作，componentDidMount 中进行 DOM 渲染完成后的工作，componentWillUnmount 中进行资源释放工作。示例如下：

```js
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = { date: new Date() };
  }

  componentDidMount() {
    this.timerID = setInterval(() => this.tick(), 1000);
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

ReactDOM.render(<Clock />, document.getElementById('root'));
```

更多生命周期介绍请参考[官方文档](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)。

## 事件处理

React 元素的事件处理和 DOM 元素的很相似，但是有一点语法上的不同:

- React 事件的命名采用小驼峰式（camelCase），而不是纯小写。
- 使用 JSX 语法时你需要传入一个函数作为事件处理函数，而不是一个字符串。

```js
<button onClick={activateLasers}>Activate Lasers</button>
```

### 注意事项：

#### 不能通过放回 false 的方式组织默认行为，而是要显式的使用 preventDefault

```html
<!-- 错误 -->
<a href="#" onclick="console.log('The link was clicked.'); return false">
  Click me
</a>
```

```js
// 正确
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

#### 回调函数中的 this 绑定问题

因为传入的回调函数在调用的时候上下文（context）已经不是该组件实例本身，所以想要 this 指向该组件实例本身，需要传入的时候显式的指定上下文。

bind 用法：

```js
class Title extends Component {
  handleClickOnTitle(e) {
    console.log(this);
  }

  render() {
    return <h1 onClick={this.handleClickOnTitle.bind(this)}>React 小书</h1>;
  }
}
```

class fields 用法（Create React App 默认启用此语法）：

```js
class LoggingButton extends React.Component {
  // 此语法确保 `handleClick` 内的 `this` 已被绑定。
  // 注意: 这是 *实验性* 语法。
  handleClick = () => {
    console.log('this is:', this);
  };

  render() {
    return <button onClick={this.handleClick}>Click me</button>;
  }
}
```

箭头函数用法：

```js
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // 此语法确保 `handleClick` 内的 `this` 已被绑定。
    return <button onClick={e => this.handleClick(e)}>Click me</button>;
  }
}
```

## 条件渲染与列表

JSX 只是 JavaScript 的语法糖，可以完全使用 JavaScript 特性。对于列表渲染，需要在加上 key，值只要保证兄弟节点间唯一即可。

## 表单

### 受控组件

表单数据由 React 组件来管理。具体实现形式如下：

```js
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: '' };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({ value: event.target.value });
  }

  handleSubmit(event) {
    alert('提交的名字: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          名字:
          <input
            type="text"
            value={this.state.value}
            onChange={this.handleChange}
          />
        </label>
        <input type="submit" value="提交" />
      </form>
    );
  }
}
```

作为受控组件，`<textarea>` 和 `<select>`元素赋值形式做了调整，改成与 input 相同，如下：

```js

<textarea value={this.state.value} onChange={this.handleChange} />

<select value={this.state.value} onChange={this.handleChange}>
  <option value="grapefruit">葡萄柚</option>
  <option value="lime">酸橙</option>
  <option value="coconut">椰子</option>
  <option value="mango">芒果</option>
</select>
```

注意点：

文件 input 标签为非受控组件`<input type="file" />`。

设置了 value 值的受控组件是不可编辑的，若仍然可编辑，那可能是 value 值为 undefined 或 null。

### 非受控组件

将在高级部分讲解，[官方文档](https://zh-hans.reactjs.org/docs/uncontrolled-components.html)。

## 状态提升

通常，多个组件需要反映相同的变化数据，这时我们建议将共享状态提升到最近的共同父组件中去。

## 组合 vs 继承

React 有十分强大的组合模式。我们推荐使用组合而非继承来实现组件间的代码重用。

### 包含关系

概念同 Vue 的插槽

```js
// 匿名
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">Welcome</h1>
      <p className="Dialog-message">Thank you for visiting our spacecraft!</p>
    </FancyBorder>
  );
}
// 具名
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">{props.left}</div>
      <div className="SplitPane-right">{props.right}</div>
    </div>
  );
}
function App() {
  return <SplitPane left={<Contacts />} right={<Chat />} />;
}
```

# 参考

http://huziketang.mangojuice.top/books/react/lesson9

https://zh-hans.reactjs.org/docs/introducing-jsx.html
