---
title: webpack进阶用法二
permalink: webpack-advance-2
tags:
  - webpack
categories:
  - 工具
date: 2019-09-17 10:19:55
updated: 2019-09-17 10:19:55
---

# 概述

012本文介绍的 webpack 用法有：Scope Hosting 的使用和原理分析、代码分割和动态 import、在 webpack 中使用 ESLint、webpack 打包组件和基础库和优化构建时命令行的显示日志。

<!-- more -->

# 详述

## Scope Hosting 的使用和原理分析

### 构建后的代码存在大量闭包代码

大量作用域包裹代码，导致体积增大（模块越多越明显）；运行代码时创建的函数作用域变多，内存开销变大。
![构建代码示例图](scope-hoisting.jpg)

### webpack 的模块机制

![](module.jpg)

### 原理

将所有模块的代码按照引用顺序放在一个函数作用域里，然后适当的重命名一些变量以防止变量名冲突。

### 使用

要求必须是 ES6 语法（原因同上一篇 tree shaking），CommonJS 不支持。若是 webpack4.x 设置 mode 为 production 后默认开启，之前版本及其他模式需要显式引用`ModuleConcatenationPlugin`插件，示例如下：

<!-- webpack.config.js -->

```js
const webpack = require('webpack')
module.exports = {
  ...
  plugins: [new webpack.optimize.ModuleConcatenationPlugin()]
};
```

## 代码分割和动态 import

对于大的 web 应用，把所有代码放到一个文件显然不合适，一般会根据需要进行分割。常见处理方式有：提出公共代码，见上一篇“提取页面公共资源”部分；脚本懒加载，使得初始加载的代码更小。

### 懒加载 JS 方式

- CommonJS：require.ensure
- ES6：动态 import（目前还没有原生支持，需要 babel 转换）

### 动态 import

#### 安装配置：

```shell
npm install @babel/plugin-syntax-dynamic-import -D
```

```json
// .babelrc
{
  "presets": ["@babel/preset-env", "@babel/preset-react"],
  "plugins": ["@babel/plugin-syntax-dynamic-import"]
}
```

#### 打包后效果示例：

![](import.jpg)

#### 懒加载使用示例：

```js
// text.js
import React from 'react';

export default () => <div>动态 import</div>;
```

```js
// index.js
'use strict';

import React from 'react';
import ReactDOM from 'react-dom';
import '../../common/index';
import logo from './images/logo.png';
import './search.less';

class Search extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      Text: null
    };
  }

  loadComponent() {
    import('./text.js').then(textModule => {
      this.setState({
        Text: textModule.default
      });
    });
  }
  render() {
    const { Text } = this.state;
    return (
      <div className='search-text'>
        {Text ? <Text></Text> : null}
        搜索文字的内容
        <img src={logo} onClick={this.loadComponent.bind(this)} />
      </div>
    );
  }
}

ReactDOM.render(<Search />, document.getElementById('root'));
```

## 在 webpack 中使用 ESLint

以下以基于 Airbnb 的 eslint 规则库的使用为例

### 安装依赖包

```shell
# airbnb的eslint规则库
npm i eslint-config-airbnb -D
# airbnb的eslint规则库依赖包
npm i eslint eslint-plugin-import eslint-plugin-react eslint-plugin-react-hooks eslint-plugin-jsx-a11y -D
# eslint解析器
npm i babel-eslint -D
# webpack loader
npm i eslint-loader -D
```

### 配置 eslint 和 webpack

```js
// .eslintrc.js
module.exports = {
  parser: 'babel-eslint',
  extends: 'airbnb',
  env: {
    browser: true,
    node: true
  },
  rules: {}
};
```

```js
// webpack.config.js
module.exports = {
    ...
    module: {
        rules: [
            {
                test: /\.(js|jsx)$/,
                exclude: /node_modules/,
                use: ['babel-loader', {
                    loader: 'eslint-loader',
                    options: {
                        fix: true
                    }
                }]
            }
        ]
    }
}
```

## webpack 打包组件和基础库

webpack 除了可以用来打包应用，也可以用来打包 js 库，当然可能打包 js 库用 rollup 更适合。本次以实现一个大整数加法库为例讲下实现过程。

要求：

- 需要打包压缩版 large-number-jovy.min.js 和非压缩版本 large-number-jovy.js；
- 支持 AMD/CJS/ESM 模块引入和 Script 标签引入；

目录结构：
![large-number项目结构图](large-number.jpg)

安装配置：

```shell
npm i terser-webpack-plugin -D
```

```js
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  mode: 'none',
  entry: {
    'large-number-jovy': './src/index.js',
    'large-number-jovy.min': './src/index.js'
  },
  output: {
    filename: '[name].js',
    library: 'largeNumberJovy', //指定库的全局变量
    libraryExport: 'default',
    libraryTarget: 'umd' //支持库引入的方式
  },
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        include: /\.min\.js$/
      })
    ]
  }
};
```

设置入口文件：

package.json 的 main 字段为 index.js

```js
// index.js
if (process.env.NODE_ENV === 'production') {
  module.exports = require('./dist/large-number-jovy.min.js');
} else {
  module.exports = require('./dist/large-number-jovy.js');
}
```

发布：
_默认已经在 npm 注册了账号，并且登录了，否则请先注册登录。_

```shell
npm publish
```

查看：
![large-number发布成功图](large-number2.jpg)

使用：

```shell
# 安装
npm i large-number-jovy -S
```

```js
// react
import LargeNumber from 'large-number-jovy';
...
{LargeNumber('999999', '1')}
```

## 优化构建时命令行的显示日志

每次构建命令行都会显示很多日志信息，我们是不是可以定制显示信息呢？例如只显示错误日志。答案是当然可以，了解下[stats](https://webpack.docschina.org/configuration/stats/#src/components/Sidebar/Sidebar.jsx)配置项。stats 值为 String 或者 Object，字符串值是内置的一套规则的名称，有如下几种：

- error-only：只在发生错误时输出
- minimal：只在发生错误或有新的编译时输出
- none：没有输出
- normal：标准输出
- verbose：全部输出

```js
// webpack.config.js
// 对于 webpack-dev-server，这个属性要放在 devServer 对象里。
module.exports = {
  //...
  stats: 'errors-only'
};
```

当然更详细配置可以用对象来罗列具体每条规则。

```js
// webpack.config.js

module.exports = {
  //..
  stats: {
    // copied from `'minimal'`
    all: false,
    modules: true,
    maxModules: 0,
    errors: true,
    warnings: true,
    // our additional options
    moduleTrace: true,
    errorDetails: true
  }
};
```

实际使用中，我们发现内置的过于简单，详细配置又过于繁琐，有没有更加友好的插件呢？

### `friendly-errors-webpack-plugin`的使用

```js
// webpack.config.js
module.exports = {
  // ..
  plugins: [new FriendlyErrorsWebpackPlugin()],
  stats: 'errors-only'
};
```

使用后效果：
![friendly-errors-webpack-plugin使用效果图](error.jpg)

# 参考

《极客时间》
