---
title: babel7.x使用及新特性测试
tags:
  - 工具
categories:
  - 前端
date: 2019-04-25 16:01:20
updated: 2019-04-25 16:01:20
---

# 概述
Babel是一个JavaScript编译器，主要用于将ES6+代码转换为当前和旧浏览器或其他环境中向后兼容的JavaScript版本。本篇主要讲下babel7.x的基本用法及新特性测试，[测试源码](https://github.com/jovysun/babel7.x-test)。
<!-- more -->

# 详述

## 基本用法

### 常用包介绍
```shell
$ npm install @babel/core @babel/preset-env @babel/polyfill @babel/runtime @babel/plugin-transform-runtime -D
```
从7.x开始，包名改用命名空间方式，例如不再是`babel-core`而是`@babel/core`；

`@babel/core` 核心工具包，配合插件包可以转换特定语法，例如加上`@babel/plugin-transform-arrow-functions`插件包，可以转换箭头函数语法；

`@babel/preset-env` 智能预置插件包，根据配置参数去自动选择对应插件转换。与第二条说的直接安装对应语法的插件包相比，这个更符合实际场景，通常大家关心的是在什么环境下能运行而不是暴力的直接把高级语法转成低级语法。例如箭头函数在最新chrome下是支持的，在IE11下不支持。如果只要求chrome下使用，那么就没必要转换箭头函数了，还有node环境，如果对应node版本支持的高级语法就没必要转换了。

![IE11](1.jpg) ![chrome](2.jpg)

[更多介绍](https://babeljs.io/docs/en/babel-preset-env)

`@babel/polyfill` 与preset-env区别是，preset-env是高级语法形式转成对应低级语法形式，基本是针对新的语法糖，例如箭头函数转成普通function函数；而polyfill是针对新增加的接口方法用另一种当前环境能支持的方式去实现，例如`findIndex`方法，ES6新加的，ES5没有对应的，想在ES5环境使用只能用polyfill方式。[MDN实现](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex)。

webpack打包示例如图：
```js
/***/ "./node_modules/core-js/modules/es6.array.find-index.js":
/*!**************************************************************!*\
!*** ./node_modules/core-js/modules/es6.array.find-index.js ***!
\**************************************************************/
/*! no static exports found */
/***/ (function(module, exports, __webpack_require__) {

"use strict";
eval("\n// 22.1.3.9 Array.prototype.findIndex(predicate, thisArg = undefined)\nvar $export = __webpack_require__(/*! ./_export */ \"./node_modules/core-js/modules/_export.js\");\nvar $find = __webpack_require__(/*! ./_array-methods */ \"./node_modules/core-js/modules/_array-methods.js\")(6);\nvar KEY = 'findIndex';\nvar forced = true;\n// Shouldn't skip holes\nif (KEY in []) Array(1)[KEY](function () { forced = false; });\n$export($export.P + $export.F * forced, 'Array', {\n  findIndex: function findIndex(callbackfn /* , that = undefined */) {\n    return $find(this, callbackfn, arguments.length > 1 ? arguments[1] : undefined);\n  }\n});\n__webpack_require__(/*! ./_add-to-unscopables */ \"./node_modules/core-js/modules/_add-to-unscopables.js\")(KEY);\n\n\n//# sourceURL=webpack:///./node_modules/core-js/modules/es6.array.find-index.js?");

/***/ }),
```
[更多介绍](https://babeljs.io/docs/en/babel-polyfill)

`@babel/runtime` 提供统一的模块化的helper，例如编译后的文件中_classCallCheck，_defineProperties，_createClass等函数就是helper。可以类比为我们日常开发中用到的工具函数，这些是可复用的，因此在运行环境最好提取出来。
`@babel/plugin-transform-runtime` runtime辅助插件，自动把runtime提取的helper引入到代码中。


## 配置
Babel有两种并行的配置文件格式，可以一起使用，也可以单独使用。

### 项目范围的配置（全局配置）
在最新的`Babel 7.x`，Babel有一个“根”目录的概念，默认为当前工作目录。对于项目范围的配置，Babel将自动搜索`babel.config.js`。这个作为默认配置，可以配置一些全局通用的规则，例如针对node_modules或、引用包文件等。

### 文件相对位置配置（局部配置）
Babel通过从正在编译的[filename](https://babeljs.io/docs/en/options#filename)开始搜索目录结构来加载.babelrc(和.babelrc.js / package.json# Babel)文件(受下面的警告限制)。这非常强大，因为它允许您为包的子部分创建独立的配置。局部配置优先级高于全局配置，因此最终结果是合并与覆盖。

局部配置边缘情况处理规则：
1. 一旦包含的目录搜索到package.json即停止搜索，因此相对配置只应用于单个包中。
2. 正在编译的[filename](https://babeljs.io/docs/en/options#filename)必须在`babelrcRoots`配置项中，否则将完全跳过搜索。
简言之，.babelrc文件只作用于自己的package中，根目录下的.babelrc文件如果没有在babelrcRoots中配置将被忽略。


### useBuiltIns
"usage" | "entry" | false, defaults to false.

"entry" 指定入口文件中引入`import "@babel/polyfill";`，编译后的polyfill代码统一加入该入口文件，示例如下：
In
```JavaScript
import "@babel/polyfill";
```
Out (different based on environment)
```JavaScript
import "core-js/modules/es7.string.pad-start";
import "core-js/modules/es7.string.pad-end";
```
webpack示例：
In
```JavaScript
entry: ['@babel/polyfill', path.join(__dirname, './src/index.js')],
```
```js
/***/ "./node_modules/core-js/modules/es7.string.pad-end.js":
/*!************************************************************!*\
  !*** ./node_modules/core-js/modules/es7.string.pad-end.js ***!
  \************************************************************/
/*! no static exports found */
/***/ (function(module, exports, __webpack_require__) {

"use strict";
eval("\n// https://github.com/tc39/proposal-string-pad-start-end\nvar $export = __webpack_require__(/*! ./_export */ \"./node_modules/core-js/modules/_export.js\");\nvar $pad = __webpack_require__(/*! ./_string-pad */ \"./node_modules/core-js/modules/_string-pad.js\");\nvar userAgent = __webpack_require__(/*! ./_user-agent */ \"./node_modules/core-js/modules/_user-agent.js\");\n\n// https://github.com/zloirock/core-js/issues/280\nvar WEBKIT_BUG = /Version\\/10\\.\\d+(\\.\\d+)?( Mobile\\/\\w+)? Safari\\//.test(userAgent);\n\n$export($export.P + $export.F * WEBKIT_BUG, 'String', {\n  padEnd: function padEnd(maxLength /* , fillString = ' ' */) {\n    return $pad(this, maxLength, arguments.length > 1 ? arguments[1] : undefined, false);\n  }\n});\n\n\n//# sourceURL=webpack:///./node_modules/core-js/modules/es7.string.pad-end.js?");

/***/ }),

/***/ "./node_modules/core-js/modules/es7.string.pad-start.js":
/*!**************************************************************!*\
  !*** ./node_modules/core-js/modules/es7.string.pad-start.js ***!
  \**************************************************************/
/*! no static exports found */
/***/ (function(module, exports, __webpack_require__) {

"use strict";
eval("\n// https://github.com/tc39/proposal-string-pad-start-end\nvar $export = __webpack_require__(/*! ./_export */ \"./node_modules/core-js/modules/_export.js\");\nvar $pad = __webpack_require__(/*! ./_string-pad */ \"./node_modules/core-js/modules/_string-pad.js\");\nvar userAgent = __webpack_require__(/*! ./_user-agent */ \"./node_modules/core-js/modules/_user-agent.js\");\n\n// https://github.com/zloirock/core-js/issues/280\nvar WEBKIT_BUG = /Version\\/10\\.\\d+(\\.\\d+)?( Mobile\\/\\w+)? Safari\\//.test(userAgent);\n\n$export($export.P + $export.F * WEBKIT_BUG, 'String', {\n  padStart: function padStart(maxLength /* , fillString = ' ' */) {\n    return $pad(this, maxLength, arguments.length > 1 ? arguments[1] : undefined, true);\n  }\n});\n\n\n//# sourceURL=webpack:///./node_modules/core-js/modules/es7.string.pad-start.js?");

/***/ }),
```

## Monorepos
简单理解就是一个总目录下，有多个package，不同的package下的可能有自己的babel配置文件。示例如下图：

![测试项目结构图](3.jpg)

过去的用法比较繁琐，babel7.x针对痛点做了相应的调整。现在的推荐用法是总目录下一个全局配置文件`babel.config.js`，然后各个子package下各一个配置文件`.babelrc.js(.babelrc)`。
至于父子级间配置文件怎么配合使用，主要体现在两个配置参数：[rootMode](https://babeljs.io/docs/en/options#rootmode)和[babelrcRoots](https://babeljs.io/docs/en/options#babelrcroots)。
### rootMode
设定子package搜索`babel.config.js`的规则

`root` 默认值，限定在当前目录。
`upward` 向上搜索，直到第一次出现`babel.config.js`为止，如果没搜到`babel.config.js`则抛出错误。
`upward-optional` 向上搜索，没找到则返回`root`设置值对应目录。

测试示例见mod1中的`package.json`，可以改下`--root-mode`的值执行`npm run babel`命令看编译后文件的变化。
```json
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "babel": "npx babel --root-mode upward src --out-dir lib"
  }
```
### babelrcRoots
设定根目录中编译子package中的文件是否可用子package的`.babelrc`文件。

测试步骤：
1. webpack配置entry
```js
entry: ['@babel/polyfill', path.join(__dirname, './src/index.js'), './packages/mod2/src/index.js'],
```
2. './packages/mod2'下的.babelrc.js文件增加一行打印代码
```js
console.log('from mod2 babelrc...............')
```
3. babel.config.js添加babelrcRoots配置项
```js
babelrcRoots: ['.', './packages/mod2']
```
4. 执行编译命令
```shell
$ npm run babel
```
测试结果如图：
![示例图4](4.jpg)

去掉babelrcRoots配置项则不会执行mod2下的.babelrc，如图：
![示例图5](5.jpg)

## 其他
配置方式：`package.json`、`.babelrc`、`.babelrc.js`、`babel.config.js`。

`.babelrc.js`与`.babelrc`用法基本一样，只是前者支持js代码。

# 参考
https://babeljs.io/docs/en/

https://www.jianshu.com/p/0ea6065cb39e

https://www.jianshu.com/p/cbd48919a0cc

https://segmentfault.com/a/1190000018358854