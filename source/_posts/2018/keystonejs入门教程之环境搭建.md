---
title: （入门）keystonejs入门教程之环境搭建
date: 2018-04-09 17:30:15
tags:
  - keystonejs
  - cms
categories:
  - 后端
---
## 基础环境 ##
Node.js 0.10+ 和MongoDB v2.4+；
## 基础知识 ##
javascript，nodejs，npm，数据库，cms；
## 执行命令 ##

 - 安装脚手架
```
npm install -g generator-keystone
```
 - 创建项目并进入目录

```
mkdir my-test-project
cd my-test-project
```

 - 安装Yeoman（脚手架是用Yeoman制作的）

```
npm install -g yo
```

 - 运行脚手架

```
yo keystone
```

----------
环境搭建完成
----------


## 启动 ##

```
node keystone
```
## 访问 ##
前台页面
[http://localhost:3000][1]
后台页面
[http://localhost:3000/keystone][2]


----------


## 备注 ##

 1. 管理员权限，数据库连接等[常见问题][3]；
 2. 后台登录用户名密码是运行脚手架时输入的，默认是user@keystonejs.com/admin；
 3. windows下安装MongoDB卡死问题，原因是默认安装mongodb compass（比较大）所致，因此安装时选择自定义custom，过程中取消mongodb compass的安装勾选，需要的可以单独安装；
 4. [keystonejs官方网站][4]


  [1]: http://localhost:3000
  [2]: http://localhost:3000/keystone
  [3]: http://keystonejs.com/getting-started/
  [4]: http://keystonejs.com/