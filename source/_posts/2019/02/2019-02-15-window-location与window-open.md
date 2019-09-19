---
title: window.location与window.open()
tags:
  - JavaScript
categories:
  - 前端
date: 2019-02-15 14:20:19
updated: 2019-02-15 14:20:19
---

# 概述
只对两者打开页面的常用场景作个总结，至于详细知识点可以看[JavaScript 高级程序设计 第 8 章](/2019/01/10/JS高设3版-8/)。
<!-- more -->

# 详述

## 基本用法

```js
window.location.href = "//www.sogou.com";
window.open("//www.sogou.com");
```

前者是在当前窗口打开页面，浏览器历史记录中会生成一条记录，因此可以“后退”到前一个页面。
后者是在新的窗口打开页面，有些浏览器默认会拦截，如图：
![图一](1.gif)

## 其他常用

```js
parent.location.href = "//www.sogou.com";
top.location.href = "//www.sogou.com";
// 当前窗口打开
window.open("//www.sogou.com", "_self");
window.open("//www.sogou.com", "_top");
```

## window.open()屏蔽检测

```js
var blocked = false;
try {
  var wroxWin = window.open("//www.sogou.com");
  if (wroxWin == null) {
    blocked = true;
  }
} catch (ex) {
  blocked = true;
}
if (blocked) {
  alert("The popup was blocked!");
}
```

# 后记
