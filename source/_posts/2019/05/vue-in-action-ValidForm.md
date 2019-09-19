---
title: Vue实战@表单校验
permalink: vue-in-action-ValidForm
tags:
  - Vue
categories:
  - 框架与库
  - Vue
date: 2019-05-24 09:31:44
updated: 2019-05-24 09:31:44
---

# 概述

表单校验很重要，分为自定义校验和动态校验。自定义校验就是自己去监听值变化来改变 dom 显示样式。动态校验就是利用框架封装的校验规则去使用。本次以 ant-design-vue 组件库中使用为例。[配套测试源码](https://github.com/jovysun/Vue-my-pro)

<!-- more -->

# 详述

## 自定义校验

要点就是自己主动监听表单项的变化，然后对应的去修改 UI 组件暴露的属性值，通过不同值来显示不同的状态（错误、警告、成功等），例如 ant-design-vue 主要是控制 validate-status 和 help 两个属性，前者对应显示样式，后者对应提示文案。示例如下：

```html
<a-form-item label="用户名" :validate-status="fieldAStatus" :help="fieldAHelp">
  <a-input placeholder="用户名" v-model="fieldA" />
</a-form-item>
```

```js
methods: {
  submitHandle() {
    if (this.fieldA.length < 6) {
      this.fieldAStatus = "error";
      this.fieldAHelp = "不能小于6个字符";
    } else {
      this.fieldAStatus = "";
      this.fieldAHelp = "";
    }
  }
}
```

## 动态校验

要点就是按照使用的 UI 组件库规则来编写代码就可以了。例如 ant-design-vue，示例如下：

```html
<template>
  <div>
    <a-form :form="form">
      <a-form-item label="付款账户">
        <a-input
          v-decorator="['payAccount',{rules: [{required: true, message: '请输入付款账号'}]}]"
        />
      </a-form-item>
      <a-form-item>
        <a-button type="primary" @click="check">下一步</a-button>
      </a-form-item>
    </a-form>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        form: this.$form.createForm(this)
      };
    },
    methods: {
      check() {
        this.form.validateFields(err => {
          if (!err) {
            console.info("success");
          }
        });
      }
    }
  };
</script>
```

# 参考

极客时间——Vue 开发实战
