---
title: GraphicsMagick for node.js环境搭建
date: 2017-07-12 16:52:41
tags: [gm]
categories: [其他]
---


## 本人背景环境windows7 64位，nodejs v6.11.0，git version 2.7.2.windows.1；

## 具体步骤如下：
1. 下载[GraphicsMagick](ftp://ftp.graphicsmagick.org/pub/GraphicsMagick/windows/)，一路Next完成安装，然后cmd调用命令窗口输入gm回车，如果能正确显示类似如下信息说明安装成功：
```     
    H:\workspace\gm\examples>gm

    GraphicsMagick 1.3.26 2017-07-04 Q16 http://www.GraphicsMagick.org/
    Copyright (C) 2002-2017 GraphicsMagick Group.
    Additional copyrights and licenses apply to this software.
    See http://www.GraphicsMagick.org/www/Copyright.html for details.
    Usage: gm command [options ...]

    Where commands include:
        batch - issue multiple commands in interactive or batch mode
    benchmark - benchmark one of the other commands
        compare - compare two images
    composite - composite images together
        conjure - execute a Magick Scripting Language (MSL) XML script
        convert - convert an image or sequence of images
        help - obtain usage message for named command
    identify - describe an image or image sequence
        mogrify - transform an image or sequence of images
        montage - create a composite image (in a grid) from separate im
        time - time one of the other commands
        version - obtain release version
    register - register this application as the source of messages           
```
2. 如果只是学习可以cmd命令窗口执行`git clone git://github.com/aheckmann/gm.git`，然后到该目录下执行`npm install`完成相关依赖包安装后就可以到examples下试试各种demo了，例如我的执行:

```
    H:\workspace\gm\examples>node append

    H:\workspace\gm\examples/imgs/append.jpg created  ::  gm "convert"
    "#222" "H:\workspace\gm\examples/imgs/lost.png" "H:\workspace\gm\e
    original.jpg" "-append" "H:\workspace\gm\examples/imgs/append.jpg"
```

3. 如果从零开始，实际示例如下：
         
    创建根目录:

            mkdir gm-study

    到该目录下:

            cd gm-study

    初始化创建package.json:

            npm init

    安装gm:

            npm install gm --save

    创建执行文件:

            type nul>index.js

    打开index.js并黏贴[gm](https://github.com/aheckmann/gm)官方示例代码并修改路径:
        
        var fs = require('fs')
        , gm = require('gm');

        // resize and remove EXIF profile data
        gm('imgs/1.jpg')
        .resize(240, 240)
        .noProfile()
        .write('out/resize.png', function (err) {
        if (!err) console.log('done');
        });
        
    执行node命令:

        node index

