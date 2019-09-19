---
title: Vue重构jQuery项目经验总结一
permalink: Vue-restructure-jQuery-1
tags:
  - Vue
  - jQuery
categories:
  - 框架与库
  - Vue
date: 2019-08-09 09:28:02
updated: 2019-08-09 09:28:02
---

# 概述

用 Vue 生态重构已有的老项目（jsp+jquery），针对目前实践中遭遇的一些坑，做下小结。具体包括：URL 转换规则、element-ui 自定义校验、`el-table`组件删除问题、自定义组件`v-model`使用和`sync`修饰符。

<!-- more -->

# 详述

初始化项目直接用的 vue-cli，安装全家桶（router、vuex），代码规范选用的 Prettier，配置文件选择放在单独文件，ui 库用 element-ui。

## URL 转换规则

对于资源的引用路径，可以用相对路径，也可以用绝对路径，但是实际中，对于嵌套太深的一般使用绝对路径比较合适。在 vue-cli 初始化的项目中默认设置了`@`别名作为 src 根目录，当然我们也可以在 webpack 配置文件中设置更多的别名。

在 js 中引用资源的相对路径与绝对路径示例如下：

```js
// 简洁
import MainTitle from "@/components/MainTitle";
// 复杂，容易搞错层级
import MainTitle from "../../../components/MainTitle";
```

既然使用别名这么方便，那么在 css 中引用图片也这样搞吧:

```scss
.select-flag {
  &:before {
    content: "";
    width: 16px;
    height: 11px;
    display: inline-block;
    margin-right: 5px;
    font-size: 0;
    background: url("@/assets/milan/common/img/icon/flags.png") no-repeat;
  }
}
```

结果报错如下：

```
Failed to compile.

./src/components/CountrySelect/index.vue?vue&type=style&index=0&id=432f6435&lang=scss&scoped=true& (./node_modules/css-loader??ref--8-oneOf-1-1!./node_modules/vue-loader/lib/loaders/stylePostLoader.js!./node_modules/postcss-loader/src??ref--8-oneOf-1-2!./node_modules/sass-loader/lib/loader.js??ref--8-oneOf-1-3!./node_modules/cache-loader/dist/cjs.js??ref--0-0!./node_modules/vue-loader/lib??vue-loader-options!./src/components/CountrySelect/index.vue?vue&type=style&index=0&id=432f6435&lang=scss&scoped=true&)
Module not found: Error: Can't resolve './@/assets/milan/common/img/icon/flags.png' in 'H:\workspace\vue\venice\src\components\CountrySelect'
```

细看这个报错信息，可知 css-loader 编译时没有找到引用的资源。解决方式为：`background: url("~@/assets/milan/common/img/icon/flags.png") no-repeat;`。原因是，`@`与`~`开头都会作为一个模块请求被解析，但是`@`开头仅作用域模板中。详细见[官网](https://cli.vuejs.org/zh/guide/html-and-static-assets.html#url-%E8%BD%AC%E6%8D%A2%E8%A7%84%E5%88%99)。

## element-ui 自定义校验

element 的表单校验与 ant-d 一样都是用的第三方库[async-validator
](https://github.com/yiminghe/async-validator/blob/master/README.md)。因为 element-ui 官网这块只是一带而过，详细的还是要查看该集成库的文档。下面以自定义同步校验与异步校验为例：

```js
  ...
  data() {

    var phoneRepeat = (rule, value, callback) => {
      request({
        url: "/api/account/phoneRepeat",
        method: "post",
        params: { regionId: this.ruleForm.regionId, phone: value }
      }).then(response => {
        let result = response.data;
        if (result.code == "10002") {
          callback(new Error(rule.message));
        } else {
          callback();
        }
      });
    };
    var telNumCn = (rule, value, callback) => {
      var reg = /^1\d{10}$/;
      if (this.ruleForm.regionId === "China" && !reg.test(value)) {
        callback(new Error(rule.message));
      } else {
        callback();
      }
    };

    return {
      titleTxt: "管理子账户 ",
      titleDesc: "仅提供业务员身份的子账户，用于管理除资金以外的所有权限。",
      ruleForm: {
        type: "01",
        name: "",
        email: "",
        regionId: "China",
        phone: "",
        password: "",
        repeatpwd: ""
      },
      rules: {
        phone: [
          {
            required: true,
            message: "请输入手机号码。",
            trigger: "blur"
          },
          {
            validator: telNumCn,
            required: true,
            message: "请提供有效的中国大陆手机号码。",
            trigger: "blur"
          },
          {
            validator: phoneRepeat,
            required: true,
            message: "此手机已被作为安全手机使用过，请更换。",
            trigger: "blur"
          }
        ]
      }
    };
  },
  ...
```

想着校验函数复用，一开始根据 element 示例，想着用一个高阶函数传入 message 生成 validator 函数。后来发现`rule`参数就是指向我们配置的规则对象，因此直接在 rules 中配置 message 即可，当然也可以增加其他参数供 validator 函数使用。

## `el-table`组件删除问题

问题很小，就是用了官方示例，删除成功，但是不是对应的那一条：
![效果图](GIF.gif)
检查代码：

```html
<el-table
  :data="tableData"
  style="width: 100%"
  :default-sort="{prop: 'name', order: 'descending'}"
>
  <el-table-column prop="name" label="姓名" sortable sort-by="name" />
  <el-table-column prop="type" label="类型" />
  <el-table-column prop="email" label="登录邮箱" />
  <el-table-column prop="phone" label="安全手机" />
  <el-table-column prop="time" label="添加时间" sortable sort-by="time" />
  <el-table-column label="操作" width="100">
    <template v-slot="scope">
      <el-button type="text" @click="handleEdit(scope.$index, scope.row)"
        >编辑</el-button
      >
      <el-button type="text" @click="handleDelete(scope.$index, scope.row)"
        >删除</el-button
      >
    </template>
  </el-table-column>
</el-table>
```

```js
    handleDelete(index, row) {
      ...
      this.tableData.splice(index, 1);
      ...
    }
```

没啥问题啊。唉！最后发现加了默认排序，导致展示的顺序与实际的数据顺序不一致，当然根据索引删除也就对不上号了。因此，有排序的，只能通过唯一性的字段，如id，来进行查询筛选了。

## 自定义组件 v-model 使用

一个组件上的 v-model 默认会利用名为 value 的 prop 和名为 input 的事件，但是像单选框、复选框等类型的输入控件可能会将 value 特性用于不同的目的。model 选项可以用来避免这样的冲突：

```HTML
<my-checkbox v-model="foo" value="some value"></my-checkbox>
```

```JS
Vue.component('my-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    // this allows using the `value` prop for a different purpose
    value: String,
    // use `checked` as the prop which take the place of `value`
    checked: {
      type: Number,
      default: 0
    }
  },
  // ...
})
```

## sync 修饰符

原以为加了 sync 的 prop 就可以像普通的 data 定义变量一样使用，结果当然是错的。其实只是一种针对通过自定义事件修改 prop 的一种简写。使用这种简写的前提是约定自定义事件名为`update:xxx`，其实只是通过约定，减少了传入事件函数，对于子组件还是需要调用`$emit`方法去触发。示例如下：

```html
<text-document
  v-bind:title="doc.title"
  v-on:update:title="doc.title = $event"
></text-document>
<!-- 可以简写成这样： -->
<text-document v-bind:title.sync="doc.title"></text-document>
```

```js
// 子组件中触发赋值意图
this.$emit("update:title", newTitle);
```
