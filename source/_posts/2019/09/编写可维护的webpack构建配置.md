---
title: 编写可维护的webpack构建配置
permalink: webpack-config
tags:
  - webpack
categories:
  - 工具
date: 2019-09-29 17:26:07
updated: 2019-09-29 17:26:07
---

# 概述

通过多个配置文件管理不同环境的 webpack 配置，并抽离成 npm 包统一管理。业务开发者只需要安装该 npm 包，并且根据具体环境需要引用对应的配置文件即可。[示例源码](https://github.com/jovysun/builder-webpack-jovy)

<!-- more -->

# 详述

## 构建配置包设计

### 通过多个配置文件管理不同环境的 webpack 配置：

基本配置：webpack.base.js

开发配置：webpack.dev.js

生产配置：webpack.prod.js

SSR 配置：webpack.ssr.js

...

### 抽离成一个 npm 包统一管理

- 规范：Git commit 日志、README、ESLint 规范、Semver 规范
- 质量：冒烟测试、单元测试、测试覆盖率和 CI

## 功能模块设计

![](01.jpg)

## 目录结构设计

![](02.jpg)

## ESLint 规范脚本

使用`eslint-config-airbnb-base`规则库，并用 eslint --fix 自动去除空格。

```js
// .eslintrc.js
module.exports = {
  parser: 'babel-eslint',
  extends: 'airbnb-base',
  env: {
    browser: true,
    node: true
  }
};
```

## 冒烟测试

冒烟测试是指对提交测试的软件在进行详细深入的测试之前而进行的预测试，这种预测试的主要目的是暴露导致软件需重新发布的基本功能失效等严重问题。

### 冒烟测试执行

- 构建是否成功
- 每次构建 build 目录是否有内容输出

```js
// test/index.js
const path = require('path');
const webpack = require('webpack');
const rimraf = require('rimraf');
const Mocha = require('mocha');

const mocha = new Mocha({
  timeout: '10000ms'
});

// template目录为webpack构建项目demo，用于测试我们的webpack配置能否构建成功
process.chdir(path.join(__dirname, 'template'));

rimraf('./dist', () => {
  const prodConfig = require('../../lib/webpack.prod');
  webpack(prodConfig, (err, stats) => {
    if (err) {
      console.error(err);
      process.exit(2);
    }
    console.log(
      stats.toString({
        colors: true,
        modules: false,
        children: false
      })
    );

    console.log('webpack compiler success, begin mocha test');

    mocha.addFile(path.join(__dirname, 'html-test.js'));
    mocha.addFile(path.join(__dirname, 'css-js-test.js'));

    mocha.run();
  });
});
```

是否有 html 文件：

```js
// html-test.js
const glob = require('glob-all');

describe('Checking generated html files', () => {
  it('should generate html files', done => {
    const files = glob.sync(['./dist/index.html', './dist/search.html']);

    if (files.length > 0) {
      done();
    } else {
      throw new Error('No html files found');
    }
  });
});
```

是否有 css、js 等文件：

```js
// css-js-test.js
const glob = require('glob-all');

describe('Checking generated css js files', () => {
  it('should generate css js files', done => {
    const files = glob.sync([
      './dist/index_*.js',
      './dist/index_*.css',
      './dist/search_*.js',
      './dist/search_*.css'
    ]);

    if (files.length > 0) {
      done();
    } else {
      throw new Error('No css js files found');
    }
  });
});
```

## 单元测试与测试覆盖率

单元测试可以选用单纯的测试框架+断言库，如`Mocha+Chai`；也可以选集成测试框架，如`Jasmine`、`Jest`。本次示例用`mocha+assert`：

```js
// test/unit/webpack-base-test.js
const assert = require('assert');

describe('webpack.base.js test case', () => {
  const baseConfig = require('../../lib/webpack.base');
  // console.log(baseConfig);

  it('entry', () => {
    assert.equal(
      baseConfig.entry.index.indexOf(
        'builder-webpack-jovy/test/smoke/template/src/index/index.js'
      ) > -1,
      true
    );
    assert.equal(
      baseConfig.entry.search.indexOf(
        'builder-webpack-jovy/test/smoke/template/src/search/index.js'
      ) > -1,
      true
    );
  });
});
```

测试覆盖率用 npm 包`nyc`，执行脚本为：

```json
  "scripts": {
    "test": "nyc ./node_modules/.bin/_mocha"
  }
```

测试结果示例图：
![](03.jpg)

## 持续集成

本次以 github 上最流行的 CI 工具`Travis CI`为例。

### 接入 Travis CI

1. https://travis-ci.org/ 使用 GitHub 账号登录
2. 在 https://travis-ci.org/account/repositories 为项目开启
3. 项目根目录下新增 `.travis.yml`

### 编写`.travis.yml`

```js
// .travis.yml
language: node_js

sudo: false

cache:
  apt: true
  directories:
    - node_modules

node_js: stable

install:
  - npm install -D
  - cd ./test/smoke/template
  - npm install -D
  - cd ../../../

scripts:
  - npm run test
```

## 发布到 npm

### 添加用户

```shell
npm adduser
```

### 升级版本

```shell
# 升级补丁版本号
npm version patch
# 升级小版本号
npm version minor
# 升级大版本号
npm version major
```

### 发布版本

```shell
npm publish
```

## Git 规范和 Changelog 生成

### 良好的 Git commit 规范优势：

- 加快 Code Review 的流程
- 根据 Git Commit 的元数据生成 Changelog
- 后续维护者可以知道 Feature 被修改的原因

### 日志规范

推荐用 angular 的 git commit 日志规范：

Commit Message 格式:

```
<type>(<scope>): <subject>
<空行>
<body>
<空行>
<footer>
```

可以看出分为三个部分，头部，主体，底部；

header，`<type>(<scope>): <subject>`包括了三个节点：

    type 类型，修改的类型级别
        feat：新功能（feature）
        fix：修补bug
        docs：文档（documentation）
        style： 格式（不影响代码运行的变动）
        refactor：重构（即不是新增功能，也不是修改bug的代码变动）
        test：增加测试
        chore：构建过程或辅助工具的变动
    scope 修改范围
    主要是这次修改涉及到的部分，最好简单的概括
    subject 修改的副标题
    主要是具体修改的加点

body，修改的主体标注

footer，里的主要放置不兼容变更和 Issue 关闭的信息，
这些东西由于我们书写代码本身就会经常性的提交，所以如果每次都这样书写的确是挺烦人的，所以我目前建议自己采用相同的但是更加简单的方式来完成

### Changelog 生成

安装`conventional-changelog-cli`，配置 scripts：

```json
  "scripts": {
    "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s -r 0"
  }
```

生成的效果示例如下：
![](04.jpg)

## 增加 precommit

安装`husky`，配置 package.json：

```json
  "husky": {
    "hooks": {
      "pre-commit": "npm test",
      "pre-push": "npm test"
    }
  }
```

这样每次执行`git commit`和`git push`命令时都会先执行`test`命令进行单元测试。

# 参考

https://segmentfault.com/a/1190000011224170?utm_source=tag-newest

《极客时间》