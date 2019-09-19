---
title: 从0搭建vue+webpack工程工作流
tags:
  - Vue
  - webpack
categories:
  - 前端
date: 2019-04-04 12:30:20
updated: 2019-04-04 12:30:20
---

# 概述
从实际出发，手动从零搭建一个vue开发工程。从基本依赖到工程工具到代码规范到生产打包，从不同功能层面逐步安装，并作对应说明。技术栈：vue2.x+webpack4.x+babel7.x。[完整代码](https://github.com/jovysun/vue-webpack-project)

<!-- more -->

# 详述
## 初始化
```shell
npm init
```

## 安装

### 基本环境
```shell
npm i vue vue-router vuex -S
```
```shell
npm i webpack webpack-cli vue-loader css-loader vue-style-loader vue-template-compiler -D
```
### 静态资源
```shell
npm i url-loader file-loader -D
```
### scss预编译
```shell
npm i sass-loader node-sass -D
```
### 开发server
```shell
npm i webpack-dev-server html-webpack-plugin -D
```
### postcss
```shell
npm i postcss-loader autoprefixer -D
```

### Babel 7.x
```shell
# 基本配置
npm i @babel/core @babel/preset-env @babel/polyfill @babel/runtime @babel/plugin-transform-runtime @babel/runtime-corejs2 -D
# .vue开发环境
npm i babel-plugin-transform-vue-jsx babel-helper-vue-jsx-merge-props babel-plugin-syntax-jsx babel-plugin-dynamic-import-webpack -D
# webpack环境的loader
npm i babel-loader -D
```

### css分离
```shell
npm i mini-css-extract-plugin -D
```
### 跨平台设置环境参数与删除文件工具
```shell
npm i cross-env rimraf -D
```
### 前端gzip压缩
```shell
npm i compression-webpack-plugin -D
```


## 代码规范

### ESlint
#### `.js`检测
```shell
npm i eslint eslint-config-standard eslint-plugin-standard eslint-plugin-promise eslint-plugin-import eslint-plugin-node -D
```
#### `.vue`检测
```shell
npm i eslint-plugin-vue -D
```
#### 根目录下创建`.eslintrc`文件，配置如下：
```json
{
  "extends": [
    "standard",
    "plugin:vue/recommended"
  ],
  "parserOptions": {
    "parser": "babel-eslint",
    "sourceType": "module"
  }
}
```
#### `package.json`中添加执行检测命令和检测并自动修复命令
```json
"scripts": {
  "lint": "eslint --ext .js --ext .jsx --ext .vue src/",
  "lint": "eslint --fix --ext .js --ext .jsx --ext .vue src/"
}
```
#### 编码时检测
`babel-eslint`用途是我们代码都是经过babel编译的，有些语法不是标准的eslint规范，因此需要指定parser为`babel-eslint`。
```shell
npm i eslint-loader babel-eslint -D
```
`.eslintrc`文件修改如下
```json
{
  "extends": [
    "standard",
    "plugin:vue/recommended"
  ],
  "parserOptions": {
    "parser": "babel-eslint",
    "sourceType": "module"
  }
}
```
配置`eslint-loader`
```js
rules: [
  {
    test: /\.(vue|js|jsx)$/,
    loader: 'eslint-loader',
    exclude: /node_modules/,
    enforce: 'pre' //预处理，在其他loader（如vue-loader）处理前先处理
  }
]
```
#### 忽略指定文件，例如忽略dist文件夹
创建`.eslintignore`，配置如下
```
dist
```

### EditorConfig
编辑器插件，让不同编辑器下编写代码用相同的规则处理。

根目录创建`.editorconfig`文件，配置如下：
```
root = true

[*]
charset = utf-8
end_of_line = 1f
indent_size = 2
indent_style = space
insert_final_newline = true
trim_trailing_whitespace = true
```

### precommit
git提交前进行eslint检测，保证不合规范的代码无法提交到远程库。注意，这是针对git提交，因此我们要先保证目录下已经完成git初始化（git init），会在.git文件夹下的hooks目录下生成各种文件。关于git钩子插件的更多介绍可以看[这里](https://www.jianshu.com/p/f0d31f92bfab)。

```shell
npm i husky -D
```
`package.json`修改为：
```json
"scripts": {
  "lint": "eslint --ext .js --ext .jsx --ext .vue src/",
  "lint-fix": "eslint --fix --ext .js --ext .jsx --ext .vue src/",
  "precommit": "npm run lint-fix"
}
```

# 后记