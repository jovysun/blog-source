---
title: nodejs实现本地虚拟主机
permalink: nodejs实现本地虚拟主机
tags:
  - nodejs
  - JavaScript
categories:
  - 其他
date: 2020-08-24 18:57:49
updated: 2020-08-24 18:57:49
---

# 概述
日常开发中经常会遇到本地启动一个或多个后端服务，例如用java集成开发工具`IntelliJ IDEA`或者`Eclipse`启动一个tomcat实例，然后本地可以通过`http://localhost:8081`访问。但是实际项目中各个子应用之间的跳转都是通过域名，例如这个对应的是`http://www.crov.com`。对于相对路径跳转没问题，但是对于一些绝对路径，或者跨域的接口，使用中就会出现问题了。本文就是解决这个问题，实现在本地可以直接通过域名访问本地应用。
<!-- more -->

# 详述
## 修改hosts文件（windows为例）
```
127.0.0.1 crov.micstatic.com
127.0.0.1 www.micstatic.com

127.0.0.1 buyer.crov.com
127.0.0.1 seller.crov.com
127.0.0.1 www.crov.com
127.0.0.1 my.crov.com
127.0.0.1 dropshipping.crov.com
127.0.0.1 dropshipping.doba.com
127.0.0.1 www.doba.com

127.0.0.1 pay.crov.com
127.0.0.1 shoppingcart.crov.com
127.0.0.1 login.crov.com
127.0.0.1 login.doba.com
```
## node搭建一个http服务

### 安装node包`http-proxy`
```js
npm install http-proxy --save
```
### 创建一个入口文件`app.js`
```js
var http = require("http");
var httpProxy = require("http-proxy");

let hosts = {
  "www.crov.com": "http://localhost:8081",
  "dropshipping.crov.com": "http://localhost:8081/dropshipping-channel",
  "www.doba.com": "http://localhost:8081/doba-com",
  "buyer.crov.com": "http://localhost:8082",
  "seller.crov.com": "http://localhost:8083",
  "login.crov.com": "http://localhost:8084",
  "login.doba.com": "http://localhost:8084/doba-com",
  
  "crov.micstatic.com": "http://localhost:2000",
  "www.micstatic.com": "http://localhost:2001",

};

var proxy = httpProxy.createProxyServer();

// 捕获异常
proxy.on("error", function (err, req, res) {
  res.writeHead(500, {
    "Content-Type": "text/plain",
  });
  res.end("Something went wrong. And we are reporting a custom error message.");
});

var proxy_server = http.createServer(function (req, res) {
  // 在这里可以自定义你的路由分发
  var host = req.headers.host,
    ip = req.headers["x-forwarded-for"] || req.connection.remoteAddress;
  console.log("client ip:" + ip + ", host:" + host);

  if (hosts[host]) {
    proxy.web(req, res, { target: hosts[host] });
  } else {
    res.writeHead(200, {
      "Content-Type": "text/plain",
    });
    res.end("Welcome to my server!");
  }

});

proxy_server.listen(80, function () {
  console.log("proxy server is running ");
});

```
### `package.json`中增加启动脚本
```json
{
  "name": "crov-proxy",
  "version": "1.0.0",
  "description": "",
  "main": "proxy.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node app.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "http-proxy": "^1.18.1"
  }
}
```
# 参考
