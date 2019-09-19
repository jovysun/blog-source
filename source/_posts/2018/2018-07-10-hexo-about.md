---
title: hexo相关知识整理
tags:
  - hexo
  - 备忘
categories:
  - 其他
date: 2018-07-10 16:51:19
updated: 2018-07-10 16:51:19
---

# 概述
hexo写博客工具本身相关的知识整理，备忘。
<!-- more -->

# 详述

## 常用命令
### Create a new post

``` bash
$ hexo new "My New Post"
```

### Run server

``` bash
$ hexo server
```

### Generate static files

``` bash
$ hexo generate
```

### Deploy to remote sites

``` bash
$ hexo deploy
```
### 合并生成部署
``` bash
$ hexo d -g
```
### 建立草稿

``` bash
$ hexo new draft "My New Draft"
```
### 本机预览
``` bash
$ hexo s --draft
```
### 草稿发布
``` bash
$ hexo p "My New Draft"
```
### 指定布局模板
``` bash
$ hexo new 布局 "文章名"
```

## 博文插入图片

### 1.安装插件
```bash
npm install hexo-asset-image --save
```
### 2.修改配置`_config.yml`
```yml
post_asset_folder: true
```
### 3.使用
创建博文同时会自动创建一个同名文件夹，把图片素材放入该文件夹，然后就可以在博文中引用了，示例如下：
![hexo博文插入图片](01.jpg)


## 多终端同步主题丢失
1. 更新`$ git clone https://github.com/theme-next/hexo-theme-next themes/next-reloaded`
2. 修改：修改对应主题中的`_config.yml`配置
