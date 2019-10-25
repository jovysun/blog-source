---
title: nodejs实现静态服务器
permalink: nodejs-static-server
tags:
  - Node.js
categories:
  - 后端
date: 2019-10-25 08:46:59
updated: 2019-10-25 08:46:59
---

# 概述

本文通过用 Node.js 实现一个 web 静态服务器来深入学习 node 和 http 相关知识。第一部分实现一个最基本的 web 静态服务器，第二部分实现一个含有缓存、压缩、命令行等功能的一个 web 静态服务器。

<!-- more -->

# 详述

## 基础实现

```js
const http = require('http');
const path = require('path');
const fs = require('fs');
const url = require('url');
const mime = require('mime');

http
  .createServer(function(req, res) {
    const publicPath = path.join(__dirname, 'public');

    // 第一个坑，如果引用的文件是带版本号的，不处理是读取不到这个文件的
    // H:\workspace\nodejs\static-server\basic\public\fonts\fontawesome-webfont.woff?v=4.2.0
    // 方法一：
    // let filename = path.join(publicPath, req.url);
    // if (filename.indexOf('?') != -1) {
    //   filename = filename.substring(0, filename.indexOf('?'))
    // }
    // 方法二：
    const { pathname } = url.parse(req.url);
    const filename = path.join(publicPath, pathname);

    fs.readFile(filename, function(err, data) {
      if (err) {
        res.setHeader('Content-type', 'text/html;charset=utf8');
        res.end('文件不存在 404');
        return;
      }
      res.setHeader('Content-Type', mime.getType(filename));
      res.end(data);
    });
  })
  .listen('3000', function() {
    console.log('server started in http://localhost:3000');
  });
```

使用 http 模块实例一个 server 对象，用 path 模块处理路径，用 fs 模块进行文件的 I/O 操作，用 url 模块对 URL 处理，用 mime 模块处理文件类型。

实现的时候遇到一个坑，测试时字体图标一直不显示：
![字体图标不显示](01.jpg)
最后才发现原来是样式文件中定义`@font-face`时引用的字体带了查询信息，直接用`req.url`拼接的文件路径也是带版本号等查询信息的，而 fs 模块读取文件是完全匹配，因此读取不到该字体文件。处理的思路就是去掉查询信息，代码中给出了两种实现方法。

## 进阶实现

这个网上有篇[深入 nodejs-搭建静态服务器（实现命令行）](https://segmentfault.com/a/1190000018101338)讲的很详细，这里只摘录其中主体代码，然后着重说下其中涉及到的知识点，但是原文没有说明的部分。

```js
// app.js
const http = require('http');
const url = require('url');
const path = require('path');
const fs = require('fs');
const mime = require('mime'); // 文件类型
const crypto = require('crypto'); // 加密
const zlib = require('zlib'); // 压缩
const openbrowser = require('open'); // 自动启动浏览器
const handlebars = require('handlebars'); // 模板引擎
const templates = require('./templates'); // 模板文件目录

class StaticServer {
  constructor(options) {
    this.host = options.host;
    this.port = options.port;
    this.rootPath = process.cwd();
    this.cors = options.cors;
    this.openbrowser = options.openbrowser;
  }

  /**
   * handler request
   * @param {*} req
   * @param {*} res
   */
  requestHandler(req, res) {
    const { pathname } = url.parse(req.url);
    const filepath = path.join(this.rootPath, pathname);

    // To check if a file exists
    fs.stat(filepath, (err, stat) => {
      if (!err) {
        if (stat.isDirectory()) {
          this.responseDirectory(req, res, filepath, pathname);
        } else {
          this.responseFile(req, res, filepath, stat);
        }
      } else {
        this.responseNotFound(req, res);
      }
    });
  }

  /**
   * Reads the contents of a directory , response files list to client
   * @param {*} req
   * @param {*} res
   * @param {*} filepath
   */
  responseDirectory(req, res, filepath, pathname) {
    fs.readdir(filepath, (err, files) => {
      if (!err) {
        const fileList = files.map(file => {
          const isDirectory = fs.statSync(filepath + '/' + file).isDirectory();
          return {
            filename: file,
            url: path.join(pathname, file),
            isDirectory
          };
        });
        const html = handlebars.compile(templates.fileList)({
          title: pathname,
          fileList
        });
        res.setHeader('Content-Type', 'text/html');
        res.end(html);
      }
    });
  }

  /**
   * response resource
   * @param {*} req
   * @param {*} res
   * @param {*} filepath
   */
  async responseFile(req, res, filepath, stat) {
    this.cacheHandler(req, res, filepath).then(
      data => {
        if (data === true) {
          res.writeHead(304);
          res.end();
        } else {
          res.setHeader(
            'Content-Type',
            mime.getType(filepath) + ';charset=utf-8'
          );
          res.setHeader('Etag', data);

          this.cors && res.setHeader('Access-Control-Allow-Origin', '*');

          const compress = this.compressHandler(req, res);

          if (compress) {
            fs.createReadStream(filepath)
              .pipe(compress)
              .pipe(res);
          } else {
            fs.createReadStream(filepath).pipe(res);
          }
        }
      },
      error => {
        this.responseError(req, res, error);
      }
    );
  }

  /**
   * not found request file
   * @param {*} req
   * @param {*} res
   */
  responseNotFound(req, res) {
    const html = handlebars.compile(templates.notFound)();
    res.writeHead(404, {
      'Content-Type': 'text/html'
    });
    res.end(html);
  }

  /**
   * server error
   * @param {*} req
   * @param {*} res
   * @param {*} err
   */
  responseError(req, res, err) {
    res.writeHead(500);
    res.end(`there is something wrong in th server! please try later!`);
  }

  /**
   * To check if a file have cache
   * @param {*} req
   * @param {*} res
   * @param {*} filepath
   */
  cacheHandler(req, res, filepath) {
    return new Promise((resolve, reject) => {
      const readStream = fs.createReadStream(filepath);
      const md5 = crypto.createHash('md5');
      const ifNoneMatch = req.headers['if-none-match'];
      readStream.on('data', data => {
        md5.update(data);
      });

      readStream.on('end', () => {
        let etag = md5.digest('hex');
        if (ifNoneMatch === etag) {
          resolve(true);
        }
        resolve(etag);
      });

      readStream.on('error', err => {
        reject(err);
      });
    });
  }

  /**
   * compress file
   * @param {*} req
   * @param {*} res
   */
  compressHandler(req, res) {
    const acceptEncoding = req.headers['accept-encoding'];
    if (/\bgzip\b/.test(acceptEncoding)) {
      res.setHeader('Content-Encoding', 'gzip');
      return zlib.createGzip();
    } else if (/\bdeflate\b/.test(acceptEncoding)) {
      res.setHeader('Content-Encoding', 'deflate');
      return zlib.createDeflate();
    } else {
      return false;
    }
  }

  /**
   * server start
   */
  start() {
    const server = http.createServer((req, res) =>
      this.requestHandler(req, res)
    );
    server.listen(this.port, () => {
      if (this.openbrowser) {
        openbrowser(`http://${this.host}:${this.port}`);
      }
      console.log(`server started in http://${this.host}:${this.port}`);
    });
  }
}

