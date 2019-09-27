---
title: 编写一个webpack的loader
permalink: webpack-loader
tags:
  - webpack
categories:
  - 工具
date: 2019-09-25 15:30:28
updated: 2019-09-25 15:30:28
---

# 概述

对于 loader，我们用了很多，熟悉的有 `css-loader`、`file-loader` 等。但是 loader 的机制是什么，基本的 loader 结构是怎样的，如何搭建一个 loader 开发调试环境，如何自己编写一个 loader，异步 loader 怎么处理，怎么在 loader 中运用第三方 npm 包。本文将对以上问题进行讲解。

<!-- more -->

# 详述

loader 只是一个导出为函数的 JavaScript 模块。loader 是用来加载处理各种形式的资源，就像工厂生产线中的一道工序，输入资源，加工处理后再输出资源。

loader 类似于其他构建工具中“任务(task)”，并提供了处理前端构建步骤的强大方法。loader 可以将文件从不同的语言（如 TypeScript）转换为 JavaScript，或将内联图像转换为 data URL。loader 甚至允许你直接在 JavaScript 模块中 import CSS 文件！

[源码分析](https://segmentfault.com/a/1190000018450503)

## 最简实现

```js
// demo-loader
module.exports = function(source) {
  return source;
};
```

## 多 loader 的执行顺序

多个 loader 是链式调用，顺序是从后到前。例如下面示例：

```js
// webpack.config.js
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /.less$/,
        use: [
          'style-loader',
          'css-loader',
          'less-loader'
        ]
      }
    ]
  }
}
```

执行顺序是`less-loader` > `css-loader` > `style-loader`。

## loader-runner

webpack 的核心依赖包，用它来执行 loader。我们可以利用它在不用搭建完整 webpack 环境下进行 loader 的开发和调试。

使用方式：

```js
const { runLoaders } = require('loader-runner');

runLoaders(
  {
    resource: '/abs/path/to/file.txt?query',
    // String: 资源的绝对路径 (可添加查询字符串)

    loaders: ['/abs/path/to/loader.js?query'],
    // String[]: loaders的绝对路径 (可添加查询字符串)
    // {loader, options}[]: 包含配置项的loaders对象的绝对路径

    context: { minimize: true },
    // 附加loader上下文作为基本上下文

    readResource: fs.readFile.bind(fs)
    // 一个读取资源的函数
    // Must have signature function(path, function(err, buffer))
  },
  function(err, result) {
    // err: Error?
    // result.result: Buffer | String
    // The result
    // result.resourceBuffer: Buffer
    // The raw resource as Buffer (useful for SourceMaps)
    // result.cacheable: Bool
    // Is the result cacheable or do it require reexecution?
    // result.fileDependencies: String[]
    // An array of paths (files) on which the result depends on
    // result.contextDependencies: String[]
    // An array of paths (directories) on which the result depends on
  }
);
```

## loader-utils

webpack loaders 的工具包，提供 `getOptions`、`parseQuery` 等工具方法。使用示例如下：

```js
const loaderUtils = require('loader-utils');

module.exports = function(source) {
  // getOptions示例
  const { name } = loaderUtils.getOptions(this);
  // parseQuery示例
  const params = loaderUtils.parseQuery(this.resourceQuery); // resource: `file?param1=foo`
  if (params.param1 === 'foo') {
    // do something
  }
};
```

[更多介绍](https://github.com/webpack/loader-utils)

## 同步 loader 与异步 loader

同步 loader 的返回值可以用 `return` 也可以用 `this.callback()`，示例如下：

```js
// 同步方式一：
module.exports = function(source) {
  // 业务处理逻辑

  return source;
};

// 同步方式二：
module.exports = function(source) {
  // 业务处理逻辑

  this.callback(null, source, map, meta);
};
```

```js
// 异步方式：
module.exports = function(input) {
  const callback = this.async();
  // 业务处理逻辑

  callback(null, source);
};
```

同步`this.callback()`异步`callback()`的参数类型相同，如下：

```js
// callback的参数类型
callback(
  err: Error | null,
  content: string | Buffer,
  sourceMap?: SourceMap,
  meta?: any
);
```

## loader 中使用缓存

webpack 中默认开启 loader 缓存，如果需要关闭缓存，可以如下设置：

```js
this.cacheable(false);
```

## 编写 name-loader

实现将源数据中的 [name] 直接替换为 loader 选项中设置的 name。然后返回包含导出文本的 JavaScript 模块。

### 准备工作

#### 目录结构

![](01.jpg)

#### 工具包安装

```shell
npm i loader-runner loader-utils -S
```

#### 源文件 index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Document</title>
  </head>
  <body>
    <h1>Hey [name]!</h1>
  </body>
</html>
```

### 代码编写

#### name-loader

```js
// name-loader.js
const { getOptions } = require('loader-utils');

module.exports = function loader(source) {
  const options = getOptions(this);

  source = source.replace(/\[name\]/g, options.name);

  return `module.exports = ${JSON.stringify(source)}`;
};
```

#### name-loader 调试脚本：

```js
// test-name-loader.js
const path = require('path');
const fs = require('fs');
const { runLoaders } = require('loader-runner');

runLoaders(
  {
    resource: path.join(__dirname, './src/name/index.html'),
    loaders: [
      {
        loader: path.join(__dirname, './loaders/name-loader.js'),
        options: {
          name: 'Jovy'
        }
      }
    ],
    context: { minimize: true },
    readResource: fs.readFile.bind(fs)
  },
  function(err, result) {
    if (err) {
      console.error(err);
      return;
    }
    console.log('runLoaders result: ', result);
  }
);
```

### 执行调试脚本：

```shell
H:\workspace\webpack\my-loader>npm run test:name

> my-loader@1.0.0 test:name H:\workspace\webpack\my-loader
> node test-name-loader.js


runLoaders result:  { result:
   [ 'module.exports = "<!DOCTYPE html>\\r\\n<html lang=\\"en\\">\\r\\n<head>\\r\\n  <meta charset=\\"UTF-8\\">\\r\\n  <meta name=\\"viewport\\" content=\\"width=device-width, initial-scale=1.0\\">\\r\\n  <meta http-equiv=\\"X-UA-Compatible\\" content=\\"ie=edge\\">\\r\\n  <title>Document</title>\\r\\n</head>\\r\\n<body>\\r\\n  <h1>Hey Jovy!</h1>\\r\\n</body>\\r\\n</html>"' ],
  resourceBuffer:
   <Buffer 3c 21 44 4f 43 54 59 50 45 20 68 74 6d 6c 3e 0d 0a 3c 68 74 6d 6c 20 6c 61 6e 67 3d 22 65 6e 22 3e 0d 0a 3c 68 65 61 64 3e 0d 0a 20 20 3c 6d 65 74 61 ... >,
  cacheable: true,
  fileDependencies:
   [ 'H:\\workspace\\webpack\\my-loader\\src\\name\\index.html' ],
  contextDependencies: [] }
```

## 编写 sprite-loader

在上一个 loader 编写的环境中我们再开发一个 loader，实现自动将 css 中引用的小图片合并成雪碧图并生成新的 css 文件。

### 准备工作

#### 目录结构：

![](02.jpg)

#### spritesmith：

雪碧图合并功能选用第三方 node 模块[spritesmith](https://www.npmjs.com/package/spritesmith)，生成雪碧图 Buffer 和坐标映射。使用示例如下：

```js
// Load in dependencies
var Spritesmith = require('spritesmith');

// Generate our spritesheet
var sprites = ['fork.png', 'github.png', 'twitter.png'];
Spritesmith.run({ src: sprites }, function handleResult(err, result) {
  result.image; // Buffer representation of image
  result.coordinates; // Object mapping filename to {x, y, width, height} of image
  result.properties; // Object with metadata about spritesheet {width, height}
});
```

#### 源文件 index.css

```css
/* src/sprite/index.css */
.img1 {
  background: url(./images/1.jpg?__sprite);
}
.img2 {
  background: url(./images/2.jpg?__sprite);
}
```

### 代码编写

#### sprite-loader

```js
// loaders/sprite-loader.js
const fs = require('fs');
const path = require('path');
const Spritesmith = require('spritesmith');

module.exports = function loader(source) {
  const callback = this.async();
  const imgs = source.match(/url\((\S*)\?__sprite\)/g);
  let matchedImgs = [];

  for (let i = 0; i < imgs.length; i++) {
    const img = imgs[i].match(/url\((\S*)\?__sprite\)/)[1];
    matchedImgs.push(path.join(process.cwd(), `/src/sprite/${img}`));
  }

  Spritesmith.run({ src: matchedImgs }, function handleResult(err, result) {
    let spritePath = path.join(process.cwd(), 'dist/sprite.jpg');
    fs.writeFileSync(spritePath, result.image);

    let imgsData = [];
    Object.keys(result.coordinates).forEach(key => {
      imgsData.push(result.coordinates[key]);
    });

    imgs.forEach((img, index) => {
      source = source.replace(
        imgs[index],
        `url("dist/sprite.jpg") ${-imgsData[index].x}px ${-imgsData[index].y}px`
      );
    });

    callback(null, source);
  });
};
```

#### sprite-loader 调试脚本

```js
// src/test-sprite-loader.js
const path = require('path');
const fs = require('fs');
const { runLoaders } = require('loader-runner');

runLoaders(
  {
    resource: path.join(__dirname, './src/sprite/index.css'),
    loaders: [path.join(__dirname, './loaders/sprite-loader.js')],
    context: { minimize: true },
    readResource: fs.readFile.bind(fs)
  },
  function(err, result) {
    if (err) {
      console.error(err);
      return;
    }

    fs.writeFileSync(
      path.join(process.cwd(), 'dist/index.css'),
      result.resourceBuffer
    );

    console.log('runLoaders result: ', result);
  }
);
```

### 执行调试脚本

```shell
H:\workspace\webpack\my-loader>npm run test:sprite

> my-loader@1.0.0 test:sprite H:\workspace\webpack\my-loader
> node test-sprite-loader.js

runLoaders result:  { result:
   [ '.img1{\r\n  background: url("dist/sprite.jpg") -777px 0px;\r\n}\r\n.img2{\r\n  background: url("dist/sprite.jpg") 0px 0px;\r\n}' ],
  resourceBuffer:
   <Buffer 2e 69 6d 67 31 7b 0d 0a 20 20 62 61 63 6b 67 72 6f 75 6e 64 3a 20 75 72 6c 28 2e 2f 69 6d 61 67 65 73 2f 31 2e 6a 70 67 3f 5f 5f 73 70 72 69 74 65 29 ... >,
  cacheable: true,
  fileDependencies:
   [ 'H:\\workspace\\webpack\\my-loader\\src\\sprite\\index.css' ],
  contextDependencies: [] }
```

## 编写 loader 用法准则及注意事项

### 用法准则

- 简单易用。
- 使用链式传递。
- 模块化的输出。
- 确保无状态。
- 使用 loader utilities。
- 记录 loader 的依赖。
- 解析模块依赖关系。
- 提取通用代码。
- 避免绝对路径。
- 使用 peer dependencies。

### 注意事项

如果该 loader 是最终执行 loader（如 file-loader、style-loader 等），那么返回值应该是字符串，包含导出文本的 JavaScript 模块；如果该 loader 不是最终执行 loader（如 css-loader、sass-loader），那么返回值应该是 Buffer，将作为链式处理的 loader 的 source。

## 自定义 loader 的使用

只有一点需要注意，就是通过 `resolveLoader` 配置项指定 loader 的搜寻顺序。

### 使用示例

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HotModuleReplacementPlugin = require('webpack')
  .HotModuleReplacementPlugin;

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.join(__dirname, 'dist'),
    filename: 'bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.html$/,
        use: [
          {
            loader: 'name-loader',
            options: {
              name: 'Jovy'
            }
          }
        ]
      },
      {
        test: /\.css$/,
        loader: ['style-loader', 'css-loader', 'sprite-loader']
      },
      {
        test: /\.jpg$/,
        loader: [
          {
            loader: 'file-loader',
            options: {
              name(file) {
                if (process.env.NODE_ENV === 'development') {
                  return '[path][name].[ext]';
                }

                return '[contenthash:8].[ext]';
              }
            }
          }
        ]
      }
    ]
  },
  devServer: {
    contentBase: path.join(__dirname, 'dist'),
    compress: true,
    port: 9000,
    hot: true
  },
  // 指定loader搜寻顺序
  resolveLoader: {
    modules: [path.join(__dirname, './loaders'), 'node_modules']
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: path.join(__dirname, `src/index.html`),
      filename: `index.html`
    }),
    new HotModuleReplacementPlugin()
  ]
};
```

### 效果展示

![](03.jpg)

# 参考

https://webpack.docschina.org/contribute/writing-a-loader/

https://segmentfault.com/a/1190000014205729

《极客时间》
