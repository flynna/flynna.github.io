---
title: vue-路由守卫场景及用法
date: 2021-06-07 10:25:02
tags: [路由守卫]
# type: "categories"
categories: [大前端, Vue]
---

### 全局路由守卫

- `beforeEach`(主) 、`afterEach`、`beforeResolve`2.5.0+ 似 `beforeEach` ([区别](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html#%E5%85%A8%E5%B1%80%E8%A7%A3%E6%9E%90%E5%AE%88%E5%8D%AB))

`通常情况全局 beforeEach 使用居多，完成系统访问前的一些逻辑`

```typescript
import { transform2IntegrationEnv } from "@/store/modules/global";
import { SET_SUPRRESS_LAYOUT } from "@/store/modules/global/types";
import { GET_ME, GET_COMMON_CONFIG } from "@/store/modules/user/types";
import store from "../store";
import access from "./access"; // 权限文件
const router = new VueRouter({ routes });

router.beforeEach(async (to, from, next) => {
  /* 访问 403 页面(添加 meta标识)， 不做任何处理 */
  if (to.meta.errorPage) {
    next();
    return;
  }

  /* 场景1：读取 yml 配置，修改系统基础信息 如 logo */
  if (!store.state.commonConfig) {
    await store.dispatch(GET_COMMON_CONFIG);
  }

  /* 需要登录才能访问的页面 */
  if (!to.meta.allowAnonymous) {
    /* 场景2：页面刷新，vuex 用户信息丢失 */
    if (!store.state.user?.me) {
      await store.dispatch(GET_ME);
    }

    /* 场景3：根据当前登录用户的信息，对路由访问添加限制 */
    if (to.meta.access) {
      const canEnter = access()[to.meta.access]();
      if (!canEnter) {
        return next("/error/403");
      }
    }
  }

  /* 场景4：通过 query 给系统添加标识，供 iframe 集成 */
  if (to.query.suppressLayout !== undefined) {
    const integrationEnv = transformIntegrationEnv(suppressLayout);
    store.commit(SET_SUPRRESS_LAYOUT, integrationEnv);
  }
  if (store.state.global?.suppressLayout || suppressLayout) {
    window.parent.postMessage({ type: "page:mounted", payload: to.name }, "*");
  }

  next();
});
```

### 路由独享守卫

- `beforeEnter`

```typescript
const router = new VueRouter({
  routes: [
    {
      path: "/foo",
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ... 在配置路由时 即可配置 独享守卫
      },
    },
  ],
});
```

### 组件路由守卫

- `beforeRouteEnter`、`beforeRouteUpdate` (2.2+)、`beforeRouteLeave`

`部分情况下，需要通过组件内守卫完成特殊处理`

- `vue+js`

```vue
<template></template>
<script>
export default {
  beforeRouteEnter(to, from, next) {},
  beforeRouteUpdate(to, from, next) {},
  beforeRouteLeave(to, from, next) {},
};
</script>
```

- `vue+ts`
  `ts 中 引入class， 需要手动注册组件内的路由守卫`

```typescript
// @src/hooks/routeHooks.ts  vue3.0-
import Component from "vue-class-component";
Component.registerHooks([
  "beforeRouteEnter",
  "beforeRouteLeave",
  "beforeRouteUpdate",
]);

// vue3.0+
// vue-class-component组件中Component上未声明 registerHooks
// registerHooks 挂载到 Vue, 作为 Vue 的属性
import { Vue } from "vue-property-decorator";
Vue.registerHooks([
  "beforeRouteEnter",
  "beforeRouteLeave",
  "beforeRouteUpdate",
]);
```

### 其他

`除此之外，可以通过监听路由变化实现相关逻辑`

```typescript
watch('$route', { immediate: true })
public onRouteChange(to, from){}
```
