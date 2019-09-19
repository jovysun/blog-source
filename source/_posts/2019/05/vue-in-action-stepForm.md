---
title: Vue实战@分步表单
permalink: vue-in-action-stepForm
tags:
  - Vue
categories:
  - 框架与库
  - Vue
date: 2019-05-24 19:53:26
updated: 2019-05-24 19:53:26
---

# 概述

假如该分步表单有三步，对应组件为 `Step1.vue`，`Step2.vue`，`Step3.vue`，第一步需要填写收款账号，第二步需要填写用户密码，可以回到上一步修改收款账号，可以直接填写密码提交后端接口进行数据可视化，成功后跳到第三步显示成功信息。主要知识点有 vuex 和编程式的导航。[配套测试源码](https://github.com/jovysun/Vue-my-pro)
![效果图](step-form.gif)

<!-- more -->

# 详述

## 创建 store，vuex 相关配置

```js
// src/store/modules/form.js
import router from "../../router";
import request from "../../utils/request";

const state = {
  step: {
    payAccount: "123456"
  }
};

const actions = {
  async submitStepForm({ commit }, payload) {
    console.log(payload);
    await request({
      url: "/api/form",
      method: "POST",
      data: payload
    });
    commit("saveStepFormData", payload);
    router.push("/form/step-form/result");
  }
};

const mutations = {
  saveStepFormData(state, payload) {
    state.step = {
      ...state.step,
      ...payload
    };
  }
};

export default {
  namespaced: true,
  state,
  actions,
  mutations
};
```

```js
// src/store/index.js

import Vue from "vue";
import Vuex from "vuex";
import form from "./modules/form";

Vue.use(Vuex);

export default new Vuex.Store({
  state: {},
  modules: {
    form
  }
});
```

## 第一步

调用 vuex 的 `mutations` 的 `saveStepFormData` 方法把数据保存到 `state`，然后路由跳转到第二步页面。

```html
<!-- Step1.vue -->
<script>
  export default {
    data() {
      return {
        formItemLayout: {
          labelCol: { span: 4 },
          wrapperCol: { span: 14 }
        },
        form: this.$form.createForm(this)
      };
    },
    computed: {
      step() {
        return this.$store.state.form.step;
      }
    },
    methods: {
      handleSubmit() {
        const { form, $router, $store } = this;
        form.validateFields((err, values) => {
          if (!err) {
            $store.commit("form/saveStepFormData", values);
            $router.push("/form/step-form/confirm");
          }
        });
      }
    }
  };
</script>
```

## 第二步

调用 `actions` 的 `submitStepForm` 方法，异步请求后端接口进行数据持久化，然后更新 `state`，最后路由跳转到成功页。

```html
<script>
  export default {
    data() {
      return {
        formItemLayout: {
          labelCol: { span: 4 },
          wrapperCol: { span: 14 }
        },
        form: this.$form.createForm(this)
      };
    },
    computed: {
      step() {
        return this.$store.state.form.step;
      }
    },
    methods: {
      handleSubmit() {
        const { form, $store, step } = this;
        form.validateFields((err, values) => {
          if (!err) {
            $store.dispatch("form/submitStepForm", { ...step, ...values });
          }
        });
      },
      handlePre() {
        this.$router.push("/form/step-form/info");
      }
    }
  };
</script>
```

# 参考

极客时间——Vue 开发实战
