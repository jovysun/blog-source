---
title: JS动画（二）——缓动函数分析及动画库
tags:
  - 动效
  - JavaScript
categories:
  - 前端
date: 2018-04-09 17:45:27
updated: 2018-08-07 08:36:16
---
上一篇讲了JS动画定时器相关知识，这一篇介绍下缓动函数及流行的动画库。
## 熟悉的图 ##

![clipboard.png](easingFunction.png)
## 实际使用 ##
jquery animate()+[jquery.easing插件][1]的使用：

```
$(selector).animate(styles,speed,easing,callback)
```
原生js使用：
[张鑫旭同学的文章][2]
## 缓动函数知识 ##
什么是缓动函数？我的理解是动画参数与数学公式结合的函数。

各流行库缓动函数对比，以easeInQuad为例，如图：

**[Tween.js][3]**

![clipboard.png](tweenjs.png)

**[jQuery.easing][4]**

![clipboard.png](jqueryEasing.png)

**[GSAP][5]**

![clipboard.png](gsap.png)

**[CreateJS][6]**

![clipboard.png](creatJs.png)

[Kute.js][7]
```
  easingFn.easingQuadraticIn = function (t) { return t*t; };
```
### 分析对比结果
基本数学公式是一样的，都是2次方；
缓动函数是独立的，与平台载体无关；
缓动函数反应的是动画进程与数值变化量的对应关系，具体分析如下：

[GSAP Ease在线示例][8]，动画进程每增加一格，数值变化量是增加量是越来越大的，效果就是由慢到快。
![clipboard.png](greensockEase.png)

与定时器无关，具体演变代码分析如下：
左侧演示的是，由于算法二次方，进程每次等量增加1/5，但是变化量却越来越大；右侧演示的是，虽然定时器改变了（间隔减小一倍，由“滴答”执行五次改成十次），但是变化量的趋势是一样的，相同的进程增量，对应的变化量也是相同。
![clipboard.png](yansuan.png)

## 动画库

动画库做的事基本就是一下四点：1，定时器；2，各种属性变量处理的封装；3，过程控制；4，缓动函数。

实际运用中还是推荐大家用动画库，不满足业务需求的可以自己整合，当然学习的时候可以找个简单的读下源码，试着自己写下核心功能，深入理解动画库的本质，入门我推荐**Kute.js**。

## 动画库推荐（各自优劣势及区别下次再详述）
[jquery animate(插件jquery.easing.js)][9]
[Tween.js][10]
[GSAP][11]	
[CreateJS][12]
[Kute.js][13]
	


  [1]: http://gsgd.co.uk/sandbox/jquery/easing/
  [2]: http://www.zhangxinxu.com/study/201612/how-to-use-tween-js.html
  [3]: https://github.com/tweenjs/tween.js/blob/master/src/Tween.js
  [4]: http://gsgd.co.uk/sandbox/jquery/easing/
  [5]: https://greensock.com
  [6]: https://github.com/CreateJS/TweenJS/blob/master/src/tweenjs/Ease.js
  [7]: http://thednp.github.io/kute.js/
  [8]: https://greensock.com/docs/Easing
  [9]: http://gsgd.co.uk/sandbox/jquery/easing/
  [10]: https://github.com/tweenjs/tween.js
  [11]: https://greensock.com
  [12]: https://github.com/CreateJS
  [13]: http://thednp.github.io/kute.js/