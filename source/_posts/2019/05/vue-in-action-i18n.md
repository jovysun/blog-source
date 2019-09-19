---
title: Vue实战@国际化
permalink: vue-in-action-i18n
tags:
  - Vue
categories:
  - 框架与库
  - Vue
date: 2019-05-28 19:38:25
updated: 2019-05-28 19:38:25
---

# 概述

国际化，包括组件库的国际化、第三库的国际化和本地业务的国际化。[配套测试源码](https://github.com/jovysun/Vue-my-pro)
测试示例效果图：
![overview](overview.jpg)

<!-- more -->

# 详述

## 组件库的国际化

以 ant-design-vue 库为例，针对国际化提供了 LocaleProvider 组件，具体使用如下：

### 全局注册组件

```js
import { LocaleProvider } from "ant-design-vue";
Vue.use(LocaleProvider);
```

### App.vue 中使用

```html
<template>
  <div id="app">
    <a-locale-provider :locale="locale">
      <router-view />
    </a-locale-provider>
  </div>
</template>

<style></style>
<script>
  import zh_CN from "ant-design-vue/lib/locale-provider/zh_CN";
  import en_US from "ant-design-vue/lib/locale-provider/en_US";
  export default {
    data() {
      return {
        locale: {
          type: Object,
          default: en_US
        }
      };
    },
    watch: {
      "$route.query.locale": function(val) {
        this.locale = val === "en_US" ? en_US : zh_CN;
      }
    }
  };
</script>
```

对于 locale 值的动态获取，本示例是通过路由的 query 实现的。例如本示例：

```html
<template>
  <div>
    <a-layout-header style="background: #fff; padding: 0">
      <a-dropdown style="float:right;margin-right:30px;">
        <a class="ant-dropdown-link" href="javascript:void(0)">
          <a-icon type="global" />
        </a>
        <a-menu slot="overlay" @click="onClick">
          <a-menu-item key="zh_CN">中文</a-menu-item>
          <a-menu-item key="en_US">English</a-menu-item>
        </a-menu>
      </a-dropdown>
    </a-layout-header>
  </div>
</template>

<script>
  export default {
    methods: {
      onClick({ key }) {
        this.$router.push({ query: { locale: key } });
      }
    }
  };
</script>
<style scoped></style>
```

组件库国际化完成后，测试效果图：
![效果图](1.jpg)

我们发现日期选择组件中并没有完全国际化，原因就是组件库引用了第三方的 moment 库，而 moment 库有自己的国际化方式，说明见下面“第三方库的国际化”。

## 第三方库的国际化

针对`moment`库的国际化，在上段代码基础上修改如下：

```html
<!-- src/App.vue -->
<script>
  import zh_CN from "ant-design-vue/lib/locale-provider/zh_CN";
  import en_US from "ant-design-vue/lib/locale-provider/en_US";
  import moment from "moment";
  import "moment/locale/zh-cn";
  export default {
    data() {
      return {
        locale: {
          type: Object,
          default: en_US
        }
      };
    },
    watch: {
      "$route.query.locale": function(val) {
        this.locale = val === "en_US" ? en_US : zh_CN;
        moment.locale(val === "en_US" ? "en" : "zh-cn");
      }
    }
  };
</script>
```

## 本地业务的国际化

推荐用[Vue I18n](https://kazupon.github.io/vue-i18n/)，本次也以该插件为例。

例如上面的效果图，日期选择组件有个 label 的文案“time”，我们对它进行国际化。

首先，本地创建国际化文件，例如：

```js
// src/local/zhCN.js
export default {
  "app.dashboard.analysis.timeLabel": "时间"
};
```

```js
// src/local/enUS.js
export default {
  "app.dashboard.analysis.timeLabel": "Time"
};
```

然后，安装相关包文件：

```shell
# query-string 是方便解析URL查询字符串
npm install vue-i18n query-string -S
```

再然后，在 main.js 中进行注册配置：

```js
// 引入相关包
import VueI18n from "vue-i18n";
import enUS from "./locale/enUS";
import zhCN from "./locale/zhCN";
import queryString from "query-string";
// 注册组件
Vue.use(VueI18n);
// 创建一个实例
const i18n = new VueI18n({
  locale: queryString.parse(location.search).locale || "zh_CN",
  messages: {
    zh_CN: { message: zhCN },
    en_US: { message: enUS }
  }
});
// 挂载到Vue实例
new Vue({
  i18n,
  router,
  store,
  render: h => h(App)
}).$mount("#app");
```

最后，在切换语言的地方，添加动态修改 locale：

```js
  onClick({ key }) {
    this.$router.push({query: {locale: key}});
    // 动态修改locale
    this.$i18n.locale = key;
  }
```

最终效果：
![效果图](2.jpg)

# 参考

极客时间——Vue 开发实战
