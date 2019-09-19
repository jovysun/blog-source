---
title: Vue实战@axios二次封装
permalink: vue-in-action-axios
tags:
  - Vue
categories:
  - 框架与库
  - Vue
date: 2019-05-22 21:02:59
updated: 2019-05-22 21:02:59
---

# 概述

axios 二次封装的相关总结。[配套测试源码](https://github.com/jovysun/Vue-my-pro)

<!-- more -->

# 详述

## 基础实现

之所以需要二次封装，因为正常情况下，我们会针对错误之类的作统一处理，除了错误还有请求头，url 等，详细可参考 axios [官方文档](http://www.axios-js.com/)。

```js
// utils>request.js
import axios from "axios";
import { notification } from "ant-design-vue";

function request(options) {
  return axios(options)
    .then(res => {
      return res;
    })
    .catch(error => {
      const {
        response: { status, statusText }
      } = error;
      notification.error({
        // jsx语法美化提示信息展示样式
        message: h => (
          <div>
            请求错误 <span style="color: red">{status}</span> : {options.url}
          </div>
        ),
        description: statusText
      });
      return Promise.reject(error);
    });
}

export default request;
```

## 特别提示

示例中 notification 的 message 部分使用了 jsx 语法，有关 vue 中使用 jsx 的方法如下：

1. 首先需要安装`@vue/babel-preset-jsx`和`@vue/babel-helper-vue-jsx-merge-props`；
2. 然后修改 babel 配置`presets: ["@vue/app", "@vue/babel-preset-jsx"],`；
3. 最后在 vue 组件中使用，如上示例代码 message 部分。

## 进阶实现

以下是摘自[奇舞周刊](https://mp.weixin.qq.com/s/eqgf9MLEcLrooBhzgCVV6A)公众号。

```js
import axios from "axios";
import router from "../router";
import { MessageBox, Message } from "element-ui";
let loginUrl = "/login";
// 根据环境切换接口地址
axios.defaults.baseURL = process.env.VUE_APP_API;
axios.defaults.headers = { "X-Requested-With": "XMLHttpRequest" };
axios.defaults.timeout = 60000;
// 请求拦截器
axios.interceptors.request.use(
  config => {
    if (router.history.current.path !== loginUrl) {
      let token = window.sessionStorage.getItem("token");
      if (token == null) {
        router.replace({
          path: loginUrl,
          query: { redirect: router.currentRoute.fullPath }
        });
        return false;
      } else {
        config.headers["Authorization"] = "JWT " + token;
      }
    }
    return config;
  },
  error => {
    Message.warning(error);
    return Promise.reject(error);
  }
);
// 响应拦截器
axios.interceptors.response.use(
  response => {
    return response.data;
  },
  error => {
    if (error.response !== undefined) {
      switch (error.response.status) {
        case 400:
          MessageBox.alert(error.response.data);
          break;
        case 401:
          if (window.sessionStorage.getItem("out") === null) {
            window.sessionStorage.setItem("out", 1);
            MessageBox.confirm("会话已失效! 请重新登录", "提示", {
              confirmButtonText: "重新登录",
              cancelButtonText: "取消",
              type: "warning"
            })
              .then(() => {
                router.replace({
                  path: loginUrl,
                  query: { redirect: router.currentRoute.fullPath }
                });
              })
              .catch(action => {
                window.sessionStorage.clear();
                window.localStorage.clear();
              });
          }
          break;
        case 402:
          MessageBox.confirm("登陆超时 ！", "提示", {
            confirmButtonText: "重新登录",
            cancelButtonText: "取消",
            type: "warning"
          }).then(() => {
            router.replace({
              path: loginUrl,
              query: { redirect: router.currentRoute.fullPath }
            });
          });
          break;
        case 403:
          MessageBox.alert("没有权限！");
          break;
        // ...忽略
        default:
          MessageBox.alert(`连接错误${error.response.status}`);
      }
      return Promise.resolve(error.response);
    }
    return Promise.resolve(error);
  }
);
// 导出基础请求类型封装
export default {
  get(url, param) {
    if (param !== undefined) {
      Object.assign(param, { _t: new Date().getTime() });
    } else {
      param = { _t: new Date().getTime() };
    }
    return new Promise((resolve, reject) => {
      axios({ method: "get", url, params: param }).then(res => {
        resolve(res);
      });
    });
  },
  getData(url, param) {
    return new Promise((resolve, reject) => {
      axios({ method: "get", url, params: param }).then(res => {
        if (res.code === 4000) {
          resolve(res.data);
        } else {
          Message.warning(res.msg);
        }
      });
    });
  },
  post(url, param, config) {
    return new Promise((resolve, reject) => {
      axios.post(url, param, config).then(res => {
        resolve(res);
      });
    });
  },
  put: axios.put,
  _delete: axios.delete
};
```

# 参考

极客时间——Vue 开发实战

奇舞周刊
