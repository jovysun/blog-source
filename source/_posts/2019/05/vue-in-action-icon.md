---
title: Vue实战@管理系统中使用的图标
permalink: vue-in-action-icon
tags:
  - Vue
categories:
  - 框架与库
  - Vue
date: 2019-05-26 11:23:28
updated: 2019-05-26 11:23:28
---

# 概述

日常开发中，除了直接使用 UI 组件库提供有限图标外，怎么使用其他图标呢？目前主要有三种方式，雪碧图、字体图标和 svg 图标。雪碧图方式古老，字体图标方式流行，但是不支持多色图标，svg 方式逐渐流行，支持多色图标。本文主要讲下字体图标和 svg 图标的使用。[配套测试源码](https://github.com/jovysun/Vue-my-pro)
![测试效果图](overview.jpg)

<!-- more -->

# 详述

本次以阿里的图标开源库 [iconfont.cn](https://www.iconfont.cn)为例。

## 出发

平台注册账号，搜索需要图标的关键字，例如“404”，找到自己需要的可以收藏、加入购物车、添加到项目等。把需要的图标添加到自己创建的项目后，就可以在项目面板页管理和使用图标了。
![项目面板截图](iconfont.jpg)

## unicode 引用

unicode 是字体在网页端最原始的应用方式，特点是：

- 兼容性最好，支持 ie6+，及所有现代浏览器。
- 支持按字体的方式去动态调整图标大小，颜色等等。
- 但是因为是字体，所以不支持多色。只能使用平台里单色的图标，就算项目里有多色图标也会自动去色。

unicode 使用步骤如下：

第一步：拷贝项目下面生成的 font-face

```css
@font-face {
  font-family: "iconfont";
  src: url("iconfont.eot");
  src: url("iconfont.eot?#iefix") format("embedded-opentype"), url("iconfont.woff")
      format("woff"), url("iconfont.ttf") format("truetype"), url("iconfont.svg#iconfont")
      format("svg");
}
```

第二步：定义使用 iconfont 的样式

```css
.iconfont {
  font-family: "iconfont" !important;
  font-size: 16px;
  font-style: normal;
  -webkit-font-smoothing: antialiased;
  -webkit-text-stroke-width: 0.2px;
  -moz-osx-font-smoothing: grayscale;
}
```

第三步：挑选相应图标并获取字体编码，应用于页面

```html
<i class="iconfont">&#x33;</i>
```

vue 使用示例如下：

```html
<template>
  <div>
    <i class="iconfont">&#xe61c;</i>
  </div>
</template>

<style scoped>
  @font-face {
    font-family: "iconfont"; /* project id 1212356 */
    src: url("//at.alicdn.com/t/font_1212356_01bv0xpc70ab.eot");
    src: url("//at.alicdn.com/t/font_1212356_01bv0xpc70ab.eot?#iefix") format("embedded-opentype"),
      url("//at.alicdn.com/t/font_1212356_01bv0xpc70ab.woff2") format("woff2"),
      url("//at.alicdn.com/t/font_1212356_01bv0xpc70ab.woff") format("woff"), url("//at.alicdn.com/t/font_1212356_01bv0xpc70ab.ttf")
        format("truetype"),
      url("//at.alicdn.com/t/font_1212356_01bv0xpc70ab.svg#iconfont") format("svg");
  }
  .iconfont {
    font-family: "iconfont" !important;
    font-size: 16px;
    font-style: normal;
    -webkit-font-smoothing: antialiased;
    -webkit-text-stroke-width: 0.2px;
    -moz-osx-font-smoothing: grayscale;
  }
</style>
```

## font-class 引用

font-class 是 unicode 使用方式的一种变种，主要是解决 unicode 书写不直观，语意不明确的问题。

与 unicode 使用方式相比，具有如下特点：

- 兼容性良好，支持 ie8+，及所有现代浏览器。
- 相比于 unicode 语意明确，书写更直观。可以很容易分辨这个 icon 是什么。
- 因为使用 class 来定义图标，所以当要替换图标时，只需要修改 class 里面的 unicode 引用。
- 不过因为本质上还是使用的字体，所以多色图标还是不支持的。

使用步骤如下：

第一步：拷贝项目下面生成的 fontclass 代码：

```css
//at.alicdn.com/t/font_8d5l8fzk5b87iudi.css
```

第二步：挑选相应图标并获取类名，应用于页面：

```html
<i class="iconfont icon-xxx"></i>
```

vue 使用示例如下：

```html
<template>
  <div>
    <i class="iconfont iconyunshu"></i>
  </div>
</template>
<style scoped>
  @import url(//at.alicdn.com/t/font_1212356_mw76lv1oaul.css);
</style>
```

## symbol 引用

这是一种全新的使用方式，应该说这才是未来的主流，也是平台目前推荐的用法。相关介绍可以参考这篇文章 这种用法其实是做了一个 svg 的集合，与上面两种相比具有如下特点：

- 支持多色图标了，不再受单色限制。
- 通过一些技巧，支持像字体那样，通过 font-size,color 来调整样式。
- 兼容性较差，支持 ie9+,及现代浏览器。
- 浏览器渲染 svg 的性能一般，还不如 png。

第一步：拷贝项目下面生成的 symbol 代码：

```js
//at.alicdn.com/t/font_8d5l8fzk5b87iudi.js
```

第二步：加入通用 css 代码（引入一次就行）：

```html
<style type="text/css">
  .icon {
    width: 1em;
    height: 1em;
    vertical-align: -0.15em;
    fill: currentColor;
    overflow: hidden;
  }
</style>
```

第三步：挑选相应图标并获取类名，应用于页面：

```html
<svg class="icon" aria-hidden="true">
  <use xlink:href="#icon-xxx"></use>
</svg>
```

普通页面测试代码：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Document</title>
    <script src="http://at.alicdn.com/t/font_1212356_01bv0xpc70ab.js"></script>
    <style type="text/css">
      .icon {
        width: 1em;
        height: 1em;
        vertical-align: -0.15em;
        fill: currentColor;
        overflow: hidden;
      }
    </style>
  </head>
  <body>
    <svg class="icon" aria-hidden="true">
      <use xlink:href="#icontixingshixiang"></use>
    </svg>
  </body>
</html>
```

`.vue`单文件开发中，因为是模块化开发，是通过 webpack 编译打包的，因此第一步略有不同：保存该 js 文件到本地，在`main.js`中引入`import "./assets/iconfont.js";`。

## ant-design-vue 中使用

```html
<template>
  <div class="icons-list">
    <icon-font type="icon-tuichu" />
    <icon-font type="icon-facebook" />
    <icon-font type="icon-twitter" />
  </div>
</template>
<script>
  import { Icon } from "ant-design-vue";

  const IconFont = Icon.createFromIconfontCN({
    scriptUrl: "//at.alicdn.com/t/font_8d5l8fzk5b87iudi.js"
  });
  export default {
    components: {
      IconFont
    }
  };
</script>
<style scoped>
  .icons-list >>> .anticon {
    margin-right: 6px;
    font-size: 24px;
  }
</style>
```

示例是局部注册，当然对于通用的图标也可以 main.js 中进行全局注册。

## 直接使用 svg 图标文件

第一步：下载 svg 文件，保存到本地；

第二步：安装 loader:

```shell
npm install vue-svg-loader -D
```

第三步：组件中引入 svg 文件，注册组件：

```js
import Logo from "@/assets/logo.svg";
export default {
  components: {
    Logo
  }
};
```

第四步：使用该图标组件：

```html
<template>
  <div>
    <Logo />
  </div>
</template>
```

# 参考

[IconFont](https://www.iconfont.cn/help/detail?spm=a313x.7781069.1998910419.d8cf4382a&helptype=code)

极客时间——Vue 开发实战
