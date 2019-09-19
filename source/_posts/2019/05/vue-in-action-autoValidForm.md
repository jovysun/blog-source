---
title: Vue实战@封装一个自动校验表单项
permalink: vue-in-action-autoValidForm
tags:
  - Vue
categories:
  - 框架与库
  - Vue
date: 2019-05-25 23:29:52
updated: 2019-05-25 23:29:52
---

# 概述

自己封装一个支持自动校验表单项，以`ant-design-vue`组件为例。[配套测试源码](https://github.com/jovysun/Vue-my-pro)

<!-- more -->

# 详述

## 自定义一个组件 ReceiverAccount

```html
<template>
  <a-input-group compact>
    <a-select style="width: 130px;" v-model="type" @change="handleTypeChange">
      <a-select-option value="aplipay">支付宝</a-select-option>
      <a-select-option value="bank">银行账户</a-select-option>
    </a-select>
    <a-input
      style="width: calc(100% - 130px)"
      v-model="number"
      @change="handleNumberChange"
    />
  </a-input-group>
</template>

<script>
  export default {
    props: {
      value: {
        type: Object
      }
    },
    watch: {
      value(val) {
        Object.assign(this, val);
      }
    },
    data() {
      const { type, number } = this.value || {};
      return {
        type: type || "aplipay",
        number: number || ""
      };
    },
    methods: {
      handleTypeChange(val) {
        this.$emit("change", { ...this.value, type: val });
      },
      handleNumberChange(e) {
        this.$emit("change", { ...this.value, number: e.target.value });
      }
    }
  };
</script>
```

## 使用自定义组件 ReceiverAccount

```html
<template>
  <div>
    <a-form :form="form" layout="horizontal">
      <a-form-item
        label="付款账户"
        :label-col="formItemLayout.labelCol"
        :wrapper-col="formItemLayout.wrapperCol"
      >
        <a-input
          v-decorator="[
            'payAccount', 
            {
              initialValue: step.payAccount,
              rules: [{required: true, message: '请输入付款账号'}]
            }
          ]"
        />
      </a-form-item>
      <a-form-item
        label="收款账户"
        :label-col="formItemLayout.labelCol"
        :wrapper-col="formItemLayout.wrapperCol"
      >
        <ReceiverAccount
          v-decorator="[
            'receiverAccount', 
            {
              initialValue: step.receiverAccount,
              rules: [{required: true, message: '请输入收款账号', validator: checkReceiverAccount}]
            }
          ]"
        />
      </a-form-item>
      <a-form-item>
        <a-button type="primary" @click="handleSubmit">
          下一步
        </a-button>
      </a-form-item>
    </a-form>
  </div>
</template>

<script>
  import ReceiverAccount from "@/components/ReceiverAccount";
  export default {
    components: {
      ReceiverAccount
    },
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
      },
      checkReceiverAccount(rule, value, callback) {
        if (value && value.number) {
          callback();
        } else {
          callback(false);
        }
      }
    }
  };
</script>
```

# 参考

极客时间——Vue 开发实战
