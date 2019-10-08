---
title: webpack构建速度与体积优化策略
permalink: webpack-optimizing
tags:
  - webpack
categories:
  - 工具
date: 2019-10-08 09:39:54
updated: 2019-10-08 09:39:54
---

# 概述
先介绍几种速度与体积分析方法的使用，紧接着介绍各种优化方法的使用：多进程构建解析、并行压缩、使用DLLPlugin分包、使用缓存、缩小构建目标、减少文件搜索范围、CSS的Tree Shaking、图片压缩、动态Polyfill。
<!-- more -->

# 详述

## 初级分析

内置的 stats，默认每次构建控制台都会显示各种统计信息，当然也可以把统计信息导出到单独文件：

```json
// package.json
{
  "scripts": {
    "build:stats": "webpack --config webpack.prod.js --json > stats.json"
  }
}
```

## 速度分析

使用`speed-measure-webpack-plugin`，分析整个打包总耗时，每个插件和 loader 的耗时情况。使用示例如下：

```js
// webpack.config.js
const SpeedMeasureWebpackPlugin = require('speed-measure-webpack-plugin');

const smp = new SpeedMeasureWebpackPlugin();

module.exports = smp.wrap({
  entry: '',
  output: '',
  ...
})
```

## 体积分析

使用`webpack-bundle-analyzer`，构建完成自动打开 8888 端口页面，可视化分析各个部分大小。使用示例如下：

```js
// webpack.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  entry: '',
  output: '',
  plugins: [
    new BundleAnalyzerPlugin()
  ]
  ...
}
```

