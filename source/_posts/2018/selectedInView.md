---
title: 产品条目不在可视区域的处理
date: 2018-04-17 18:23:05
tags:
  - JavaScript
categories:
  - 前端
---
> 难度系数：简单

> 关键词：length outerHeight position   

# 前言
案例很简单，但是对于初学者可以延伸学习下jquery的相关知识点：
1. 判断选择的元素是否存在用`length`;
2. 获取元素的高度height()与outerHeight()的异同；
3. 判断元素位置position()与offset()的异同。

# 应用场景：
在做动态创建目录的时候，选择的条目不在可视区域，如图：
![场景图](selectedInView.jpg)

默认情况滚动条都是在最上面的，导致超出可视区域的选择条目没有呈现在可视区域，因此要脚本处理下。代码很简单，如下：
## 设置父容器相对定位
```
ul{
    postion:relative;
}
```
## 动态创建完dom结构后调用函数
```
function setSelectedInView(){
    $('.J-item.selected').each(function(){
        var $this = $(this);
        if($this.length > 0){
            var $item = $this.parent(),
                itemHeight = $item.outerHeight(),
                itemTop = $item.position().top;
            var $container = $item.parent('.J-items'),
                containerHeight = $container.height();
            // 如果该条目元素相对于父级的位置超出可视框高度，设置滚动条
            if(itemTop > containerHeight){
                $container.scrollTop(itemTop - containerHeight + itemHeight);
            }
        }
    })
}
```