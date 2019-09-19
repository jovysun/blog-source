---
title: apply,call,bind--知识总结及实践案例
tags:
  - JavaScript
categories:
  - 前端
date: 2019-01-31 10:56:39
updated: 2019-01-31 10:56:39
---
# 前言
apply和call经常使用，今天突然看到bind方法，不熟悉，于是把三者整体梳理学习下。

# 概述
三者都是为了改变函数的作用域（上下文context），也就是改变函数中this的指向，同时可以附加参数。但是三者的具体使用有什么异同呢？

<!-- more -->
# 详述

## 语法区别
语法：
```js
func.apply(thisArg[, argsArray])

func.call(thisArg[, arg1[, arg2[, ...]]])
s
var otherFunc = func.bind(thisArg[, arg1[, arg2[, ...]]]);
otherFunc();
```
区别：
1. apply与call类似，区别在于apply第二个参数是一个参数数组或者arguments对象，call从第二个参数开始，罗列一个个参数。（对于记忆，我的方法是apply+arguments，都是a开头，这样就不会混淆两者的传参形式了。）
2. bind是创建一个新的函数，参数形式与call一样。
3. apply/call与bind的区别是apply/call是在执行函数函数指定this指向，bind是先创建一个改变了this指向的新函数，然后再调用执行。

## 使用场景
### 检测数组

```js
if(Object.prototype.toString.call(obj) === '[object Array]'){
  //执行数组操作
}
```
当然，检测数组，同一个全局变量下可以用a`rr instanceof Array`，有框架结构就不行了，另外也可以用ES5的`Array.isArray(arr)`;

### 类数组转成数组
常用类数组对象有arguments,NodeList（包括类NodeList的元素集合）。
```js
Array.prototype.slice.call(arguments)
```

### 继承
```js
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function() {
  alert(this.name);
};
function SubType(name, age) {
  //继承属性
  SuperType.call(this, name);
  this.age = age;
}
//继承方法
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function() {
  alert(this.age);
};
```
### 事件绑定
```js
var foo = {
    bar : 1,
    eventBind: function(){
        $('.someClass').on('click',function(event) {
            /* Act on the event */
            console.log(this.bar);      //1
        }.bind(this));
    }
}

```
### setTimeout
```js
function LateBloomer() {
  this.petalCount = Math.ceil(Math.random() * 12) + 1;
}

// 在 1 秒钟后声明 bloom
LateBloomer.prototype.bloom = function() {
  setTimeout(this.declare.bind(this), 1000);
};

LateBloomer.prototype.declare = function() {
  console.log('I am a beautiful flower with ' +
    this.petalCount + ' petals!');
};

var flower = new LateBloomer();
flower.bloom();  // 一秒钟后, 调用'declare'方法
```
# 后记