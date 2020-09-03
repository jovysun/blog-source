---
title: 浏览器输入URL后发生了什么
permalink: browser-enter-url
tags:
  - HTML
  - CSS
  - JavaScript
categories:
  - 其他
date: 2020-09-02 16:09:28
updated: 2020-09-02 16:09:28
---

# 概述

“浏览器输入 URL 后发生了什么”，这是一个高频的前端面试问题，经常作为压轴题出场，主要是对 web 知识进行综合考察。涉及的知识点：URL 解析、缓存机制、域名解析机制、TCP 链接机制、HTTP 请求与响应、页面渲染机制等。

<!-- more -->

# 详述

主体流程为：首先解析 URL，然后查找缓存，存在有效缓存资源则直接返回 304 响应，否则进行域名解析，然后建立 TCP 连接，然后发送 HTTP 请求，然后服务器收到请求后进行响应，返回 response 对象，然后浏览器收到响应后进行页面渲染，最后关闭 TCP 连接。

## URL

URL 是统一资源定位符，是互联网上标准资源的地址。包括以下几个部分：

- protocol 协议，例如 http、https、ftp、telnet、file 等。
- host 可以是域名，也可以是主机名，或 IP 地址。
- port 端口号，HTTP 默认端口 80，HTTPS 默认端口 443。
- path 路径，访问的资源在服务器下的相对路径，是服务器上的一个目录或者文件地址。
- query 查询参数，查询搜索的部分，通过问号？连接到路径后面，有时候也归类到路径中。
- hash 即#后的值，一般用来定位到某个位置。

## 缓存

判断是直接使用缓存内容还是重新向服务器请求资源，逻辑如下：

![缓存](cache.png)

## DNS

域名系统（英文：Domain Name System，缩写：DNS）是互联网的一项服务。它作为将域名和 IP 地址相互映射的一个分布式数据库，能够使人更方便地访问互联网。

域名系统(Domain Name System,DNS)是 Internet 上解决网上机器命名的一种系统。就像拜访朋友要先知道别人家怎么走一样，Internet 上当一台主机要访问另外一台主机时，必须首先获知其地址，TCP/IP 中的 IP 地址是由四段以“.”分开的数字组成，记起来总是不如名字那么方便，所以，就采用了域名系统来管理名字和 IP 的对应关系。

在域名解析的过程中会有多级的缓存，浏览器首先看一下自己的缓存里有没有，如果没有就向操作系统的缓存要，还没有就检查本机域名解析文件 hosts。如果没找到则会查找本地 DNS 解析器缓存，如果查找到则返回。如果还是没有找到则会查找本地 DNS 服务器，如果查找到则返回。

### DNS 域名解析的过程

网络客户端就是我们平常使用的电脑，打开浏览器，输入一个域名。比如输入 `www.163.com`，这时，你使用的电脑会发出一个 DNS 请求到本地 DNS 服务器。本地 DNS 服务器一般都是你的网络接入服务器商提供，比如中国电信，中国移动。

查询 `www.163.com` 的 DNS 请求到达本地 DNS 服务器之后，本地 DNS 服务器会首先查询它的缓存记录，如果缓存中有此条记录，就可以直接返回结果。如果没有，本地 DNS 服务器还要向 DNS 根服务器进行查询。

根 DNS 服务器没有记录具体的域名和 IP 地址的对应关系，而是告诉本地 DNS 服务器，你可以到域服务器上去继续查询，并给出域服务器的地址。

本地 DNS 服务器继续向域服务器发出请求，在这个例子中，请求的对象是.com 域服务器。.com 域服务器收到请求之后，也不会直接返回域名和 IP 地址的对应关系，而是告诉本地 DNS 服务器，你的域名的解析服务器的地址。

最后，本地 DNS 服务器向域名的解析服务器发出请求，这时就能收到一个域名和 IP 地址对应关系，本地 DNS 服务器不仅要把 IP 地址返回给用户电脑，还要把这个对应关系保存在缓存中，以备下次别的用户查询时，可以直接返回结果，加快网络访问。

![DNS](dns.jpg)

## TCP 连接

DNS 域名解析后，获取到了服务器的 IP 地址，然后与服务器通过 TCP 三次握手实现连接，完成三次握手，客户端与服务器开始传送数据。具体如下：

