---
title: 基于 Qiankun 的微前端解决方案实践
date: 2023-10-17 16:37:08
updated: 2023-10-17 16:37:08
tags:
  - qiankun
categories:
  - 技术分享
  - 微前端
---

### 什么是微前端？

> `Techniques, strategies and recipes for building a modern web app with multiple teams that can ship features independently. -- Micro Frontends`
>
> 微前端是一种多个团队通过独立发布功能的方式来共同构建现代化 `web` 应用的技术手段及方法策略。

简单说就是：**将前端应用程序分解成多个小块，每个小块被称为微前端。每个微前端都是一个独立的部分，可以由不同的团队开发和维护。这些微前端可以独立部署，甚至可以使用不同的技术栈和框架开发。最终组合在一起呈现出完整的前端应用程序**

<!-- more -->

### 微前端能够解决哪些问题？

- 降低开发成本。应用程序日渐复杂，开发技术迭代，有许多历史已有项目功能需要在新项目中集成使用。

- 提升用户体验。模块分解后，主程序资源大小会大幅缩减，用户只有使用到其中某个功能模块时才会加载相应资源。

- 更方便业务集成扩展。微前端更具灵活性，可以根据需求添加新的微前端模块，同时不受技术栈限制。

### 为什么选择 `qiankun`？

> `qiankun` 是一个基于 `single-spa` 的微前端实现库，旨在帮助大家能更简单、无痛的构建一个生产可用微前端架构系统。

