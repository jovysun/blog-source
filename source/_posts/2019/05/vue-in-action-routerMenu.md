---
title: Vue实战@根据路由生成菜单
permalink: vue-in-action-routerMenu
tags:
  - Vue
categories:
  - 框架与库
  - Vue
date: 2019-05-21 15:02:51
updated: 2019-05-21 15:02:51
---

# 概述

整体思路就是在每个路由下面增加 meta 属性维护需要在菜单显示的数据，例如`meta: { icon: "dashboard", title: "仪表盘" }`，以及加些标志位区分需要过滤掉的路由，然后遍历`routes`获取相关数据，最后页面渲染显示。[配套测试源码](https://github.com/jovysun/Vue-my-pro)

<!-- more -->

# 详述

## 路由配置参考

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
    },
    {
      path: "/403",
      name: "403",
      hideInMenu: true,
      component: Fobidden
    },
    {
      path: "*",
      name: "404",
      hideInMenu: true,
      component: NotFound
    }
  ]
});
```

## 提取菜单显示所需信息

```js
getMenuData(routes = [], parentKeys = [], selectedKey) {
  const menuData = [];
  routes.forEach(element => {
    if (element.name && !element.hideInMenu) {
      this.openKeysMap[element.path] = parentKeys;
      this.selectedKeysMap[element.path] = [selectedKey || element.path];
      const newItem = { ...element };
      delete newItem.children;
      if (element.children && !element.hideChildrenInMenu) {
        newItem.children = this.getMenuData(element.children, [
          ...parentKeys,
          element.path
        ]);
      } else {
        this.getMenuData(
          element.children,
          selectedKey ? parentKeys : [...parentKeys, element.path],
          selectedKey || element.path
        );
      }
      menuData.push(newItem);
    } else if (
      !element.hideInMenu &&
      !element.hideChildrenInMenu &&
      element.children
    ) {
      menuData.push(
        ...this.getMenuData(element.children, [...parentKeys, element.path])
      );
    }
  });
  return menuData;
}
```

# 参考

极客时间——Vue 开发实战
