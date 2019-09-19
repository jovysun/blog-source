---
title: species-in-pieces网站动效的JS实现
tags:
  - 动效
  - svg
  - JavaScript
categories:
  - 前端
date: 2019-03-06 10:48:36
updated: 2019-03-06 10:48:36
---
# 前言
看到[species](http://species-in-pieces.com/)网站做的很炫，想要借鉴，发现主要是用css3的`clip-path`实现的，兼容不好，因此想着用js实现下。下面作简单介绍，需要详细代码见[github库](https://github.com/jovysun/species-JS)。
# 概述
效果图如下：
![效果图](1.gif)
<!-- more -->
# 详述

## 基础知识
1. SVG基本知识，重点viewBox，polygon；
2. GSAP动画平台，重点TimelineMax，TweenMax；
3. parcel构建工具的基本使用，[parcel](http://www.css88.com/doc/parcel/)。
## 实现思路
根据参考网站的代码，动物图案是用`clip-path: polygon()`实现的，第一时间想到了SVG的polygon;另外对于转场动画，过渡动画，找个自己熟悉的动画库实现就行了。需要特别说明的是：
1. css的clip-path用的用的百分比数值，svg的polygon的points值不能用百分比数值，知道viewBox概念的应该清楚，其实points的值也不是一般认为的绝对像素值，因此写了个工具函数`parsePolygonStr`。
2. 因为图案是分动物和场景（树枝，石头等）两部分，并且希望先绘制动物再绘制场景，因此HTML部分用g标签分成`extra`和`anis`。
## 主要的代码如下：
### 入口文件HTML
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>species</title>
</head>

<body>
    <div id="wrap">
        <svg class="stage" viewBox="0 0 1000 700" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
            <g id="extra">
                <polygon points="793.50,476.88,949.50,500.88,805.50,518.88" fill="hsla(0, 0%, 100%, 0)"></polygon>
                <polygon points="793.50,476.88,949.50,500.88,805.50,518.88" fill="hsla(0, 0%, 100%, 0)"></polygon>
                <polygon points="793.50,476.88,949.50,500.88,805.50,518.88" fill="hsla(0, 0%, 100%, 0)"></polygon>
            </g>
            <g id="anis">
                
            </g>
        </svg>
        <button id="go_btn">GO!</button>
        <h1 class="name"></h1>
        <h2 class="desc"></h2>
    </div>
    <script src="./index.js"></script>
</body>

</html>
```
### 主要js脚本
```js
// 导入一个 SCSS module
import '../css/main.scss';

import { data, preData } from './data'

import { TweenMax, TweenLite, TimelineMax } from 'gsap'

// NodeList转换Array
function NodeList2Array(nodelist) {
    let arr = [];
    if (nodelist.length) {
        arr = Array.prototype.slice.call(nodelist, 0);
    }
    return arr;
}
// 把'clip-path'值转成svg polygon可用的值
function parsePolygonStr(polygonStr, width, height) {
    let pointsArr = polygonStr.split(/\s+|,\s/);
    let newPointArr = pointsArr.map(function(currentVal, index, arr) {
        if (index % 2 === 0) {
            return (parseFloat(currentVal) * width / 100).toFixed(2);
        } else {
            return (parseFloat(currentVal) * height / 100).toFixed(2);
        }
    });

    return newPointArr;
}



let body = document.querySelector('body'),
    wrap = document.querySelector('#wrap'),
    name = wrap.querySelector('.name'),
    desc = wrap.querySelector('.desc'),
    stage = wrap.querySelector('.stage'),
    anis = document.querySelector('#anis'),
    extra = document.querySelector('#extra'),
    goBtn = document.querySelector('#go_btn'),
    anisPolygons = null,
    extraPolygons = null;


let currentSpeciesIndex = 0,
    width = 1000,
    height = 700;


function init() {
    let initSpecies = preData.preload;
    name.innerHTML = initSpecies.name;
    desc.innerHTML = initSpecies.desc;
    body.style.background = initSpecies.background;

    let polygonArr = initSpecies.polygon;

    if (Object.prototype.toString.call(polygonArr) === '[object Array]') {
        let polygonHtml = '';

        polygonArr.forEach(function(element, index) {
            let pointsVal = parsePolygonStr(element[0], width, height);
            polygonHtml += '<polygon points="' + pointsVal + '" fill="' + element[1] + '"/>';
        });
        anis.innerHTML = polygonHtml;

    }
}
init();
anisPolygons = anis.querySelectorAll('polygon');
extraPolygons = extra.querySelectorAll('polygon');

let tl = new TimelineMax({ delay: 0.2 });
// 初始的loading动画
NodeList2Array(anisPolygons).forEach(function(target, index) {
    let tm = TweenMax.fromTo(target, 0.9, { attr: { fill: 'rgba(0, 0, 0, .7)' } }, { attr: { fill: 'rgba(200, 20, 20, .45)' }, ease: Power0.easeNone, repeat: -1, yoyo: true });
    tl.add(tm, 0.9 - 0.03 * index);
})

// 模拟加载完成
setTimeout(function() {
    // 清除tl
    tl.clear();
    // loading完之后的一系列动画
    // 1，变色，放大，爆炸碎片
    tl.add(
            [
                TweenMax.to('#anis polygon', .6, {
                    attr: {
                        fill: function(index) {
                            let fillVal = '#111';
                            if (index % 5 === 0) {
                                fillVal = '#28282a';
                            } else if (index % 5 === 1) {
                                fillVal = '#111';
                            } else if (index % 5 === 2) {
                                fillVal = '#333';
                            } else if (index % 5 === 3) {
                                fillVal = '#222';
                            } else if (index % 5 === 4) {
                                fillVal = '#121212';
                            }
                            return fillVal;
                        }
                    }
                }),
                TweenMax.to('#wrap .stage', .6, {
                    scale: 1,
                    ease: Back.easeOut.config(1.7)
                }),
                TweenMax.to('#anis polygon', .6, {

                    attr: {
                        points: function(index, target) {
                            let nextSpeciesPolygon = preData.ready.polygon;
                            // debugger
                            return parsePolygonStr(nextSpeciesPolygon[index][0], width, height)
                        },
                        fill: function(index, target) {
                            let nextSpeciesPolygon = preData.ready.polygon;
                            return nextSpeciesPolygon[index][1];
                        },
                    }
                    // ease: Power2.easeInOut,

                })
            ]
        )
        // 2，海豚
        .add(
            TweenLite.to('#anis polygon', .6, {

                attr: {
                    points: function(index, target) {
                        let nextSpeciesPolygon = preData.preAni.polygon;
                        return parsePolygonStr(nextSpeciesPolygon[index][0], width, height)
                    },
                    fill: function(index, target) {
                        let nextSpeciesPolygon = preData.preAni.polygon;
                        return nextSpeciesPolygon[index][1];
                    },
                },
                // ease: Power2.easeInOut,
            })
        )
        // 3，爆炸碎片
        .add(
            TweenMax.to('#anis polygon', .6, {

                attr: {
                    points: function(index, target) {
                        let nextSpeciesPolygon = preData.ready.polygon;
                        // debugger
                        return parsePolygonStr(nextSpeciesPolygon[index][0], width, height)
                    },
                    fill: function(index, target) {
                        let nextSpeciesPolygon = preData.ready.polygon;
                        return nextSpeciesPolygon[index][1];
                    },
                }
            }),
            '+=0.4'
        )
        // 4，“piece”logo
        .add(
            TweenMax.to('#anis polygon', .6, {

                attr: {
                    points: function(index, target) {
                        let nextSpeciesPolygon = preData.title.polygon;
                        // debugger
                        return parsePolygonStr(nextSpeciesPolygon[index][0], width, height)
                    },
                    fill: function(index, target) {
                        let nextSpeciesPolygon = preData.title.polygon;
                        return nextSpeciesPolygon[index][1];
                    },
                }
            }),
            '+=0.4'
        );

}, 3000);



// 动物图案切换
function playHandler() {
    let nextSpecies = data[currentSpeciesIndex++];
    if (!nextSpecies) {
        return false;
    }

    name.innerHTML = nextSpecies.name;
    desc.innerHTML = nextSpecies.desc;
    body.style.background = nextSpecies.background;

    let nextSpeciesPolygon = nextSpecies.polygon;

    let subTl = new TimelineMax({ pause: true });

    let arr1 = NodeList2Array(anisPolygons);
    let arr2 = NodeList2Array(extraPolygons);

    // 之所以没用TweenMax.staggerTo是因为属性对象中没法用获得index，如下实现不了
    // attr: {
    //     points: pointVal.join(' '),
    //     fill: function(index){return nextSpeciesPolygon[index][1];}
    // }

    arr1.concat(arr2).forEach(function(target, index) {
        let pointVal = parsePolygonStr(nextSpeciesPolygon[index][0], width, height),
            fillVal = nextSpeciesPolygon[index][1];
        subTl.add(
            TweenMax.to(target, 0.5, {
                attr: {
                    points: pointVal.join(' '),
                    fill: fillVal
                },
                ease: Back.easeOut.config(1.7)
            }),
            '-=0.47'
        )
    });

    subTl.play();


}

goBtn.addEventListener('click', playHandler, false);
```
# 后记
[GSAP动画平台](https://greensock.com/gsap)真的很强大，推荐有交互动效方面需求的可以关注下。