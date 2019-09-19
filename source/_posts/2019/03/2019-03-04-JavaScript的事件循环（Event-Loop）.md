---
title: JavaScript的事件循环（Event Loop）
tags:
  - JavaScript
categories:
  - 前端
date: 2019-03-04 13:31:43
updated: 2019-03-04 13:31:43
---
# 前言
对于事件循环，这次将从浏览器端和node端分别讲解。
# 概述
JavaScript是一门单线程非阻塞的脚本语言。对于单线程，主要是因为当初这么语言设计的时候也是为了操作DOM，如果多线程就无法保证操作的顺序性。那么既然单线程了，为什么会非阻塞呢？这是因为事件循环的机制，这个机制处理了我们熟悉的浏览器端的异步事件回调。当然node是基于js的，也有自己的事件循环机制。
<!-- more -->
# 详述
## 浏览器端
浏览器端js是有专门的JS引擎来执行的，在执行JS的主线程中，遇到异步事件，会把回调函数放到执行栈中，直到主线程同步代码执行完，然后开始从执行栈中依次执行添加的回调函数。
那么这个异步任务有没有优先级呢？
有。这就是大家常说的宏任务（macro task）与微任务（micro task）。常见的如下：
宏任务：
```js
    setInterval()
    setTimeout()
```    
微任务
```js
    new Promise()
    new MutaionObserver()
```
其中微任务优先级高于宏任务，也就是先执行，可以帮上面说的执行栈再细分为微任务执行栈和宏任务执行栈。

## node端
node端的事件循环模型官方给出如下示例图：
![node事件循环](1.jpg)

具体各阶段任务为：

timers: 这个阶段执行定时器队列中的回调如 setTimeout() 和 setInterval()。
I/O callbacks: 这个阶段执行几乎所有的回调。但是不包括close事件，定时器和setImmediate()的回调。
idle, prepare: 这个阶段仅在内部使用，可以不必理会。
poll: 等待新的I/O事件，node在一些特殊情况下会阻塞在这里。
check: setImmediate()的回调会在这个阶段执行。
close callbacks: 例如socket.on('close', ...)这种close事件的回调。

总的循环顺序为：
首先是添加异步任务回调到poll中
=>执行完之后到check阶段，主要执行setImmediate()
=>然后执行close callbacks，主要是socket的close之类的回调
=>然后到timer阶段，主要是setTimeout() 和 setInterval()回调
=>I/O callbacks，执行除了模型中列出的阶段回调之外所有的
=>idle, prepare，内部清理，准备下次循环。

# 后记
参考：https://www.cnblogs.com/cangqinglang/p/8967268.html