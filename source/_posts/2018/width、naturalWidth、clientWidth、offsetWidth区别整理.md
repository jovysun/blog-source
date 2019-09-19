---
title: width、naturalWidth、clientWidth、offsetWidth区别整理
date: 2018-04-08 20:24:31
tags:
  - HTML
categories:
  - 前端
---
今天在做图片裁剪功能的时候，参考了下网友的资料，发现大家对图片宽度的获取方式不尽相同，于是详细整理下各个属性的区别（详细请参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API)）。

1，`HTMLImageElement.width`是一个`unsigned long` 类型反映了 [`width` ](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/img#attr-width)HTML 属性, 说明图像在CSS像素中渲染的宽度。

2，`HTMLImageElement.naturalWidth`返回一个 `unsigned long` 类型,表明图像在CSS像素中固有的宽度,如果可用的话； 否则, 返回`0`。

3，`Element.clientWidth` 属性表示元素的内部宽度，以像素计。该属性包括内边距，但不包括垂直滚动条（如果有）、边框和外边距。该属性值会被四舍五入为一个整数。如果你需要一个小数值，可使用 [`element.getBoundingClientRect()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect "Element.getBoundingClientRect()方法返回元素的大小及其相对于视口的位置。")。
![clientWidth](clientWidth.png)

4，`HTMLElement.offsetWidth` 是一个只读属性，返回一个元素的布局宽度。一个典型的（译者注：各浏览器的offsetWidth可能有所不同）offsetWidth是测量包含元素的边框(border)、水平线上的内边距(padding)、竖直方向滚动条(scrollbar)（如果存在的话）、以及CSS设置的宽度(width)的值。
![offsetWidth](offsetWidth.png)