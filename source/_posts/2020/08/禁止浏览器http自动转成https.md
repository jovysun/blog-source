---
title: 禁止浏览器http自动转成https
permalink: ban-browser-http-to-https
tags:
  - HTML
  - CSS
  - JavaScript
categories:
  - 其他
date: 2020-08-18 17:07:12
updated: 2020-08-18 17:07:12
---

# 概述

主要是 chrome 浏览器，之前尝试重启，清缓存都不行，这个方法试了下可以。

<!-- more -->

# 详述

## Chrome 浏览器

1. 地址栏中输入`chrome://net-internals/#hsts`
2. 在 Delete domain security policies 中输入项目的域名（不是包含协议的完整地址），并 Delete 删除
3. 可以在 Query domain 测试是否删除成功
4. 这里如果还是不行，请清除浏览器缓存！

## Safari 浏览器

1. 完全关闭 Safari
2. 删除 ~/Library/Cookies/HSTS.plist 这个文件
3. 重新打开 Safari 即可
4. 极少数情况下，需要重启系统

## Opera 浏览器

和 Chrome 方法一样

## Firefox 浏览器

关闭所有已打开的页面
清空历史记录和缓存

# 参考

https://zhuanlan.zhihu.com/p/138518884