module.exports = StaticServer;
```

像`url.parse()`、`fs.stat()`等都是标准的 node 内置模块 API，直接查官方文档就好。其他延展知识点有：

### `process.cwd()`与`__dirname` 区别

process.cwd()返回的是当前 Node.js 进程执行时的工作目录，保证了文件在不同的目录下执行时，路径始终不变。
\_\_dirname 是当前模块的目录名，是当前被执行的 js 所在的目录。

### Node 实现四种缓存

```js
const http = require('http');

// Expires
let server = http.createServer((req, res) => {
  res.setHeader('Expires', new Date().toGMTString());
  res.end('harttle.land');
});

// Cache-Control
let server = http.createServer((req, res) => {
  res.setHeader('Cache-Control', 'public, max-age=86400');
  res.end('harttle.land');
});

// Etag
let server = http.createServer((req, res) => {
  console.log(req.url, req.headers['if-none-match']);
  if (req.headers['if-none-match']) {
    res.statusCode = 304;
    res.end();
  } else {
    res.setHeader('Etag', '12345678');
    res.end('harttle.land');
  }
});

// Last-Modified
let server = http.createServer((req, res) => {
  console.log(req.url, req.headers['if-modified-since']);
  if (req.headers['if-modified-since']) {
    res.statusCode = 304;
    res.end();
  } else {
    res.setHeader('Last-Modified', new Date().toISOString());
    res.end('harttle.land');
  }
});

console.log('server start at http://localhost:3333');
server.listen(3333);
```

[更多介绍](https://www.jianshu.com/p/227cee9c8d15)

### `/\bgzip\b/`中的`\b`是什么

`\b`表示字母数字与非字母数字的边界，非字母数字与字母数字的边界。其实就是一个位置匹配，就像开头`^`，结尾`$`。

### 自动启动默认浏览器

这里用了第三方模块`open`，当然可以自己封装个：

```js
//打开默认浏览器
const openDefaultBrowser = function(url) {
  var exec = require('child_process').exec;
  console.log(process.platform);
  switch (process.platform) {
    case 'darwin':
      exec('open ' + url);
      break;
    case 'win32':
      exec('start ' + url);
      break;
    default:
      exec('xdg-open', [url]);
  }
};
openDefaultBrowser('http://localhost:3000');
```

### `#!/usr/bin/env node`干嘛用的

指定用 node 来执行脚本文件。[更多介绍](https://juejin.im/post/5cb93cd651882578b148c637)

# 参考

https://edu.aliyun.com/lesson_1730_14108?spm=5176.10731542.0.0.7fe04a3e9HronY#_14108

https://segmentfault.com/a/1190000018101338

https://www.jianshu.com/p/aecab1749734

https://blog.csdn.net/qq1036548849/article/details/86470140
