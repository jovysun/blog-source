---
title: 前端路由原理基础
permalink: 前端路由原理基础
tags:
  - JavaScript
categories:
  - 其他
date: 2019-05-22 15:17:09
updated: 2019-05-22 15:17:09
keywords:
description:
---

# 概述

对于当下组件化的 SPA 应用，前端路由是一个重要组成部分，不管是 react 还是 vue，都有各自的前端路由插件。那么前端路由的基础是什么呢？本文针对 hash 与 history 模式的实现基础做个总结。[测试源码](https://github.com/jovysun/my-router)
![测试效果预览图](my-router.gif)

<!-- more -->

# 详述

对于单页面应用，每个页面都是组件，页面切换即组件切换，前端路由要实现的就是页面的切换，不引起刷新，同时浏览器的 history 对象中都有记录，这样才能保证“前进”“后退”有效。

## hash 模式

hash 是 URL 中的 hash 部分，直观表现为#及其后面的部分，例如`#/dashboard`。通常通过 a 标签实现页面锚点定位。特点就是改变 hash 部分不会引起页面的刷新，但是却可以触发 hashchange 事件。因此可以通过这个特性实现前端路由。

## history 模式

HTML5 引入了 history.pushState() 和 history.replaceState() 方法，它们分别可以添加和修改历史记录条目。这些方法通常与 window.onpopstate 配合使用。因此也可以通过这个特性实现前端路由。[API 详细知识见 MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/History_API)

## 两种模式的优缺点

hash 模式兼容性好，简单，但是不美观。history 模式美观，但是不兼容 IE8 及以下，刷新会报错，需要后端配置处理。

## 简单实现

糙了一点，但是对于理解两种模式的路由基础实现应该可以了。

```js
window.addEventListener("DOMContentLoaded", readyHandler);
function readyHandler() {
  console.log("dom ready+++++++++++++++");

  let myContentDom = document.getElementById("myContent");

  // 路由
  function router() {
    let route = "";
    if (location.hash) {
      route = location.hash.split("#")[1];
    } else {
      route = location.pathname;
    }

    let component = null;
    switch (route) {
      case "/dashboard":
        component = { txt: "我是Dashboard组件" };
        break;
      case "/my":
        component = { txt: "我是My组件" };
        break;
      default:
        break;
    }
    renderDom(component);
  }
  // 渲染组件
  function renderDom(component) {
    myContentDom.innerHTML = component.txt;
  }

  // hash事件处理
  function hashchangeHandler() {
    router();
  }
  window.addEventListener("hashchange", hashchangeHandler);

  // history事件处理
  function popstateHandler() {
    router();
  }
  window.addEventListener("popstate", popstateHandler);

  document
    .getElementById("historyBlock")
    .addEventListener("click", function(e) {
      // 阻止链接点击默认跳转事件
      e.preventDefault();
      // 添加历史，实现浏览器前进后退
      history.pushState(null, "", e.target.href);
      popstateHandler();
    });
}
```
