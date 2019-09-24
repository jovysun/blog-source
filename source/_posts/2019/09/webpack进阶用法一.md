---
title: webpack进阶用法一
permalink: webpack-advance-1
tags:
  - webpack
categories:
  - 工具
date: 2019-09-16 09:18:30
updated: 2019-09-16 09:18:30
---

# 概述

本文介绍的 webpack 用法有：自动清理构建目录、自动补齐 CSS3 前缀、移动端 CSS 中的 px 自动转成 rem、静态资源内联、多页面应用打包通用方案、使用 source map、提取页面公共资源和 tree shaking 的使用和原理分析。

<!-- more -->

# 详述

## 自动清理构建目录

若频次很低，我们可以直接手动删除已构建文件。但是实际中我们要经常执行构建命令，这时候就需要自动清理构建目录文件了。思路一是用`npm scripts`，直接用系统命令：`rm -rf ./dist && webpack`，或者用 npm 包 rimraf：`rimraf ./dist && webpack`。思路二是用 webpack 插件`clean-webpack-plugin`，使用很简单，安装配置下即可，完成后每次构建就会自动清理已构建的目录文件。

<!-- package.json -->

```json
  "scripts": {
    "build": "rm -rf ./dist && webpack --config webpack.prod.js",
    // "build": "rimraf ./dist && webpack --config webpack.prod.js",
  }
```

<!-- webpack.prod.js -->

```js
plugins: [new CleanWebpackPlugin()];
```

## 自动补齐 CSS3 前缀

由于各种浏览器对于 CSS3 新特性支持程度的不同，导致实际使用中需要针对不同内核浏览器给出前缀，示例如下：

```css
.box {
  -moz-border-radius: 10px;
  -webkit-border-radius: 10px;
  -o-border-radius: 10px;
  border-radius: 10px;
}
```

实际生产中我们当然不想这么低效率，因此可以使用构建工具来自动完成这些。在 webpack 中可以使用 PostCSS 插件 autoprefixer 来实现。至于哪些属性需要加前缀，主要是根据配置的 browserslist 和[Can I Use](https://caniuse.com/)规则确定。具体使用，首先安装 npm 包`postcss-loader`和`autoprefixer`，然后配置具体参数，最后示例如下：

```js
// webpack.config.js
  {
      test: /.css$/,
      use: [
          'css-loader',
          {
              loader: 'postcss-loader',
              options: {
                  plugins: () => [
                      require('autoprefixer')()
                  ]
              }
          }
      ]
  },
```
package.json
```json
// package.json
  "browserslist": [
    "last 2 version",
    "> 1%",
    "iOS 7"
  ],
```

## 移动端 CSS 中的 px 自动转成 rem

![终端分辨率对比图](px2rem.jpg)
为什么要转换，因为各终端分辨率不同，px 是绝对单位，想要响应式布局，就得针对不同分辨率作相应的调整。思路一是用媒体查询：

```css
@media screen and (max-width: 980px) {
  .header {
    width: 900px;
  }
}
@media screen and (max-width: 480px) {
  .header {
    height: 400px;
  }
}
@media screen and (max-width: 350px) {
  .header {
    height: 300px;
  }
}
```

思路二是用相对单位 rem，rem 是相对于页面根元素 html 的大小来确定其他元素的大小。使用 rem，条件一是动态设置 html 的 font-size，条件二是根据视觉稿编写具体元素的对应 px 的 rem 值。计算根元素的 font-size 值可以使用手淘的[lib-flexible](https://github.com/amfe/lib-flexible)库，px 转成 rem 可以利用样式预编译库的类函数功能去实现，也可以利用构建工具去处理。推荐`lib-flexible`+`px2rem-loader`，示例如下：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Document</title>
    <script>
      // lib-flexible库代码
    </script>
  </head>
  <body></body>
</html>
```

```js
{
    test: /.css$/,
    use: [
        'css-loader',
        {
            loader: 'px2rem-loader',
            options: {
                remUnit: 75,
                remPrecision: 8
            }
        }
    ]
}
```

## 静态资源内联

资源内联意义有：页面框架的初始化脚本、上报相关打点、css 内联避免页面闪动和减少 http 网络请求数。

### HTML 和 JS 内联

可以用 webpack 的 raw-loader 实现，示例如下（raw-loader@0.5.1）：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    ${require('raw-loader!./meta.html')}
    <title>Document</title>
    <script>
      ${require('raw-loader!babel-loader!../../node_modules/lib-flexible/flexible.js')}
    </script>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

### CSS 内联

用 webpack 的 style-loader

```js
module: {
  rules: [
    {
      test: /\.scss$/,
      use: [
        {
          loader: 'style-loader',
          options: {
            insertAt: 'top', // 样式插入到<head>
            singleton: true //将所有的style标签合并成一个
          }
        },
        'css-loader',
        'sass-loader'
      ]
    }
  ];
}
```

### 小图片或者字体内联

用 webpack 的 url-loader

```js
module: {
  rules: [
    {
      test: /.(png|jpg|gif|jpeg)$/,
      use: [
        {
          loader: 'url-loader',
          options: {
            limit: 10240
          }
        }
      ]
    },
    {
      test: /.(woff|woff2|eot|ttf|otf)$/,
      use: [
        {
          loader: 'url-loader',
          options: {
            limit: 10240
          }
        }
      ]
    }
  ];
}
```

## 多页面应用打包通用方案

思路一，每个页面对应一个 entry，一个 html-webpack-plugin，缺点就是增删页面需要修改 webpack 配置文件。
思路二，动态获取 entry 和设置 html-webpack-plugin 数量，前提是约定各个页面文件结构，例如每个页面一个单独文件夹，每个入口文件名都为 index.js。示例如下：

```js
// webpack.config.js
'use strict';

const glob = require('glob');
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

const setMPA = () => {
  const entry = {};
  const htmlWebpackPlugins = [];

  const entryFiles = glob.sync(path.join(__dirname, './src/*/index.js'));

  Object.keys(entryFiles).map(index => {
    const entryFile = entryFiles[index];
    const match = entryFile.match(/src\/(.*)\/index\.js/);
    const fileName = match && match[1];
    entry[fileName] = entryFile;

    htmlWebpackPlugins.push(
      new HtmlWebpackPlugin({
        template: path.join(__dirname, `src/${fileName}/index.html`),
        filename: `${fileName}.html`,
        chunks: ['vendors', fileName],
        inject: true,
        minify: {
          html5: true,
          collapseWhitespace: true,
          preserveLineBreaks: false,
          minifyCSS: true,
          minifyJS: true,
          removeComments: false
        }
      })
    );
  });

  return {
    entry,
    htmlWebpackPlugins
  };
};

const { entry, htmlWebpackPlugins } = setMPA();

module.exports = {
  entry: entry,
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name]_[chunkhash:8].js'
  },

  plugins: [new CleanWebpackPlugin()].concat(htmlWebpackPlugins)
};
```

这个示例中使用了 npm 包`glob`来读文件，当然也可以直接用 node 的 fs 模块实现，或者其他功能包。

## 使用 source map

作用是通过 source map 定位到源代码，主要用于调试，所以开发环境开启，线上环境关闭。source map 类型很多，重点是掌握几个关键字的含义，然后各种类型含义也只是关键字的组合。关键字含义如下：

- source map: 产生.map 文件
- eval: 使用 eval 包裹模块代码
- cheap: 不包含列列信息
- inline: 将.map 作为 DataURI 嵌入，不不单独生成.map 文件
- module:包含 loader 的 sourcemap

## 提取页面公共资源

### 基础库分离

将 react、react-dom 基础包通过 cdn 引入，不打入 bundle 中，可以使用`html-webpackexternals-plugin`实现：

```js
// webpack.config.js
const HtmlWebpackExternalsPlugin = require('html-webpack-externals-plugin');
module.exports = {
  plugins: [
    new HtmlWebpackExternalsPlugin({
      externals: [
        {
          module: 'react',
          entry: 'https://unpkg.com/react@16/umd/react.production.min.js',
          global: 'React'
        },
        {
          module: 'react-dom',
          entry:
            'https://unpkg.com/react-dom@16/umd/react-dom.production.min.js',
          global: 'ReactDOM'
        }
      ]
    })
  ]
};
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Document</title>
  </head>
  <body>
    <div id="root"></div>
    <script src="https://unpkg.com/react@16/umd/react.production.min.js"></script>

    <script src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js"></script>
  </body>
