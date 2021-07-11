---
title: vue-动态&异步组件理解
date: 2021-06-08 16:40:39
tags: [动态组件, async-component]
# type: "categories"
categories: [大前端, Vue]
---

### 动态组件基础用法

` 指在不同组件之间进行动态切换，在 is 属性发生改变时，组件重新渲染`

- is: 已注册组件的名字 或 一个组件的选项对象

```vue
<template>
  <div id="dynamic-component-demo" class="demo">
    <button
      v-for="tab in tabs"
      v-bind:key="tab"
      v-bind:class="['tab-button', { active: currentTab === tab }]"
      v-on:click="currentTab = tab"
    >
      {{ tab }}
    </button>
    <!-- 组件会在 `currentTabComponent` 改变时改变 -->
    <component v-bind:is="currentTabComponent" class="tab"></component>
  </div>
</template>

<script>
Vue.component("tab-home", {
  template: "<div>Home component</div>",
});
Vue.component("tab-posts", {
  template: "<div>Posts component</div>",
});
Vue.component("tab-archive", {
  template: "<div>Archive component</div>",
});

new Vue({
  el: "#dynamic-component-demo",
  data: {
    currentTab: "Home",
    tabs: ["Home", "Posts", "Archive"],
  },
  computed: {
    currentTabComponent: function () {
      return "tab-" + this.currentTab.toLowerCase();
    },
  },
});
</script>
```

### 动态组件缓存

`部分场景下，需要对动态创建的组件做缓存`

```html
<!-- 失活的组件将会被缓存！keep-alive 包裹的组件 必须拥有组件 name -->
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

### 动态组件与常规组件比较

- 常规组件通过 v-if、v-else 切换模板的显示，动态组件通过 is 属性的变化，切换模板的显示，都拥有 props 及 methods。
- 动态组件模板更加简洁灵活，只需改变 is 属性值，即可完成对应组件的切换。

### 异步组件

`又指组件的动态导入，Vue 允许你以一个工厂函数的方式定义你的组件，这个工厂函数会异步解析你的组件定义，接收 resolve 以及 reject 回调，当然也可以直接返回 Promise`

```typescript
// 动态导入
Vue.component(
  'async-webpack-example',
  // 这个动态导入会返回一个 `Promise` 对象。
  () => import('./my-async-component'),
);

// 局部导入
@Component({ components: { 'my-component': () => import('./my-async-component') } })
```
