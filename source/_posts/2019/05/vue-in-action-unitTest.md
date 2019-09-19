---
title: Vue实战@单元测试
permalink: vue-in-action-unitTest
tags:
  - Vue
categories:
  - 框架与库
  - Vue
date: 2019-05-30 08:55:08
updated: 2019-05-30 08:55:08
---

# 概述

单元测试在日常开发中可能使用率不高，但是对于功能性的模块，开源的项目就显得非常必要了。Vue CLI 拥有开箱即用的通过 Jest 或 Mocha 进行单元测试的内置选项。本次我们在用 cli 建项目的时候选择了 Jest 方式，因此本文说下在 Vue CLI 构建的项目中 Jest 的使用。[配套测试源码](https://github.com/jovysun/Vue-my-pro)

<!-- more -->

# 详述

## 待测代码

```js
// src/auth.js
const currentAuth = ["admin"];
export { currentAuth };

export function getCurrentAuthority() {
  return currentAuth;
}

export function check(authority) {
  const current = getCurrentAuthority();
  return current.some(item => authority.includes(item));
}

export function isLogin() {
  const current = getCurrentAuthority();
  return current && current[0] !== "guest";
}
```

## 执行单测命令

```shell
npm run test:unit -- --watch
```

## 编写单测脚本

在 tests/unit 下新建文件 auth.spec.js，然后编写脚本，简单的可以参考 example.spec.js 代码，详细的看 Jest[官方文档](https://jestjs.io/docs/en/getting-started)。本次示例代码：

```js
import { check, currentAuth } from "../../src/utils/auth";

// auth测试组
describe("auth test", () => {
  it("empty auth", () => {
    // 清空currentAuth
    currentAuth.splice(0, currentAuth.length);
    expect(check(["user"])).toBeFalsy();
    expect(check(["admin"])).toBeFalsy();
  });

  it("user auth", () => {
    // 清空currentAuth
    currentAuth.splice(0, currentAuth.length);
    currentAuth.push("user");
    expect(check(["user"])).toBeTruthy();
    expect(check(["admin"])).toBeFalsy();
  });

  it("admin auth", () => {
    // 继续添加admin角色
    currentAuth.push("admin");
    expect(check(["user"])).toBeTruthy();
    expect(check(["admin"])).toBeTruthy();
    expect(check(["user", "admin"])).toBeTruthy();
  });
});
```