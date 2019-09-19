---
title: Vue实战@ant-disign第一坑
permalink: vue-in-action-antd
tags:
  - Vue
categories:
  - 框架与库
  - Vue
date: 2019-05-20 10:25:48
updated: 2019-05-20 10:25:48
---

# 概述

第一次用 ant-design-vue，刚启动就踩坑，记录下。[配套测试源码](https://github.com/jovysun/Vue-my-pro)

<!-- more -->

# 详述

vue-cli3.x 创建项目

按需引用 ant-design-vue 组件

```shell
npm install ant-design-vue --save
npm install babel-plugin-import --save-dev
```

启动项目报错如下：
![](1.jpg)
项目刚开始就报错，真是让人不愉快，但是不用着急，仔细看下错误提示信息，发现有这样一句`https://github.com/ant-design/ant-motion/issues/44`，点开找下点赞比较多的，主要有两条思路：
一，降低 less 版本；
二，配置 less-loader 的 javascriptEnabled 为 true，如下：
![](2.jpg)

我这里是采用了方案二，创建一个 vue.config.js 然后加上配置参数，运行 OK。

# 参考

极客时间——Vue 开发实战