特性（摘自[官网](https://qiankun.umijs.org/zh/guide#%E7%89%B9%E6%80%A7)）：

- 📦 基于 `single-spa` 封装，提供了更加开箱即用的 `API`。

- 📱 技术栈无关，任意技术栈的应用均可 使用/接入，不论是 `React/Vue/- Angular/JQuery` 还是其他等框架。

- 💪 `HTML Entry` 接入方式，让你接入微应用像使用 `iframe` 一样简单。

- 🛡​ 样式隔离，确保微应用之间样式互相不干扰。

- 🧳 `JS` 沙箱，确保微应用之间 全局变量/事件 不冲突。

- ⚡️ 资源预加载，在浏览器空闲时间预加载未打开的微应用资源，加速微应用打开速度。

- 🔌 `umi` 插件，提供了 `@umijs/plugin-qiankun` 供 `umi` 应用一键切换成微前端架构系统。

### 了解 `single-spa`

> `single-spa` 是一个 `JavaScript` 前端微前端框架，它允许您构建和组织多个独立的前端应用程序（微前端）以实现单一页面应用程序（`SPA`）的集成

它做的事情其实就是: **注册一个微应用 ---> 监听 `URL` 变化 ---> 加载微应用 ---> 渲染微应用 ---> 卸载微应用**。

大致是这样：

```ts
import * as singleSpa from 'single-spa';

// 定义微应用的配置
const microAppConfig = {
  app: () => {
    // micro-app 相关资源
    loadScripts('./chunk-a.js');
    loadScripts('./chunk-b.js');
    return loadScripts('./entry.js');
  }, // 加载微应用的入口模块
  activeWhen: ['/micro-app'], // 触发微应用的URL路径
};

// 注册微应用
singleSpa.registerApplication('micro-app', microAppConfig, () => true);

// 启动Single-spa
singleSpa.start();
```

子应用提供相对应的生命周期钩子，方便 `single-spa` 进行管理：

```ts
// micro-app/src/index.js

import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

function bootstrap(props) {
  // 初始化微应用，如果有需要的话
}

function mount(props) {
  ReactDOM.render(<App />, document.getElementById('root'));
}

function unmount(props) {
  ReactDOM.unmountComponentAtNode(document.getElementById('root'));
}

// 导出生命周期钩子
export { bootstrap, mount, unmount };
```

如此便能够满足微前端的基本需求：`URL` 变化的时候加载/卸载子应用。

但它本身并不够完善，比如不能实现 `JS/CSS` 隔离，可能存在逻辑/样式冲突，此外子应用之间的通信处理也需要自己解决...

### 了解 `qiankun`

`qiankun` 是基于 `single-spa` 实现的，它解决了 `single-spa` 的一些痛点，是更完善的微前端解决方案。

#### 资源自动化加载

`qiankun` 会加载子应用入口的 `html`，将 `head` 部分转换为 `qiankun-head`，解析出 `scripts/styles`，单独去加载（实现在 ` import-html-entry` 这个模块里）。而无需开发者指定如何去加载资源，如图：

[![qiankun-p1](/images/share/qiankun/p1.png)](/images/share/qiankun/p1.png)

`single-spa` 的实现叫做 `Config Entry` 或者 `JS Entry`，也就是要自己指定怎么加载子应用，而 `qiankun` 这种叫做 `Html Entry`，会自动解析 `html` 实现加载。

#### `JS、CSS` 沙箱

理论上隔离 `JS` 只需要完成 `window` 全局变量隔离即可，函数内本就是在不同作用域下执行的。

可实行的方案：

- `快照、diff 比对` 加载之前记录，卸载后再恢复。缺陷就是不能同时存在多个子应用。

- `Proxy 代理`，通过代理对象访问。这也是比较常用的方案。

`CSS` 的隔离可实行方案：

- 使用 `shadow dom` 实现，这是浏览器支持的特性，`shadow root` 的 `dom` 不会影响其他 `dom`。

- `scoped css`，为元素添加属性 `id`，在 `css` 里通过前缀进行约束。

以上方案都可以通过配置来选择。

#### 子应用间通信

- 通过注册时传入的 `props` 和回调来实现状态管理，如下：

```ts
registerMicroApps([
  {
    name: 'sub-app',
    entry: 'http://localhost:3001', // 子应用的入口
    container: '#sub-app-container',
    activeRule: '/sub-app',
    props: {
      sharedState: {
        message: 'Hello from Main App',
      }, // 将共享状态传递给子应用
    },
  },
]);
```

子应用：

```ts
function render(props) {
  const { container } = props;
  ReactDOM.render(
    <App />,
    container ? container.querySelector('#root') : document.querySelector('#root'),
  );
}

export async function mount(props) {
  const { sharedState } = props;
  render(props);
}
```

- 通过 `initGlobalState` 来实现状态管理

主应用里定义子应用获取全局状态的方法：`getGlobalState` 和 监听状态变化的 `onGlobalStateChange`：

```ts
import { initGlobalState } from 'qiankun';

// 初始化全局状态
const actions = initGlobalState({
  count: 0,
});

// 定义一个改变状态的方法
actions.onGlobalStateChange((newState, prev) => {
  console.log('Global state changed:', newState);
});

actions.getGlobalState = (key) => {
  return key ? initialState[key] : initialState;
};

// 定义一个设置状态的方法
actions.setGlobalState({ count: 42 });

// 导出 actions 对象
export default actions;
```

子应用里通过 `props` 获取状态和修改状态：

```ts
export function mount(props) {
  const { setGlobalState, getGlobalState } = props;
  props.setGlobalState({ message: 'hello' });
}
```

### 实践

本次 `qiankun` 实践是在 `vue` 应用（`hash` 模式）里集成 `umi` 子应用（`hash` 模式）的部分页面。

在改造之前，两个应用都是单独部署运行的，是通过 `iframe` 的方式进行集成。所以这里采用了 `loadMicroApp` 进行子应用的加载：

#### 主应用改造

首先安装 `qiankun`：

```bash
yarn add qiankun
```

下面开始页面组件改造，通过 `loadMicroApp` 在组件内部动态加载和卸载子应用。例如之前集成了一个 `http://www.site.com:8000/appA/#/pageA` 这样一个页面。这个地址的组成：`http://www.site.com:8000/appA/` 是子应用 `appA` 的项目根地址，我们需要集成它的 `/pageA` 页面：

添加相关的路由，并保证一致性（最后总结有解释为什么需要添加子应用相关路由）：

```ts
// router/index.ts
import Vue from 'vue';
import VueRouter from 'vue-router';
import type { RouteConfig } from 'vue-router';

Vue.use(VueRouter);

const routes: RouteConfig[] = [
  {
    path: '/pageA', // 和子应用路由一致 （当然你可以添加自定义的主应用路由前缀，例如 /pc，即 /pc/pageA，然后添加相关配置即可，后面有提到）
    name: 'pageA',
    component: () => import('@/views/pageA/index.vue'),
  },
];

const router = new VueRouter({
  routes,
});

export default router;
```

然后在 `pageA/index.vue` 组件中手动加载相关页面资源：

```html
<!-- pageA/index.vue -->

<template>
  <div ref="containerRef"></div>
</template>

<script setup lang="ts">
  import type { MicroApp } from 'qiakun';
  import { loadMicroApp } from 'qiankun';

  const containerRef = ref<HTMLDivElement>();
  const app = ref<MicroApp>();

  // 子应用 appA 的根地址
  const entryUrl = computed(() => `http://www.site.com:8000/appA/`);

  onMounted(() => {
    if (containerRef.value) {
      app.value = loadMicroApp({
        name: 'appA',
        entry: entryUrl.value, // 子应用入口 index.html
        container: containerRef.value, // 挂载的 dom
        // eg.1
        props: {
          history: {
            type: 'hash', // 指定子应用使用的路由模式
          },
          // 访问子应用时浏览器的路由前缀，默认就是 /，如果像上面我提到的路由前缀添加了 /pc，那么这里就是 /pc
          // base: '/' 时： /pageA -> /pageA, base: '/pc' 时： /pc/pageA（主） -> /pageA（子）
          base: '/',
        },
        // eg.2
        // props 指定默认的子应用页面 更加的动态化，就像是使用 iframe 一样方便
        // props: {
        //   history: {
        //     // 子应用里不是这个模式，这里同样可以设置为 memory
        //     // memory 模式下，子应用路由跳转不改变浏览器的 URl，通常用于 mobile 端
        //     type: 'memory',
        //     initialEntries: [initEntry.value], // 设置默认的子应用路由信息
        //     initialIndex: 0, // 不传默认取 initialEntries 的第一个值，即默认访问的子应用路由
        //   },
        // },
      });
    }
  });

  onBeforeUnmount(() => {
    app.value?.unmount();
  });
</script>

<style scoped lang="less"></style>
```

这里并没有指定默认打开的子应用路由页面，所以使用的是子应用根路由【**这里不是需要 `/pageA` 吗？两种方案：一种是上面的配置指定路由模式为 `memory`，然后配置默认的路由页面（并无限制子应用需要是这个路由模式）；一种是为子应用根路由添加重定向**】。

至此，主应用的改造就完成了，是不是很简单？

#### 子应用改造

`umi` 项目内置了 `qiankun`，只需要开启配置即可：

```ts
// /config/config.ts

const pathPrefix = isDev ? '/' : '/appA/';

export default defineConfig({
  // ...
  hash: true,
  base: pathPrefix,
  publicPath: pathPrefix,
  headScripts: [{ src: './scripts/loading.js', async: true }], // 类似这样的配置，将资源路径改为相对路径
  // 开启 qiankun
  qiankun: {
    slave: {},
  },
});
```

确保子应用中存在刚刚主应用集成的路由，没有则添加：

```ts
// /config/routes.ts

export default [
  {
    path: '/',
    redirect: '/pageA', // 这里写了 redirect，所以主应用挂载时并没有指定具体路由
  },
  {
    path: '/pageA',
    component: '@/pages/pageA', // 组件正常编写即可，无需特殊处理
  },
];
```

### 总结

<div class="info">

> `Tips`:
>
> 1. 区分 `loadMicroApp` 和 `registerMicroApps` 两者在路由规则上的差异：前者本身并没有内置路由的管理和拦截机制，所以需要保证子应用路由和主应用路由一致，防止触发浏览器的页面刷新行为（`memory` 路由模式除外）。后者通过匹配到 `activeRule` 时，加载子应用模块，内部实现了路由拦截机制，所以无需保证子应用路由和主应用路由一致。
>
> 2. `loadMicroApp` 更适用于动态和个别子应用加载的场景（例如之前 `iframe` 集成某个页面时），而 `registerMicroApps` 适用于整体的子应用配置管理。
>
> 3. 只有 `memory` 路由模式下才能通过 `initialEntries` 设置初始化路由（这无关乎采用的是 `loadMicroApp` 还是 `registerMicroApps`）。

</div>
