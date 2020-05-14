---
title: React安装使用
permalink: react-hello
tags:
  - React
categories:
  - 框架与库
date: 2019-07-03 08:00:07
updated: 2019-07-03 08:00:07
---

# 概述

介绍 React 的各种安装使用方式，从网页直接使用，到脚手架，再到用打包工具webpack自己搭建。

<!-- more -->

# 详述

## 安装

### 网页中直接使用

一个 DOM 容器，一个 React 组件，三个 Script 标签。

```html
<!-- ... 其它 HTML ... -->

<div id="like_button_container"></div>

<!-- ... 其它 HTML ... -->
```

```js
'use strict';

const e = React.createElement;

class LikeButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = { liked: false };
  }

  render() {
    if (this.state.liked) {
      return 'You liked this.';
    }

    return e(
      'button',
      { onClick: () => this.setState({ liked: true }) },
      'Like'
    );
  }
}

const domContainer = document.querySelector('#like_button_container');
ReactDOM.render(e(LikeButton), domContainer);
```

```html
  <!-- ... 其它 HTML ... -->

  <!-- 加载 React。-->
  <!-- 注意: 部署时，将 "development.js" 替换为 "production.min.js"。-->
  <script src="https://unpkg.com/react@16/umd/react.development.js" crossorigin></script>
  <script src="https://unpkg.com/react-dom@16/umd/react-dom.development.js" crossorigin></script>

  <!-- 加载我们的 React 组件。-->
  <script src="like_button.js"></script>

</body>
```

#### 可选使用 JSX

```js
// 显示一个 "Like" <button>
return <button onClick={() => this.setState({ liked: true })}>Like</button>;
```

```shell
npm init -y
npm install babel-cli@6 babel-preset-react-app@3

npx babel --watch src --out-dir . --presets react-app/prod
```

### 创建新的 React 应用

#### Create React App 脚手架

```shell
npx create-react-app my-app
cd my-app
npm start
```

#### webpack 从零搭建

```json
// package.json
  "devDependencies": {
    "@babel/core": "^7.4.5",
    "@babel/plugin-transform-runtime": "^7.4.4",
    "@babel/preset-env": "^7.4.5",
    "@babel/preset-react": "^7.0.0",
    "babel-loader": "^8.0.6",
    "html-webpack-plugin": "^3.2.0",
    "react-redux": "^7.1.0",
    "react-router-dom": "^5.0.1",
    "redux": "^4.0.1",
    "redux-saga": "^1.0.4",
    "webpack": "^4.35.0",
    "webpack-cli": "^3.3.5",
    "webpack-dev-server": "^3.7.2"
  },
  "dependencies": {
    "react": "^16.8.6",
    "react-dom": "^16.8.6"
  }
```

```json
// .babelrc
{
  "presets": ["@babel/preset-env", "@babel/preset-react"],
  "plugins": [
    [
      "@babel/plugin-transform-runtime",
      {
        "corejs": false,
        "helpers": true,
        "regenerator": true,
        "useESModules": false
      }
    ]
  ]
}
```

```js
// webpack.config.js
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader'
        }
      }
    ]
  },
```

# 参考

https://zh-hans.reactjs.org/docs/getting-started.html
