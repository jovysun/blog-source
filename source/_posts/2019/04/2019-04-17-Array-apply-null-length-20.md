---
title: '深入理解Array.apply(null, {length: 20})'
tags:
  - JavaScript
categories:
  - 前端
date: 2019-04-17 20:38:21
updated: 2019-04-17 20:38:21
---

# 概述
学习Vue[渲染函数](https://cn.vuejs.org/v2/guide/render-function.html)章节，有段说明是用渲染函数渲染20个相同的段落：
```js
render: function (createElement) {
  return createElement('div',
    Array.apply(null, { length: 20 }).map(function () {
      return createElement('p', 'hi')
    })
  )
}
```
对于`Array.apply(null, { length: 20 })`的用法有些费解，于是查资料研究了下。
<!-- more -->

# 详述

## 核心知识点
1. 构造函数也可以像普通函数一样调用，`Array(1, 2, 3)`结果为`[1, 2, 3]`；
2. apply方法的第一个参数为null或者undefined等价于全局对象（非严格模式）；
3. apply方法的第二个参数`{ length: 20 }`为类数组；
4. 浏览器环境下等价于`window.Array(undefined, undefined, undefined, ...)`。

## 深究apply
apply()方法调用一个具有给定this值的函数，以及作为一个数组（或类似数组对象）提供的参数。

语法为`func.apply(thisArg, [argsArray])`。

thisArg：如果这个函数处于非严格模式下，则指定为 null 或 undefined 时会自动替换为指向全局对象。浏览器环境下，这个值为null、undefined和window是等价的，测试结果如下：

![示例图](3.jpg)

argsArray：一个数组或者类数组对象，这里是类数组，关于类数组，[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)有这样一段描述：

> 从 ECMAScript 第5版开始，可以使用任何种类的类数组对象，就是说只要有一个 length 属性和(0..length-1)范围的整数属性。例如现在可以使用 NodeList 或一个自己定义的类似 {'length': 2, '0': 'eat', '1': 'bananas'} 形式的对象。

测试结果如下：

![示例图](4.jpg)

## `Array.apply(null, { length: 20 })`与`Array(20)`
两者的测试结果如下：

![示例图一](1.jpg)

从结果可以看出前者是声明了一个长度为20的数组，并且初始化每个值为`undefined`；后者只是声明了一个长度为20的数组，值为`empty`，等价于`new Array(20)`。

而[map()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map)方法是针对数组中的每个**元素**进行迭代执行函数，最终返回一个新的数组，既然后者没有元素，map也就无意义了，测试结果如下：

![示例图二](2.jpg)