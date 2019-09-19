---
title: Vue实战@高效的构建打包发布
permalink: vue-in-action-bundle
tags:
  - Vue
categories:
  - 框架与库
  - Vue
date: 2019-05-29 07:28:59
updated: 2019-05-29 07:28:59
---

# 概述

本文主要是针对 vue 项目的 webpack 打包文件大小优化相关讲解。[配套测试源码](https://github.com/jovysun/Vue-my-pro)

<!-- more -->

# 详述

总的思路就是尽量减小打包文件大小，具体方法就是按需加载，也就是更小颗粒度的去引用需要的功能模块。

## 基础优化

一，组件库按需引入组件：

```js
// src/main.js
// babel-plugin-import 会帮助你加载 JS 和 CSS
import {
  Layout,
  Menu,
  ...
} from "ant-design-vue";
```

二，路由中使用“webpackChunkName”:

```js
// src/router.js
...
{
  path: "/user/login",
  name: "login",
  component: () =>
    import(/* webpackChunkName: "user" */ "./views/User/Login.vue")
},
...
```

三，第三方库按需打包
如 lodash，可以直接引入具体方法，也可以通过 webpack 插件来按需打包：

`import debounce from "lodash/debounce"`或者`import { debounce } from "lodash"`+ `babel-plugin-lodash lodash-webpack-plugin`

## 分析打包报告并优化

### 导出报告

```shell
npm run build -- --report
```

示例图如下：
![初始报告图](report.jpg)

根据该分析报告，可以看出比较大的模块有@ant-design、moment 和 echarts。本次也将以此为例讲解。

### @ant-design 优化

这部分最大的优化点就是 icons 部分，本次讲下社区提供的[优化方案](https://github.com/HeskeyBaozi/reduce-antd-icons-bundle-demo)。

创建 `src/icons.js`

```js
// 列举你需要的图标
export {
  default as MenuFoldOutline
} from "@ant-design/icons/lib/outline/MenuFoldOutline";

export {
  default as MenuUnfoldOutline
} from "@ant-design/icons/lib/outline/MenuUnfoldOutline";
```

配置 `vue.config.js`

```js
configureWebpack: {
  resolve: {
    alias: {
      "@ant-design/icons/lib/dist$": path.resolve(__dirname, "./src/icons.js")
    }
  }
},
```

该方案有个缺点就是你得列举所有用到的图标，包括组件内用到的，目前没发现其他更好的方案。

### moment 优化

针对 locale 语言包部分，本次也以社区提供的[优化方案](https://github.com/jmblog/how-to-optimize-momentjs-with-webpack)为例讲下。

整体思路为两种，一种是先忽略整体语言包，然后业务层面引入特定的语言包文件；另一种是直接指定打包哪些语言文件。主要会用到 webpack 的两个内置插件 `IgnorePlugin` 和 `ContextReplacementPlugin`。

一，IgnorePlugin 的使用

移除所有语言包

```js
// vue.config.js
const webpack = require("webpack");
module.exports = {
  //...
  configureWebpack: {
    plugins: [
      // Ignore all locale files of moment.js
      new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/)
    ]
  }
};
```

在需要的地方引入需要的语言包

```html
<!-- src/App.vue -->
<script>
  import moment from "moment";
  import "moment/locale/zh-cn";
  export default {
    watch: {
      "$route.query.locale": function(val) {
        moment.locale(val === "en_US" ? "en" : "zh-cn");
      }
    }
  };
</script>
```

二，ContextReplacementPlugin 的使用

```js
const webpack = require("webpack");
module.exports = {
  //...
  configureWebpack: {
    plugins: [
      // load `moment/locale/zh-cn.js`
      new webpack.ContextReplacementPlugin(/moment[/\\]locale$/, /zh-cn/)
    ]
  }
};
```

这样在你需要的地方就不用单独引入语言包了

```html
<!-- src/App.vue -->
<script>
  import moment from "moment";
  // import "moment/locale/zh-cn";
  export default {
    watch: {
      "$route.query.locale": function(val) {
        moment.locale(val === "en_US" ? "en" : "zh-cn");
      }
    }
  };
</script>
```

### echarts 优化

由整体引入 echarts 包改成按需引入。

```js
// 整体引入：
import echarts from "echarts";
```

```js
// 按需引入：
// 核心功能包
import echarts from "echarts/lib/echarts";
// 用到的柱状图功能包
import "echarts/lib/chart/bar";
// 标题功能包
import "echarts/lib/component/title";
```

经过以上三个模块的优化之后，报告图如下：
![优化后报告图](report1.jpg)
优化前后大小对比：
![优化前](size0.jpg)
![优化后](size1.jpg)

## webpack 性能优化——DLL

https://www.cnblogs.com/ghost-xyx/p/6472578.html

# 参考

极客时间——Vue 开发实战
