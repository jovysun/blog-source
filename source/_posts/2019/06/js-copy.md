---
title: js复制文本到剪贴板
permalink: js-copy
tags:
  - JavaScript
categories:
  - 其他
date: 2019-06-28 14:15:47
updated: 2019-06-28 14:15:47
---

# 概述

实现点击按钮复制指定的内容到黏贴板，主要涉及两个 API`element.select()`和`document.execCommand()`。本文将介绍基本实现及常见坑，最后是推荐一个不错的第三方库。
![示例图](demo.jpg)

<!-- more -->

# 详述

实际场景是：系统要提供一个批量上传商品的功能，但是上传的 Excel 文件中需要制定商品属于哪个类目，因此需要提供一个交互是用户选择类目然后可点击按钮复制类目码。示例图如上。

## select 方法

select() 方法用于选择该元素中的文本。

### 语法

```js
textareaObject.select();
```

### 注意点

该元素为可编辑文本的元素，如 textarea、input([type=text])，因此常见坑有：

1. 该元素设置了 hidden 属性；
2. 该元素设置了 disabled 属性；
3. 该元素为`<input type="hidden">`；
4. input 元素的 width 或者 height 为 0；

## execCommand 方法

当一个 HTML 文档切换到设计模式时，document 暴露 execCommand 方法，该方法允许运行命令来操纵可编辑内容区域的元素。

### 语法

```js
bool = document.execCommand(aCommandName, aShowDefaultUI, aValueArgument);
```

aCommandName：一个 DOMString ，命令的名称。可用命令列表请参阅 [命令](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/execCommand#%E5%91%BD%E4%BB%A4) 。

aShowDefaultUI：一个 Boolean， 是否展示用户界面，一般为 false。Mozilla 没有实现。

aValueArgument：一些命令（例如 insertImage）需要额外的参数（insertImage 需要提供插入 image 的 url），默认为 null。

## 实现点击复制示例

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>js复制</title>
  </head>
  <body>
    <textarea id="text" style="width: 0">hello JovySun</textarea>
    <!-- <input type="text" value="hello JovySun" id="text" hidden> -->
    <button id="btn">click</button>
    <script>
      window.onload = function(event) {
        var button = document.getElementById("btn");
        button.onclick = function(event) {
          document.getElementById("text").select();
          document.execCommand("copy", false, null);
        };
      };
    </script>
  </body>
</html>
```

## 第三方库

[clipboardjs](https://clipboardjs.com/)，一种将文本复制到剪贴板的现代方法，无其他依赖，gzipe 压缩后只有 3kb。

提供从其他元素复制，剪切文本，或者从自身属性复制文本的方法，另外也提供了 success 和 error 自定义事件方便你作进一步的交互反馈。

浏览器支持 IE9+及其他所有主流现代浏览器。
