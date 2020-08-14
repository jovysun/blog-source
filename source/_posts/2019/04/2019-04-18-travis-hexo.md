---
title: Travis CI实现hexo博客自动部署
tags:
  - CI/CD
  - 实战
categories:
  - 其他
date: 2019-04-18 17:55:02
updated: 2019-04-18 17:55:02
---
# 前言

更新2020.05.14：超简单的[官方教程](https://docs.travis-ci.com/user/tutorial/#to-get-started-with-travis-ci-using-github)。
更新：发现另一种更方便的[实现方法](https://juejin.im/post/5c415a25f265da61285a6010)。


此篇默认读者已经使用过hexo写过几篇博文，并且已经创建好SSH key。

# 概述
hexo写博客，为了多端同步我们会切个开发分支，每次写完都要提交代码，同时还得部署，文章多了，等待部署时间较长。能不能提交后自动部署呢？
<!-- more -->

# 详述
Travis CI 是一个持续集成(Continuous integration，简称CI)的工具。它可以在公共的 Github 仓库上免费使用。

1. [Travis CI](https://travis-ci.org)注册账号，可以github账号授权登录；
2. 根目录创建`travis.yml`和`.travis`文件夹；
3. 安装travis（macOS系统为例）并登录；
    ```shell
    $ sudo gem install travis
    $ travis login --auto
    ```
4. 加密私钥
    ```shell
    $ travis encrypt-file ~/.ssh/id_rsa --add
    ```
    会在当前目录生成`id_rsa.enc`文件，并且`.travis.yml`文件中自动生成如下代码：
    ```
    before_install:
    - openssl aes-256-cbc -K $encrypted_830d3b21a25d_key -iv $encrypted_830d3b21a25d_iv
      -in ~/.ssh/id_rsa.enc -out ~/.ssh/id_rsa -d
    ```
5. 在`.travis`文件夹下创建ssh_config文件，并添加代码如下：
    ```
    Host github.com
        User git
        StrictHostKeyChecking no
        IdentityFile ~/.ssh/id_rsa
        IdentitiesOnly yes
    ```
6. 进入Travis CI网站设置页面，打开我们要集成的库，示例如下：
    ![示例图](1.png)

7. 配置`.travis.yml`文件，示例如下：

    ```yml
    # 配置语言及相应版本
    language: node_js

    node_js:
      - "10"

    #项目所在分支
    # branches:
    #   only:
    #   - master

    # 缓存
    cache:
      directories:
        - node_modules

    before_install:
    - openssl aes-256-cbc -K $encrypted_eced380b421a_key -iv $encrypted_eced380b421a_iv
      -in id_rsa.enc -out ~/.ssh/id_rsa -d
    # 改变文件权限
    - chmod 600 ~/.ssh/id_rsa 
    # 配置 ssh
    - eval $(ssh-agent)
    - ssh-add ~/.ssh/id_rsa
    - cp .travis/ssh_config ~/.ssh/config
    # 配置 git 替换为自己的信息
    - git config --global user.name 'jovysun'
    - git config --global user.email jovy_sun@163.com


    # 安装依赖
    install:
    - npm install hexo-cli -g
    - npm install

    # 部署的命令
    script:
    - npm run deploy  # hexo clean && hexo g -d

    ```
8. 打开package.json文件添加hexo部署命令，示例如下：
    ```json
      "scripts": {
        "deploy": "hexo clean && hexo g -d"
      }
    ```

这样每次提交完代码，就会自动触发Travis CI的自动集成即自动部署。
# 参考
https://segmentfault.com/a/1190000004667156