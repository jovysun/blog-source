---
title: JS动画（一）——定时器详解
tags:
  - 动效
  - JavaScript
categories:
  - 前端
date: 2018-04-09 17:40:42
updated: 2018-08-07 08:36:16
---
广义说：一切通过js改变的视觉呈现都叫动画；例如，按钮，链接等元素交互反馈。
狭义说：通过定时器连续调用js函数进行元素属性改变产生的视觉动画效果。
## 定时器 ##
定时器是JavaScript动画的核心技术；
setTimeout(),setInterval()是大家熟知的，以前经常使用的；
一般都是做些辅助性，锦上添花的事；
细心的人可能会发现一个现象，从其他标签页切换到有循环动画页面会有卡顿和急速帧切换现象；
问题就在于他们的内在运行机制；

### 认识setTimeout ###
第一个参数推荐用函数形式，字符串形式会两次解析，还有eval一样的问题；
不止两个参数，可以更多，见示例1；
this指向问题，见示例2；
返回值是个整数；
clearTimeout(timer)取消定时器；
setInterval,clearInterval同上；

示例1：
```
setTimeout(function(a,b){ 
	console.log(a+b); 
},1000,1,1)；

```
示例2：
```
var a = 0;
function foo(){
    console.log(this.a);
};
var obj = {
    a : 2,
    foo:foo
}
setTimeout(obj.foo,100);

```
### 运行机制 ###
示例：

```
setTimeout(function(){ 
	console.log(1); 
}); 
console.log(0);
```
原因：加入队列，阻塞执行。

setTimeout图例：
![](setTimeout.png)

setInterval图例：
![](setInterval.png)
### 存在即合理 ###
父子元素事件冒泡，需要先执行父元素，见示例3；
用户自定义的回调函数，通常在浏览器的默认动作之前触发，见示例4；

示例3：
```
<div id="myDiv" style="height: 100px;width: 100px;background-color: pink;"></div>
<script>
myDiv.onclick = function(){
    setTimeout(function(){
        alert(0);
    })
}
document.onclick = function(){
    alert(1);
}
</script>

```
示例4：

```
<input type="text" id="myInput">
<script>
myInput.onkeypress = function(event) {
    setTimeout(function(){
        myInput.value = myInput.value.toUpperCase();
    });
}
</script>

```
### 认识requestAnimationFrame ###
用法与setTimeout类似，只是不需要时间参数；
机制完全不同：
1， setTimeout是异步操作，加入任务队列（ event loop ），当js引擎线程中同步代码执行完才会从任务队列中取出执行；
2，raf是用户代理（浏览器）专门针对动画开发的接口，用户代理会以合适的频率进行动画帧更新（一般同显示器刷新频率，1000/60ms），在隐藏或者非活动页面会停止帧更新，节省CPU资源；
3，[raf示例][1]
### raf简单兼容 ###

```
window.requestAnimFrame = (function(){ 
    return  window.requestAnimationFrame || 
            window.webkitRequestAnimationFrame ||         
            window.mozRequestAnimationFrame || 
            function( callback ){ 	
                window.setTimeout(callback, 1000 / 60);
         	};
     })();

```
参考：
[setTimeout详细介绍][2]


  [1]: https://codepen.io/lovechoose/pen/OzVPgg
  [2]: http://www.cnblogs.com/xiaohuochai/p/5773183.html