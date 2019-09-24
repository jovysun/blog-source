---
title: 编写一个webpack插件
permalink: webpack-plugin
tags:
  - webpack
categories:
  - 工具
date: 2019-09-24 19:06:56
updated: 2019-09-24 19:06:56
---

# 概述

插件向第三方开发者提供了 webpack 引擎中完整的能力。本文将分两部分，第一部分介绍下插件编写的基本要求及基础 demo，第二部分将以一个实际的插件开发过程为线进一步讲解相关常用 API。

<!-- more -->

# 详述

## 最简插件

最低要求，定义一个具名函数，原型提供一个 apply 方法：

```js
class DemoPlugin {
  apply() {
    console.log('hello demo-plugin');
  }
}
module.exports = DemoPlugin;
```

使用：

```js
// webpack.config.js
const DemoPlugin = require('./plugins/demo-plugin');

module.exports = {
  ...
  plugins: [
    new DemoPlugin({
      name: 'jovy'
    })
  ]
}
```

## 基本结构

- 一个具名 JavaScript 函数。
- 在它的原型上定义 apply 方法。
- 指定一个触及到 webpack 本身的 事件钩子。
- 操作 webpack 内部的实例特定数据。
- 在实现功能后调用 webpack 提供的 callback。

```js
// 一个 JavaScript class
class MyExampleWebpackPlugin {
  // 将 `apply` 定义为其原型方法，此方法以 compiler 作为参数
  apply(compiler) {
    // 指定要附加到的事件钩子函数
    compiler.hooks.emit.tapAsync(
      'MyExampleWebpackPlugin',
      (compilation, callback) => {
        console.log('This is an example plugin!');
        console.log(
          'Here’s the `compilation` object which represents a single build of assets:',
          compilation
        );

        // 使用 webpack 提供的 plugin API 操作构建结果
        compilation.addModule(/* ... */);

        callback();
      }
    );
  }
}
```

## 实战一个压缩资源为 zip 的插件

### 环境搭建

一个基础的 webpack 环境即可，目录示例如下：
![目录结构](01.jpg)

```js
// package.json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^4.40.2",
    "webpack-cli": "^3.3.9"
  },
  "dependencies": {
    "jszip": "^3.2.2",
    "webpack-sources": "^1.4.3"
  }
}
```

### 知识准备

#### 通过 Compilation 进行文件写入

Compilation 上的 assets 可以用于文件写入，可以将 zip 资源包设置到 compilation.assets 对象上，文件写入需要使用 [webpack-sources](https://www.npmjs.com/package/webpacksources)。

```js
const { RawSource } = require('webpack-sources');
module.exports = class DemoPlugin {
  constructor(options) {
    this.options = options;
  }
  apply(compiler) {
    const { name } = this.options;
    compiler.hooks.emit.tapAsync('ZipPlugin', (compilation, callback) => {
      compilation.assets[name] = new RawSource('demo');
      cb();
    });
  }
};
```

#### Node.js 里面将文件压缩为 zip 包

使用[jszip](https://www.npmjs.com/package/jszip)

```js
var zip = new JSZip();
zip.file('Hello.txt', 'Hello World\n');
var img = zip.folder('images');
img.file('smile.gif', imgData, { base64: true });
zip.generateAsync({ type: 'blob' }).then(function(content) {
  // see FileSaver.js
  saveAs(content, 'example.zip');
});
```

### 具体编写

安装具体 node 包及配置 scripts：

```js
// package.json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^4.40.2",
    "webpack-cli": "^3.3.9"
  },
  "dependencies": {
    "jszip": "^3.2.2",
    "webpack-sources": "^1.4.3"
  }
}

```

`ZipPlugin`插件编写：

```js
// zip-plugin.js
const JSZip = require('jszip');
const { RawSource } = require('webpack-sources');

const zip = new JSZip();

class ZipPlugin {
  constructor(options) {
    this.options = options;
  }

  apply(compiler) {
    compiler.hooks.emit.tapAsync('ZipPlugin', (compilation, callback) => {
      const folder = zip.folder(this.options.filename);
      for (let filename in compilation.assets) {
        let source = compilation.assets[filename].source();
        folder.file(filename, source);
      }

      zip
        .generateAsync({
          type: 'nodebuffer'
        })
        .then(content => {
          const outputPath = this.options.filename + '.zip';

          compilation.assets[outputPath] = new RawSource(content);

          callback();
        });
    });
  }
}

module.exports = ZipPlugin;
```

`ZipPlugin`插件使用：

```js
const path = require('path');
const ZipPlugin = require('./plugins/zip-plugin');

module.exports = {
  mode: 'production',
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.join(__dirname, './dist')
  },
  plugins: [
    new ZipPlugin({
      filename: 'offline'
    })
  ]
};
```

`ZipPlugin`测试：

```shell
npm run build
```

![ZipPlugin使用效果](02.jpg)

# 参考

《极客时间》
