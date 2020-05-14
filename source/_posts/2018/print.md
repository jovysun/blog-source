---
title: 网页打印实践及参考资料推荐
date: 2018-04-25 17:11:06
tags:
  - CSS
  - 实战
categories:
  - 前端
---
>难度系数：简单
关键词：css print

# 实践
处理前截图：
![图片描述][1]
处理后截图：
![图片描述][2]
## 实践小结
第一次接到网页要提供打印，可能会有些不知所措，我这里只说下遇到表格、图像、列表项内容在尾部被断开的问题处理。
1. 方案选用的是直接调用[window.print()](https://developer.mozilla.org/en-US/docs/Web/API/Window/print)方法;
2. 本次用到的只是处理固定块内容不被截断，对于不想被断开的元素设置css值`page-break-inside: avoid`，例如：
```
.vo-cnt-list{
    li{
        page-break-inside: avoid;
    }
}
```
# 备注
需要了解更详细的，可以参考[ web打印的几种方案 ](https://blog.csdn.net/a89004088/article/details/78362915)和[打印样式设计 ](https://www.w3cplus.com/css/designing-for-print-with-css.html)。


  [1]: print1.jpg
  [2]: print2.jpg