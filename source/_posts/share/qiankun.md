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

两个应用都是单独部署运行的，在改造之前，是通过 `iframe` 的方式集成。下面开始：

> `因为是 vue 应用里集成 umi 子应用，所以这里 vue 应用是主应用，umi 应用就需要改成微应用。案例采用的是手动加载的方式`：

- 首先，在主应用里安装 `qiankun`：

```bash
yarn add qiankun
```

- 因为是手动加载的方式集成的子应用，所以需要外部路由和子应用路由相匹配：

> 例如之前 `iframe` 是通过 `http://www.xxx.com:8080/appA/#/pc/pageA` 访问子应用 `A`，那么现在在主应用里应该也存在 `/pc/pageA` 的路由。

以上述为例主应用 `/pc/pageA` ----> 指向的页面 `A.vue`。在 `A.vue` 需要手动加载子应用：

```html
<template>
  <div ref="containerRef"></div>
</template>

<script setup lang="ts">
  import type { MicroApp } from 'qiankun';
  import { loadMicroApp } from 'qiankun';

  const containerRef = ref<HTMLDivElement>();
  const app = ref<MicroApp>();

  // 子应用 appA 的根地址
  const entryUrl = computed(
    () =>
      `${
        process.env.NODE_ENV === 'development'
          ? 'http://localhost:8000/'
          : `${process.env.BASE_URL}appA/`
      }`,
  );

  onMounted(() => {
    if (containerRef.value) {
      app.value = loadMicroApp({
        name: 'appA',
        entry: entryUrl.value,
        container: containerRef.value,
        props: {
          history: {
            type: 'hash', // 子应用使用哈希路由模式，所以这里添加了 type: hash。详细如下面子应用配置
          },
          base: '/pc', // 访问子应用时浏览器的路由前缀，如果不添加，默认就是 /
        },
      });
    }
  });

  onBeforeUnmount(() => {
    app.value?.unmount();
  });
</script>

<style scoped lang="less"></style>
```

- 然后改造子应用（是 `umi` 项目）：

> `umi` 项目内置了 `qiankun`，只需要开启配置即可：

```ts
// /config/config.ts

const pathPrefix = isDev ? '/' : '/appA/';

export default defineConfig({
  // ...
  hash: true,
  base: pathPrefix,
  publicPath: pathPrefix,
  headScripts: [{ src: './scripts/loading.js', async: true }], // from headScripts: [{ src: '/scripts/loading.js', async: true }],

  qiankun: {
    slave: {},
  },
});
```

确保子应用中存在刚刚主应用需要的路由，没有则添加：

```ts
// /config/routes.ts

export default [
  {
    path: '/',
    redirect: '/pageA', // 前面配置时添加了 /pc 的前缀，这里又写了 redirect，所以主应用挂载时并没有指定具体路由
  },
  {
    path: '/pageA',
    component: '@/pages/pageA', // 组件正常编写即可，无需特殊处理
  },
  {
    path: '/mobile',
    component: '@/pages/Mobile/index.tsx',
    routes: [
      {
        path: '/mobile',
        redirect: 'userInfo',
      },
      {
        path: 'userInfo',
        name: '个人信息',
        component: '@/pages/Mobile/Info/index.tsx',
      },
    ],
  },
];
```

---

除了上面这种方式指向默认的子应用跟路由，我们也可以定义访问的子应用路由，例如（`/mobile/userInfo`），修改上面主应用的加载配置：

```html
<template>
  <div ref="containerRef"></div>
</template>

<script setup lang="ts">
  import type { MicroApp } from 'qiankun';
  import { loadMicroApp } from 'qiankun';

  const containerRef = ref<HTMLDivElement>();
  const app = ref<MicroApp>();
  const isDev = process.env.NODE_ENV === 'development';

  // 子应用 appA 的根地址
  const entryUrl = computed(
    () => `${isDev ? 'http://localhost:8000/' : `${process.env.BASE_URL}appA/`}`,
  );

  const initEntry = computed(
    () => `${isDev ? 'http://localhost:8000/' : entryUrl.value}mobile/userInfo`,
  );

  onMounted(() => {
    if (containerRef.value) {
      app.value = loadMicroApp({
        name: 'appA',
        entry: entryUrl.value,
        container: containerRef.value,
        props: {
          history: {
            type: 'memory', // memory 模式下，子应用路由跳转不改变浏览器的 URl，通常用于 mobile 端
            initialEntries: [initEntry.value],
            initialIndex: 1,
          },
        },
      });
    }
  });

  onBeforeUnmount(() => {
    app.value?.unmount();
  });
</script>

<style scoped lang="less"></style>
```
