---
title: Vue实战@权限控制
permalink: vue-in-action-auth
tags:
  - Vue
categories:
  - 框架与库
  - Vue
date: 2019-05-21 16:55:07
updated: 2019-05-21 16:55:07
---

# 概述

在管理系统中权限控制是一个必不可少的组成部分，包括路由控制，菜单控制，内容区控制等。[配套测试源码](https://github.com/jovysun/Vue-my-pro)

<!-- more -->

# 详述

## 路由控制

### 辅助函数

首先可以在项目想创建 utils 文件夹，然后创建 auth.js 文件，编写需要的功能函数。

```js
export function getCurrentAuthority() {
  // 后端返回用户角色
  return ["user"];
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

### 导航守卫

配置路由导航守卫，控制前端路由跳转。

```js
router.beforeEach((to, from, next) => {
  if (to.path !== from.path) {
    NProgress.start();
  }
  const record = findLast(to.matched, item => item.meta.authority);
  if (record && !check(record.meta.authority)) {
    if (!isLogin() && to.path !== "/user/login") {
      next({
        path: "/user/login"
      });
    } else if (to.path !== "/403") {
      notification.error({
        message: "403",
        description: "你没有权限访问，请联系管理员咨询。"
      });
      next({
        path: "/403"
      });
    }
    NProgress.done();
  }
  next();
});

router.afterEach(() => {
  NProgress.done();
});
```

## 菜单控制

思路就是判断权限，过滤显示菜单内容。例如基于之前动态生成菜单的实现方式，可以如下处理：

```js
let menuData = [];
for (let element of routes) {
  if (
    element.meta &&
    element.meta.authority &&
    !check(element.meta.authority)
  ) {
    break;
  }
  ...
  return menuData;
}
```

## 精细化权限控制

### 权限组件

首选创建一个组件`Authorized.vue`，这里用函数式组件比较合适：

```js
<script>
import { check } from "../utils/auth";
export default {
  // 指定该组件为函数式组件
  functional: true,
  props: {
    authority: {
      type: Array,
      required: true
    }
  },
  render(h, context) {
    const { props, scopedSlots } = context;
    return check(props.authority) ? scopedSlots.default() : null;
  }
};
</script>
```

然后注册组件，推荐全局注册：

```js
import Authorized from "./components/Authorized";
Vue.component("Authorized", Authorized);
```

最后就可以在具体页面中使用了：

```html
<Authorized :authority="['admin']">
  <div>我是只有admin权限的用户才能看到的内容模块</div>
</Authorized>
```

### 权限指令

首先创建一个指令插件 `auth.js`：

```js
import { check } from "../utils/auth";
// 提供install方法（语法）
function install(Vue, options = {}) {
  // 定义指令
  Vue.directive(options.name || "auth", {
    // 钩子函数
    inserted(el, binding) {
      if (!check(binding.value)) {
        el.parentNode && el.parentNode.removeChild(el);
      }
    }
  });
}

export default { install };
```

然后全局注册插件：

```js
import Auth from "./directives/auth";
Vue.use(Auth);
```

最后就可以在需要的元素上使用该指令了：

```html
<a-icon
  v-auth="['admin']"
  class="trigger"
  :type="collapsed ? 'menu-unfold' : 'menu-fold'"
  @click="foldHandler"
/>
```

# 参考

极客时间——Vue 开发实战

