---
title: Vue实战@使用mock数据
permalink: vue-in-action-mock
tags:
  - Vue
categories:
  - 框架与库
  - Vue
date: 2019-05-22 19:35:06
updated: 2019-05-22 19:35:06
---

# 概述

在开发中使用 mock 数据有好多种方式，大点公司自己搭建接口平台系统，其他的可以使用 Easy Mock、Mock.js 等。在 webpack 工程项目下可以利用 webpack-dev-server 搭建一个简单快速灵活的 mock 服务。[配套测试源码](https://github.com/jovysun/Vue-my-pro)

<!-- more -->

# 详述

## 创建 mock 数据文件

项目根目录创建 mock 文件夹来存放 mock 数据文件，然后根据业务创建对应的 mock 文件，例如 dashboard_chart.js，内容如下：

```js
function chart(method) {
  let res = null;
  switch (method) {
    case "GET":
      res = [120, 30, 40, 60, 20, 45];
      break;

    default:
      res = null;
      break;
  }
  return res;
}
// 因为node端使用，因此用CMD规范，而不是ES6
module.exports = chart;
```

## 配置 webpack-dev-server

本示例是 vue-cli 创建的项目，对应的就是在 vue.config.js 中配置 devServer，示例如下：

```js
module.exports = {
  devServer: {
    port: 3000,
    proxy: {
      "/api": {
        target: "http://localhost:3000",
        bypass: function(req, res) {
          if (req.headers.accept.indexOf("html") !== -1) {
            console.log("Skipping proxy for browser request.");
            return "/index.html";
          } else if (process.env.MOCK !== "none") {
            //根据环境变量
            const name = req.path
              .split("/api/")[1]
              .split("/")
              .join("_");
            const chart = require(`./mock/${name}`);
            const result = chart(req.method);
            delete require.cache[require.resolve(`./mock/${name}`)];
            return res.send(result);
          }
        }
      }
    }
  }
};
```

## 数据请求

具体业务组件中的使用，axios 使用为例，如下：

```js
getChartData() {
  axios
    .get("/api/dashboard/chart", { params: { ID: 12345 } })
    .then(response => {
      this.chartOption = {
        title: {
          text: "ECharts 入门示例"
        },
        tooltip: {},
        xAxis: {
          data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
        },
        yAxis: {},
        series: [
          {
            name: "销量",
            type: "bar",
            data: response.data //请求的数据
          }
        ]
      };
    });
}
```

## 多环境判断

实际中，会面对多个环境，除了开发环境，还有测试环境，生产环境等。那么对于数据接口的调用也将是根据环境来区分。

本次示例：
安装 cross-env 实现跨平台，在 package.json 配置执行命令`"serve:no-mock": "cross-env MOCK=none vue-cli-service serve"`，然后在请求是判断是否用 mock 数据，示例如上。

# 参考

极客时间——Vue 开发实战