- 第一次握手： 建立连接时，客户端发送 syn 包（seq=x）到服务器，并进入 SYN_SENT 状态，等待服务器确认；
- 第二次握手： 服务器收到 syn 包，必须确认客户的 SYN（ack=x+1），同时自己也发送一个 SYN 包（seq=y），即 SYN+ACK 包，此时服务器进入 SYN_RECV 状态；
- 第三次握手： 客户端收到服务器的 SYN+ACK 包，向服务器发送确认包 ACK(ack=y+1），此包发送完毕，客户端和服务器进入 ESTABLISHED（TCP 连接成功）状态，完成三次握手。

![TCP](tcp-hello.png)

## HTTP 请求与响应

### 请求

一个 HTTP 请求报文由请求行（request line）、请求头部（headers）、空行（blank line）和请求数据（request body）4 个部分组成。

- 请求行分为三个部分：请求方法、请求地址 URL 和 HTTP 协议版本，它们之间用空格分割。例如，GET /index.html HTTP/1.1。
- 请求头部为请求报文添加了一些附加信息，由“名/值”对组成，每行一对，名和值之间使用冒号分隔。请求头部的最后会有一个空行，表示请求头部结束，接下来为请求数据。
- 请求数据不在 GET 方法中使用，而在 POST 方法中使用。POST 方法适用于需要客户填写表单的场合。与请求数据相关的最长使用的请求头部是 Content-Type 和 Content-Length。

#### 常用的请求头部

```
Accept: 接收类型，表示浏览器支持的MIME类型
（对标服务端返回的Content-Type）
Accept-Encoding：浏览器支持的压缩类型,如gzip等,超出类型不能接收
Content-Type：客户端发送出去实体内容的类型
Cache-Control: 指定请求和响应遵循的缓存机制，如no-cache
If-Modified-Since：对应服务端的Last-Modified，用来匹配看文件是否变动，只能精确到1s之内，http1.0中
Expires：缓存控制，在这个时间内不会请求，直接使用缓存，http1.0，而且是服务端时间
Max-age：代表资源在本地缓存多少秒，有效时间内不会请求，而是使用缓存，http1.1中
If-None-Match：对应服务端的ETag，用来匹配文件内容是否改变（非常精确），http1.1中
Cookie: 有cookie并且同域访问时会自动带上
Connection: 当浏览器与服务器通信时对于长连接如何进行处理,如keep-alive
Host：请求的服务器URL
Origin：最初的请求是从哪里发起的（只会精确到端口）,Origin比Referer更尊重隐私
Referer：该页面的来源URL(适用于所有类型的请求，会精确到详细页面地址，csrf拦截常用到这个字段)
User-Agent：用户客户端的一些必要信息，如UA头部等

```

### 响应

服务器在收到浏览器发送的 HTTP 请求之后，会将收到的 HTTP 报文封装成 HTTP 的 Request 对象，并通过不同的 Web 服务器进行处理，处理完的结果以 HTTP 的 Response 对象返回。
HTTP 响应报文由状态行（status line）、相应头部（headers）、空行（blank line）和响应数据（response body）4 个部分组成。

#### 状态码

- 1xx：指示信息–表示请求已接收，继续处理。
- 2xx：成功–表示请求已被成功接收、理解、接受。
- 3xx：重定向–要完成请求必须进行更进一步的操作。
- 4xx：客户端错误–请求有语法错误或请求无法实现。
- 5xx：服务器端错误–服务器未能实现合法的请求。

响应头主要由 Cache-Control、 Connection、Date、Pragma 等组成。响应体为服务器返回给浏览器的信息，主要由 HTML，css，js，图片文件组成。

#### 常用的响应头部（部分）：

```
Access-Control-Allow-Headers: 服务器端允许的请求Headers
Access-Control-Allow-Methods: 服务器端允许的请求方法
Access-Control-Allow-Origin: 服务器端允许的请求Origin头部（譬如为*）
Content-Type：服务端返回的实体内容的类型
Date：数据从服务器发送的时间
Cache-Control：告诉浏览器或其他客户，什么环境可以安全的缓存文档
Last-Modified：请求资源的最后修改时间
Expires：应该在什么时候认为文档已经过期,从而不再缓存它
Max-age：客户端的本地资源应该缓存多少秒，开启了Cache-Control后有效
ETag：请求变量的实体标签的当前值
Set-Cookie：设置和页面关联的cookie，服务器通过这个头部把cookie传给客户端
Keep-Alive：如果客户端有keep-alive，服务端也会有响应（如timeout=38）
Server：服务器的一些相关信息
```

#### 请求示例分析

![request](request.png)

## 页面渲染

前面有提到 http 交互，那么接下来就是浏览器获取到 html，然后解析，渲染

1.  解析 HTML，构建 DOM 树
2.  解析 CSS，生成 CSS 规则树
3.  合并 DOM 树和 CSS 规则，生成 render 树
4.  布局 render 树（Layout/reflow），负责各元素尺寸、位置的计算
5.  绘制 render 树（paint），绘制页面像素信息
6.  浏览器会将各层的信息发送给 GPU，GPU 会将各层合成（composite），显示在屏幕上

### 示例图：

![页面渲染](render.png)

![DOM树](dom-tree.png)

![样式树](style-tree.png)

在浏览器还没接收到完整的 HTML 文件时，它就开始渲染页面了，在遇到外部链入的脚本标签或样式标签或图片时，会再次发送 HTTP 请求重复上述的步骤。在收到 CSS 文件后会对已经渲染的页面重新渲染，加入它们应有的样式，图片文件加载完立刻显示在相应位置。在这一过程中可能会触发页面的回流和重绘。

- 回流，一般意味着元素的内容、结构、位置或尺寸发生了变化，需要重新计算样式和渲染树，这个过程称为 Reflow，也称作 Layout。
- 重绘，意味着元素发生的改变只是影响了元素的一些外观之类的时候（例如，背景色，边框颜色，文字颜色等），此时只需要应用新样式绘制这个元素就 OK 了，这个过程称为 Repaint。

所以说 Reflow 的成本比 Repaint 的成本高得多的多。DOM 树里的每个结点都会有 reflow 方法，一个结点的 reflow 很有可能导致子结点，甚至父点以及同级结点的 reflow。

## TCP 连接关闭

![TCP](tcp-bye.png)
通过四次挥手关闭连接(FIN ACK, ACK, FIN ACK, ACK)。

第一次挥手是浏览器发完数据后，发送 FIN 请求断开连接。
第二次挥手是服务器发送 ACK 表示同意，如果在这一次服务器也发送 FIN 请求断开连接似乎也没有不妥，但考虑到服务器可能还有数据要发送，所以服务器发送 FIN 应该放在第三次挥手中。
这样浏览器需要返回 ACK 表示同意，也就是第四次挥手。

# 参考

https://www.jianshu.com/p/7eea6fbc5fcd

https://www.cnblogs.com/crazylqy/p/7110357.html

https://blog.csdn.net/ailunlee/article/details/90600174
