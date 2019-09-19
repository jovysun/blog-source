---
title: React Router概览
permalink: react-router-overview
tags:
  - React
categories:
  - 框架与库
  - React
date: 2019-07-12 09:02:54
updated: 2019-07-12 09:02:54
---

# 概述
本文是对react-router4.x的一个概览（web应用部分），不涉及具体编码细节，只作脉络梳理。
<!-- more -->

# 详述
React Router是一组导航组件，支持Web应用与React Native应用。对应的npm包为`react-router-dom`与`react-router-native`，两者都集成了核心包`react-router`。在web分支，最基本的三个组件是总容器组件`BrowserRouter`、占位组件`Route`和链接导航组件`Link`，总容器包裹整个需要导航的组件，一般包裹所有应用组件；占位组件定义了路由匹配规则及符合规则的组件渲染于此；链接导航组件的原型是a标签，定义了路由名称及对应句柄展示内容。

## 基本示例
```js
import React from "react";
import { BrowserRouter as Router, Route, Link } from "react-router-dom";

function Index() {
  return <h2>Home</h2>;
}

function About() {
  return <h2>About</h2>;
}

function Users() {
  return <h2>Users</h2>;
}

function AppRouter() {
  return (
    <Router>
      <div>
        <nav>
          <ul>
            <li>
              <Link to="/">Home</Link>
            </li>
            <li>
              <Link to="/about/">About</Link>
            </li>
            <li>
              <Link to="/users/">Users</Link>
            </li>
          </ul>
        </nav>

        <Route path="/" exact component={Index} />
        <Route path="/about/" component={About} />
        <Route path="/users/" component={Users} />
      </div>
    </Router>
  );
}

export default AppRouter;

```
## API
- `<Router>`：所有路由器组件的通用底层接口。通常情况下，应用程序会使用其中一种高级路由器:`<BrowserRouter>、<HashRouter>、<MemoryRouter>、<NativeRouter>、<StaticRouter>`。
- `<BrowserRouter>`：使用HTML5 history API的路由实现。
- `<HashRouter>`：使用URL的hash特性的路由实现。
- `<MemoryRouter>`：<Router>的一种实现，它将您的“URL”的历史记录保存在内存中(不读取或写入地址栏)。在测试和非浏览器环境(如React Native)中非常有用。
- `<StaticRouter>`：`<Router>`的一种实现，一个从不改变位置的<路由器>，在服务器端和单元测试中很有用。
- `<Link>`：提供声明性的、可访问的导航。
- `<NavLink>`：特殊的`<Link>`，当它匹配当前URL时，将向呈现的元素添加样式属性。
- `<Redirect>`：导航到一个新的位置，类别服务器端的重定向。
- `<Route>`：核心组件，最基本的职责是在位置与路由路径匹配时呈现一些UI。
- `<Switch>`：呈现与位置匹配的第一个子节点`<Route>`或`<Redirect>`。
- `<Prompt>`：用于在离开页面之前提示用户。当您的应用程序进入一种应该防止用户导航离开的状态(就像表单只填写了一半)时，呈现一个<Prompt>。
- history：待完善
- location：待完善
- match：待完善
- matchPath：待完善
- withRouter：待完善
# 参考