详细见[官网](https://github.com/webpack-contrib/webpack-bundle-analyzer)

## 使用高版本的 webpack 和 Node.js

毋庸赘述，工具本身每次版本升级，基本都会有相应的性能提升。

## 多进程/多实例：构建解析

资源并行解析可选方案，webpack4.x 推荐用`thread-loader`，之前版本可以用`happypack`插件。`thread-loader`使用示例如下：

```js
// webpack.config.js
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: [
          {
            loader: 'thread-loader',
            options: {
              workers: 3  //开启进程数量
            }
          },
          'babel-loader'
        ]
      }
    ]
  }
  ...
}
```

## 多进程/多实例：并行压缩

webpack4.x 推荐用`terser-webpack-plugin`，其他版本可选方案`parallel-uglify-plugin`、`uglifyjs-webpack-plugin`。`terser-webpack-plugin`使用示例如下：

```js
// webpack.config.js
module.exports = {
  ...
  optimization: {
    minimizer: [
        new TerserWebpackPlugin({
            parallel: true
        })
    ]
  },
  ...
}
```

## 分包

前面的文章中介绍过可以用`html-webpack-externalsplugin`来分离基础包，或者用`splitChunks`来提取公共包。除此之外，更好的分包方式是用`DLLPlugin`，这个方式是预编译资源模块，因此构建速度更快。使用示例如下：

### 新增`webpack.dll.js`

```js
// webpack.dll.js
const path = require('path');
const webpack = require('webpack');

module.exports = {
  mode: 'none',
  entry: {
    library: ['react', 'react-dom', 'redux', 'react-redux']
  },
  output: {
    filename: '[name].dll.js',
    path: path.join(__dirname, 'build/library'),
    library: '[name]'
  },
  plugins: [
    new webpack.DllPlugin({
      name: '[name]',
      path: path.join(__dirname, 'build/library/[name].json')
    })
  ]
};
```

### 新增一条`scripts`

```json
{
  "scripts": {
    "dll": "webpack --config webpack.dll.js"
  }
}
```

### 配置映射

```js
// webpack.config.js

const webpack = require('webpack')

module.exports = {
  ...
  plugins: [
    new webpack.DllReferencePlugin({
      manifest: require('./build/library/library.json')
    })
  ]
}
```

### 执行分包命令

```
npm run dll
```

### 引入分包文件

```html
<!DOCTYPE html>
<html lang="en">
  <head></head>
  <body>
    <script src="../build/library/library.dll.js"></script>
  </body>
</html>
```

## 缓存

提升二次构建速度。

### `babel-loader` 开启缓存

```js
// webpack.config.js
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          {
            loader: 'babel-loader',
            options: {
              cacheDirectory: true //默认false
            }
          }
        ]
      }
    ]
  }
};
```

更多[官网](https://www.webpackjs.com/loaders/babel-loader/)

### `terser-webpack-plugin` 开启缓存

```js
// webpack.config.js
module.exports = {
  ...
optimization: {
    minimizer: [
      new TerserWebpackPlugin({
        parallel: true,
        cache: true // 只在mode为production模式下有效，默认true
      })
    ]
  }
};
```

更多[官网](https://webpack.js.org/plugins/terser-webpack-plugin/#cache)

### 使用`hard-source-webpack-plugin`

```js
// webpack.config.js
const HardSourceWebpackPlugin = require('hard-source-webpack-plugin');
module.exports = {
  ...
  plugins: [
    new HardSourceWebpackPlugin()
  ]
```

## 缩小构建目标

exclude 与 include 的使用，示例如下：

```js
// webpack.config.js
const path = require('path');
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        // include: path.resolve('src'),
        use: ['babel-loader']
      }
    ]
  }
```

## 减少文件搜索范围

- 优化 resolve.modules 配置（减少模块搜索层级）
- 优化 resolve.mainFields 配置
- 优化 resolve.extensions 配置
- 合理使用 alias

```js
// webpack.config.js
const path = require('path');
module.exports = {
  ...
  resolve: {
    alias: {
      react: path.resolve(__dirname, './node_modules/react/umd/react.production.min.js')
    }, //直接指定react搜索模块，不设置默认会一层层的搜寻
    modules: [path.resolve(__dirname, 'node_modules')], //限定模块路径
    extensions: ['.js'], //限定文件扩展名
    mainFields: ['main'] //限定模块入口文件名
  }
```

## 使用 Tree Shaking 擦除无用的 JS 和 CSS

使用 Tree Shaking 擦除无用的 JS，见之前文章[webpack 进阶用法一](/webpack-advance-1)，本文讲下对 CSS 的处理。

```js
// webpack.config.js
const glob = require('glob');
const PurgecssPlugin = require('purgecss-webpack-plugin')

const PATHS = {
  src: path.join(__dirname, 'src')
}
module.exports = {
  ...
  plugins: [
    new PurgecssPlugin({
      paths: glob.sync(`${PATHS.src}/**/*`,  { nodir: true }),
    })
  ]
```

## 图片压缩

使用基于 Node 库的`imagemin`实现的`image-webpack-loader`。

```js
// webpack.config.js
const glob = require('glob');
const PurgecssPlugin = require('purgecss-webpack-plugin')

const PATHS = {
  src: path.join(__dirname, 'src')
}
module.exports = {
  ...
  modules: {
    rules: [
      {
        test: /.(png|jpg|gif|jpeg)$/,
        use: [
          {
            loader: 'file-loader',
            options: {
              name: '[name]_[hash:8].[ext]'
            }
          },
          {
            loader: 'image-webpack-loader',
            options: {
              mozjpeg: {
                progressive: true,
                quality: 65
              },
              // optipng.enabled: false will disable optipng
              optipng: {
                enabled: false,
              },
              pngquant: {
                quality: [0.65, 0.90],
                speed: 4
              },
              gifsicle: {
                interlaced: false,
              },
              // the webp option will enable WEBP
              webp: {
                quality: 75
              }
            }
          }
        ]
      }
    ]
  }
```

## 使用动态 Polyfill 服务

原理：识别 User Agent，下发不同的 Polyfill。

### polyfill.io 官方提供的服务

```html
<script src="https://polyfill.io/v3/polyfill.js"></script>
```

### 基于官方自建 polyfill 服务

```
//huayang.qq.com/polyfill_service/v2/polyfill.min.js?unknown=polyfill&features=Promise,Map,Set
```

# 参考

《极客时间》
