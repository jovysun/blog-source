---
title: Vue实战@高扩展性路由
permalink: vue-in-action-router
tags:
  - Vue
categories:
  - 框架与库
  - Vue
date: 2019-05-21 11:39:53
updated: 2019-05-21 11:39:53
---

# 概述

一个高扩展性的路由应该是根据页面展示结构的特点进行的抽象，再结合业务模块进行的合理层级划分。本次总结是根据中后台管理系统[Ant Design Pro](https://preview.pro.ant.design/dashboard/analysis)。[配套测试源码](https://github.com/jovysun/Vue-my-pro)

<!-- more -->

# 详述

整体来说没什么难点，主要对新手有个目录结构案例参考，其次对于单纯提供 `router-view` 的组件可以用函数式组件`component: { render: h => h("router-view") }`。

## 第一层级

第一层级应该是布局模板，例如登录注册页面与其他的内容展示页，通常页面布局结构是不同的，因此可以分成两个 UserLayout.vue 和 BasicLayout.vue。

## 第二层级

一级业务模块，直观的就是对应的一级菜单，例如 Dashboard、个人页等。

## 其他层级

按照业务模块层层嵌套，例如 Dashboard 下面有 分析页 和 监控页等。

## 示例代码

```js
const router = new Router({
  mode: "history",
  base: process.env.BASE_URL,
  routes: [
    {
      path: "/user",
      hideInMenu: true,
      component: () =>
        import(/* webpackChunkName: "user" */ "./layouts/UserLayout.vue"),
      children: [
        {
          path: "/user",
          redirect: "/user/login"
        },
        {
          path: "/user/login",
          name: "login",
          component: () =>
            import(/* webpackChunkName: "user" */ "./views/User/Login.vue")
        },
        {
          path: "/user/register",
          name: "register",
          component: () =>
            import(/* webpackChunkName: "user" */ "./views/User/Register.vue")
        }
      ]
    },
    {
      path: "/",
      meta: { authority: ["user", "admin"] },
      component: () =>
        import(/* webpackChunkName: "basic" */ "./layouts/BasicLayout.vue"),
      children: [
        {
          path: "/",
          redirect: "/dashboard/analysis"
        },
        // dashboard
        {
          path: "/dashboard",
          name: "dashboard",
          meta: { icon: "dashboard", title: "仪表盘" },
          component: { render: h => h("router-view") },
          children: [
            {
              path: "/dashboard/analysis",
              name: "analysis",
              meta: { title: "分析页" },
              component: () =>
                import(
                  /* webpackChunkName: "dashboard" */ "./views/Dashboard/Analysis.vue"
                )
            }
          ]
        },
        // form
        {
          path: "/form",
          name: "form",
          meta: { icon: "form", title: "表单", authority: ["admin"] },
          component: { render: h => h("router-view") },
          children: [
            {
              path: "/form/basic-form",
              name: "basic-form",
              meta: { title: "基础表单" },
              component: () =>
                import(
                  /* webpackChunkName: "form" */ "./views/Forms/BasicForm.vue"
                )
            },
            {
              path: "/form/step-form",
              name: "step-form",
              hideChildrenInMenu: true,
              meta: { title: "分布表单" },
              component: () =>
                import(
                  /* webpackChunkName: "form" */ "./views/Forms/StepForm.vue"
                ),
              children: [
                {
                  path: "/form/step-form",
                  redirect: "/form/step-form/info"
                },
                {
                  path: "/form/step-form/info",
                  name: "info",
                  component: () =>
                    import(
                      /* webpackChunkName: "form" */ "./views/Forms/StepForm/Step1"
                    )
                },
                {
                  path: "/form/step-form/confirm",
                  name: "confirm",
                  component: () =>
                    import(
                      /* webpackChunkName: "form" */ "./views/Forms/StepForm/Step2"
                    )
                },
                {
                  path: "/form/step-form/result",
                  name: "result",
                  component: () =>
                    import(
                      /* webpackChunkName: "form" */ "./views/Forms/StepForm/Step3"
                    )
                }
              ]
            }
          ]
        }
      ]
    }
  ]
});
```

# 参考

极客时间——Vue 开发实战
