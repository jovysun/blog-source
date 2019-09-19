---
title: JavaScript高阶函数常用场景
tags:
  - JavaScript
categories:
  - 前端
date: 2019-04-10 14:09:46
updated: 2019-04-10 14:09:46
---

# 概述

数组有一维到多维，函数也有低阶到高阶，高阶函数的直观表现形式为一个函数接收另一个函数作为参数。

<!-- more -->

# 详述

## 最简单的高阶函数示例

```js
function add(x, y, fn) {
  return f(x) + f(y);
}
```

## sort 排序

数组的 sort 方法默认是按照字典排序，很多时候并不适合我们的实际需求，例如：

```js
[2, 10, 3].sort()[
  // [10, 2, 3]

  ("Google", "apple", "Microsoft")
].sort();
// ['Google', 'Microsoft', 'apple']
```

另外实际中也经常会涉及到按照对象的某个属性排序，这时候就需要我们自己创建比较函数，然后作为参数传给 sort 方法。例如：

```js
//示例一
[2, 10, 3]
  .sort((x, y) => {
    return x - y;
  })
  [
    // [2, 3, 10]

    // 示例二
    ("Google", "apple", "Microsoft")
  ].sort(function(s1, s2) {
    var x1 = s1.toUpperCase();
    var x2 = s2.toUpperCase();
    if (x1 < x2) {
      return -1;
    } else if (x1 > x2) {
      return 1;
    } else {
      return 0;
    }
  });
// ["apple", "Google", "Microsoft"]

// 示例三
var arr = [
  {
    id: 1,
    name: "Jack"
  },
  {
    id: 3,
    name: "Tom"
  },
  {
    id: 2,
    name: "Alax"
  }
];

function createComparisonFunction(propertyName) {
  return function(object1, object2) {
    var value1 = object1[propertyName];
    var value2 = object2[propertyName];
    if (value1 < value2) {
      return -1;
    } else if (value1 > value2) {
      return 1;
    } else {
      return 0;
    }
  };
}
arr.sort(createComparisonFunction("id"));
// [{id: 1, name: "Jack"}, {id: 2, name: "Alax"}, {id: 3, name: "Tom"}]
```

## map/reduce
```js
function pow(x) {
    return x * x;
}
var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
arr.map(pow); 
// [1, 4, 9, 16, 25, 36, 49, 64, 81]


var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
arr.map(String); 
// ['1', '2', '3', '4', '5', '6', '7', '8', '9']


var arr = [1, 3, 5, 7, 9];
arr.reduce(function (x, y) {
    return x + y;
}); 
// 25
```

## filter
```js
//例如，在一个Array中，删掉偶数，只保留奇数，可以这么写：
var arr = [1, 2, 4, 5, 6, 9, 10, 15];
var r = arr.filter(function (x) {
    return x % 2 !== 0;
});
r; // [1, 5, 9, 15]


//把一个Array中的空字符串删掉，可以这么写：
var arr = ['A', '', 'B', null, undefined, 'C', '  '];
var r = arr.filter(function (s) {
    return s && s.trim(); // 注意：IE9以下的版本没有trim()方法
});
r; // ['A', 'B', 'C']


//利用filter，可以巧妙地去除Array的重复元素：
var arr = ['apple', 'strawberry', 'banana', 'pear', 'apple', 'orange', 'orange', 'strawberry'];
var r = arr.filter(function (element, index, self) {
    return self.indexOf(element) === index;
});
r; // ["apple", "strawberry", "banana", "pear", "orange"]
//去除重复元素依靠的是indexOf总是返回第一个元素的位置，后续的重复元素位置与indexOf返回的位置不相等，因此被filter滤掉了。
```
# 参考
http://www.cnblogs.com/goloving/p/8361705.html