</html>
```

### 利用 SplitChunksPlugin 进行公共脚本分离

利用 SplitChunksPlugin 分离基础包

```js
module.exports = {
  optimization: {
    splitChunks: {
      cacheGroups: {
        commons: {
          test: /(react|react-dom)/, //匹配出需要分离的包
          name: 'vendors',
          chunks: 'all'
        }
      }
    }
  }
};
```

利用 SplitChunksPlugin 分离页面公共文件

```js
module.exports = {
  optimization: {
    splitChunks: {
      minSize: 0, //分离的包体积的大小
      cacheGroups: {
        commons: {
          name: 'commons',
          chunks: 'all',
          minChunks: 2 //设置最小引用次数为2次
        }
      }
    }
  }
};
```

## tree shaking 的使用和原理分析

### 概念

1 个模块可能有多个方法，只要其中的某个方法使用到了，则整个文件都会被打到
bundle 里面去，tree shaking 就是只把用到的方法打入 bundle，没用到的方法会在
uglify 阶段被擦除掉。

### 使用

webpack4.x 在 mode 设置为 production 时默认开启，webpack3.x 与 webpack2.x 在.babelrc 里设置 modules: false 即可。

### DCE (Dead code elimination)

在保持代码运行结果不变的前提下，去除无用的代码。

- 代码执行的结果不会被用到
- 代码不会被执行，不可到达
- 代码只会影响死变量（只写不不读）
  tree shaking 是 DCE 的一种方式，它可以在打包时忽略没有用到的代码。

```js
if (false) {
console.log('这段代码永远不会执行’);
}
```

### 原理

利用 ES6 模块的特点（只能作为模块顶层的语句句出现、import 的模块名只能是字符串串常量和引入的模块不可改变），在打包阶段对静态代码进行 AST 语法分析，对有用和无用的模块打上不同标签，“uglify”阶段删除无用代码。

### 局限性

1，只能是静态声明和引用的 ES6 模块，不能是动态引入和声明的；

在打包阶段对冗余代码进行删除，就需要 webpack 需要在打包阶段确定模块文件的内部结构，而 ES 模块的引用和输出必须出现在文件结构的第一级（'import' and 'export' may only appear at the top level），否则会报错。

```js
// webpack编译时会报错
if (condition) {
  import module1 from './module1';
} else {
  import module2 from './module2';
}
```

而 CommonJS 模块支持动态结构的，所以不能对 CommonJS 模块进行 tree-shaking 处理。

2，只能处理模块级别，不能处理函数级别的冗余；
因为 webpack 的 tree-shaking 是基于模块间的依赖关系，所以并不能对模块内部自身的无用代码进行删除。

3，只能处理 JS 相关冗余代码，不能处理 CSS 冗余代码。

# 参考

《极客时间》
