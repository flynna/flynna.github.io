---
title: 教你如何写一个 vue 的 UI 组件库
date: 2023-09-17 19:45:31
updated: 2023-09-17 19:45:31
tags:
  - vue
  - UI
categories:
  - 技术分享
  - Vue
---

在此之前，你需要先了解如何发布一个 `npm`，参考 [如何发布一个 npm-package?](/tools/npm-publish)，以及 `Vue.use 和 Vue.component` 实现，参考 [Vue.component 和 Vue.use 两者之间的区别](/share/vue-component-vs-use)。

在下一篇文章中，我将学习并介绍三方库说明文档搭建：~~先空着。。。🙄🙄🙄~~

<!-- more -->

### 初始化项目

#### 脚手架构建

因为是从 `0` 到 `1` 构建，所以就选择了 `vue3` 了，参考 [https://vuejs.org/guide/quick-start.html](https://vuejs.org/guide/quick-start.html)

<div class="info">

> npm create vue@latest
> ✔ Project name: … xxxxx
> ✔ Add TypeScript? … No / Yes
> ✔ Add JSX Support? … No / Yes
> ✔ Add Vue Router for Single Page Application development? … No / Yes
> ✔ Add Pinia for state management? … No / Yes
> ✔ Add Vitest for Unit testing? … No / Yes
> ✔ Add an End-to-End Testing Solution? … No / Cypress / Playwright
> ✔ Add ESLint for code quality? … No / Yes
> ✔ Add Prettier for code formatting? … No / Yes
> Scaffolding project in .xxxxx...
> Done.

</div>

#### 目录结构调整

默认生成的项目目录结构如下：

[![vue-components-ui-install-p1](/images/share/vue-components-ui-install/p1.png)](/images/share/vue-components-ui-install/p1.png)

你很容易发现，大多数的 `UI` 项目组件库，都是将 `Demo` 调试资源放置在了 `example/` 目录下，而组件资源则放置到了 `packages/`。

为了保证结构化统一，我们需要重命名 `src` 为 `example`（`demo` 展示，可以直接是一个完整的 `vue` 项目 ☺☺☺）：

此外，我们需要再新建一个 `src 和 packages` 文件夹，作为组件包的实际入口文件以及包相关的资源。修改后的核心目录说明：

<div class="warning">

> `example`：用于测试组件的相关功能，用作 `UI` 组件的 `Demo` 展示（原脚手架创建的 `src` 目录）。
> `packages`：用于存放封装的自定义组件，仅含文件夹，每个子文件夹代表一个组件资源。
> `src`：组件库打包的入口文件存放处，同时也可以存放 `UI` 库相关的其他资源，例如指令、工具函数、类型定义等。

</div>

新目录结构如下：

[![vue-components-ui-install-p2](/images/share/vue-components-ui-install/p2.png)](/images/share/vue-components-ui-install/p2.png)

#### 配置调整

由于修改了原脚手架的 `src` 目录名称，又新建了资源目录，所以需要我们对项目配置做相应的调整：

- **修改 `index.html` 的 `script.src` 属性，由 `/src/main.ts` 修改为 `/example/main.ts`**

```diff
- <script type="module" src="/src/main.ts"></script>
+ <script type="module" src="/example/main.ts"></script>
```

- **修改 `vite.config.ts` 中的 `resolve.alias @` 符号的别名配置**

```diff
- '@': fileURLToPath(new URL('./src', import.meta.url))
+ '@': fileURLToPath(new URL('./example', import.meta.url))
```

- **修改 `package.json` 中的 `format` 指令 `prettier` 的文件夹路径**

```diff
- "format": "prettier --write src/"
+ "format": "prettier --write src/ example/ packages/"
```

- **修改 `tsconfig.app.json` 下相关路径，并添加 `include` 配置**

```json
{
  // ...
  "include": [
    "env.d.ts",
    "src/**/*",
    "packages/**/*",
    "packages/**/*.vue",
    "example/**/*",
    "example/**/*.vue"
  ],
  "exclude": ["example/**/__tests__/*"],
  "compilerOptions": {
    "composite": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./example/*"]
    }
  }
}
```

<div class="danger">

> 注意：由于缓存的问题，修改了配置后，可能仍存在异常提示，这时需要手动清除掉 `node_modules/.vite` 目录，关掉服务，并重启编辑器，再重启项目。

</div>

### 组件库设计

本文旨在记录组件库的实现过程和思路，所以只写了一个 `demo` 组件。~~后续如有需要，会在此基础上添加完善其他的组件实现 😶😶😶~~

#### `Demo` 按钮组件实现

在 `packages/` 文件夹下创建 `button/` 文件夹，用于存放按钮组件相关资源。

`由于 vue3 版本改动原因，v-listeners 被合并到 $attrs 统一处理，所以就没有绑定 v-on="$listeners" 了`

```html
// packages/button/src/index.vue

<script setup lang="ts"></script>

<script>
  export default {
    name: 'vui-button',
  };
</script>

<template>
  <div>
    <!-- vue3 the $listeners is deprecated -->
    <button type="button" class="vui-button" v-bind="$attrs">
      <slot></slot>
    </button>
  </div>
</template>

<style lang="less" scoped>
  .vui-button {
    color: #317a2e;
  }
</style>
```

#### 插件注册支持

很多时候，使用第三方 `UI` 组件库，我们可以通过 `vue.use` 来注册全局组件，而 `vue.use` 需要我们提供 `install` 方法：

```ts
// packages/button/index.ts

import VuiButton from './src/index.vue';

VuiButton.install = function (Vue: any) {
  Vue.component(VuiButton.name || VuiButton.__name, VuiButton);
};

export default VuiButton;
```

#### `less` 支持

由于脚手架并没有内置 `less`，所以需要单独安装：

```bash
yarn add less less-loader -D
```

并在 `vite.config.ts` 中添加如下配置：

```ts
export default defineConfig({
  // ...
  css: {
    preprocessorOptions: {
      less: {
        javascriptEnabled: true,
      },
    },
  },
});
```

**`ok，如上所示，一个单一的组件功能和结构就实现好了，它已经具备了被集成的基础条件`。**

---

#### 全局批量注册支持

上面的处理，让组件可以单独导入并通过 `Vue.use or Vue.component` 注册，下面将主要针对整个组件库的安装注册做介绍。

在根目录下新创建的 `src` 下面新建入口文件 `index.ts`，用于完成组件库的安装注册。如下：

```ts
// /src/index.ts

import VuiButton from '../packages/button';
// ...other component

type InstallFunction = {
  (Vue: any, options?: { size?: 'small' | 'middle' | 'large'; theme?: 'dark' | 'light' }): void;
  installed?: boolean;
};

interface Window extends globalThis.Window {
  [k: string]: any;
  Vue?: any;
}

// 所有自定义的组件
const vuiComponents = [VuiButton];

// 支持 use.use 全局注册所有组件
const install: InstallFunction = function (Vue, options = {}) {
  if (install.installed) return;

  // 因为组件内部实现了 install 方法，所以可以直接 Vue.use
  vuiComponents.forEach((component) => Vue.use(component));

  // 如果没有实现，则如下：
  // vuiComponents.forEach(component => Vue.component(component.name || component.__name, component))

  // -------------------------------------------------------

  // vue3 使用 app.config.globalProperties 替代 prototype
  Vue.config.globalProperties.$vui = {
    ...options,
    size: options.size || 'middle',
    theme: options.theme || 'light',
  };
  // 组件内部 使用以下方式读取：
  // import { getCurrentInstance } from 'vue';

  // const app = getCurrentInstance();
  // const vuiConfig = app?.appContext.config.globalProperties.$vui

  // --------------------------------------------------------

  // vue2 版本的全局配置
  // Vue.prototype.$vui = {
  //   ...options,
  //   size: options.size || 'middle',
  //   theme: options.theme || 'light',
  // };
};

// 直接给浏览器或 AMD loader 使用
if (typeof window !== 'undefined' && (window as Window).Vue) {
  install((window as Window).Vue);
}

export default {
  install, // 用于ES modules，import Vue 后直接使用 Vue.use()
  VuiButton, // 解构赋值导出单个组件
  // ...other component
};
```

<div class="success">

> 由于 `vue3 和 vue2` 的版本差异性，`vue3` 中插件 `install` 方法提第一个参数 `app`，并不能访问到 `prototype`
>
> 意味着之前 `vue2` 通过 `prototype` 为全局添加配置的方式不适用了，可以通过 **`app.config.globalProperties 替代 prototype`**，具体用法可以参考上面实现。
>
> 可以在组件库实现的时候就考虑兼容，或者在后续组件库版本中升级迭代完成兼容性

</div>

### 组件本地测试

在 `example/main.ts` 中，引入并注册组件库：

```ts
// 测试自定义的组件
import Vui from '../src/index';

const app = createApp(App);

app.use(Vui);
// ...

app.mount('#app');
```

在 `App.vue` 中使用组件库：

```html
<script setup lang="ts">
  import { RouterLink, RouterView } from 'vue-router';
  import HelloWorld from './components/HelloWorld.vue';
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="@/assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="You did it!" />

      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
      </nav>
    </div>
  </header>
  <!-- 主要是下面这一句，由于是全局安装的组件，所以这里无需导入 -->
  <vui-button>测试一下自定义的button</vui-button>
  <RouterView />
</template>
```

看看效果：

[![vue-components-ui-install-p3](/images/share/vue-components-ui-install/p3.png)](/images/share/vue-components-ui-install/p3.png)
