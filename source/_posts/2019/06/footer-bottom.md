---
title: 页脚居底最简单实现
permalink: footer-bottom
tags:
  - CSS
categories:
  - 前端
date: 2019-06-28 15:57:22
updated: 2019-06-28 15:57:22
---

# 概述

仅用简单的 css 实现页脚始终居于页面底部。主要知识点为`min-height:100%`和`box-sizing:border-box`。

<!-- more -->

# 详述

## 直接看代码：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>页脚居底最简单实现</title>
    <style>
      /* reset */
      body {
        margin: 0;
        padding: 0;
      }
      header {
        height: 80px;
        background-color: antiquewhite;
      }
      footer {
        height: 60px;
        background-color: brown;
      }

      /* 核心实现部分 */
      html {
        height: 100%;
      }
      body {
        position: relative;
        min-height: 100%;
        box-sizing: border-box; /* 设置高度包括padding */
        padding-bottom: 60px; /* footer 的高度，为 footer 占位 */
      }
      footer {
        position: absolute;
        bottom: 0;
        left: 0;
        width: 100%;
        height: 60px;
      }
    </style>
  </head>
  <body>
    <header>this is header content</header>
    <section>this is main content</section>
    <footer>this is footer content</footer>
  </body>
</html>
```

## 注意点一

设置 body 的`min-height: 100%;`需要设置 html 的高度为 100%。

## 注意点二

CSS 中的 box-sizing 属性定义了宿主环境应该如何计算一个元素的总宽度和总高度。

在 CSS 盒子模型的默认定义里，你对一个元素所设置的 width 与 height 只会应用到这个元素的内容区。如果这个元素有任何的 border 或 padding ，绘制到屏幕上时的盒子宽度和高度会加上设置的边框和内边距值。这意味着当你调整一个元素的宽度和高度时需要时刻注意到这个元素的边框和内边距。当我们实现响应式布局时，这个特点尤其烦人。

box-sizing 属性可以被用来调整这些表现:

- content-box 是默认值。如果你设置一个元素的宽为 100px，那么这个元素的内容区会有 100px 宽，并且任何边框和内边距的宽度都会被增加到最后绘制出来的元素宽度中。
- border-box 告诉浏览器：你想要设置的边框和内边距的值是包含在 width 内的。也就是说，如果你将一个元素的 width 设为 100px，那么这 100px 会包含它的 border 和 padding，内容区的实际宽度是 width 减去(border + padding)的值。大多数情况下，这使得我们更容易地设定一个元素的宽高。

# 参考

https://developer.mozilla.org/zh-CN/docs/Web/CSS/box-sizing
