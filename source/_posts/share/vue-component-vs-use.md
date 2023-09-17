---
title: Vue.component 和 Vue.use 两者之间的区别
date: 2023-09-17 15:37:16
updated: 2023-09-17 15:37:16
tags:
  - vue
  - vueComponent
  - vueUse
categories:
  - 技术分享
  - Vue
---

突然想起，项目里大多时候，`UI` 组件库在进行全局或者局部注册的时候，会使用到 `Vue.use`，有时候也会使用到 `Vue.component`，那么两者之间有什么区别呢？我们又应该如何根据场景选择注册方式呢？

<!-- more -->

### `Vue.component`

> 用于注册全局组件。全局组件是可以在整个应用程序中使用的组件，不需要在每个单独的组件中再次注册。

通常，你会在根 `Vue` 实例创建之前使用 `Vue.component` 来注册全局组件。`eg:`

```ts
import Vue from 'vue';
import MyComponent from './MyComponent.vue';

Vue.component('my-component', MyComponent);
```

完成注册后，你可以在任何 `Vue` 组件的模板中使用这个组件。

```html
<my-component></my-component>
```

### `Vue.use`

> 用于注册 `Vue.js` 插件。插件通常是第三方库或功能模块，它们可以在 `Vue` 应用中全局使用。

当你使用 `Vue.use` 注册一个插件时，它会调用插件的 `install` 方法，传递 `Vue` 构造函数作为参数，以便插件可以进行必要的初始化和扩展。`eg:`

```ts
import Vue from 'vue';
import MyPlugin from './my-plugin';

Vue.use(MyPlugin, { size: 'middle' });
```

`MyPlugin` 就需要我们提供 `install` 方法，或者当 `MyPlugin` 本身就是一个函数的时候，则直接运行：

```ts
// ./my-plugin.js
import MyComponent from './MyComponent.vue';

// 当使用 Vue.use() 注册插件时，install 方法会被调用，在该方法内部，即可实现全局注册的相关逻辑
const install = (Vue, options) => {
  console.log('Vue', Vue); // Vue 构造函数
  console.log('MyPlugin installed!');

  // 期望用户执行 Vue.use(MyPlugin) 时，1.注册组件：
  Vue.component('my-component', MyComponent);

  // 2.期望用户在注册的时候为组件添加一些全局配置，例如主题颜色尺寸等... 【通过在 Vue 的原型上添加全局方法或属性实现】
  Vue.prototype.$MyPlugin = {
    ...options,
    size: options.size,
  };

  // 3.添加全局方法或属性
  Vue.globalFunction = () => {};

  // 4.挂载全局指令
  Vue.directive('my-directive', {
    bind (el, binding, vnode, oldVnode) {
      // 逻辑...
    }
    ...
  });

  // 5.注入全局资源 --- 虽然不推荐这么做
  Vue.mixin({
    created: function () {
      // 逻辑...
    }
    ...
  });
}

export default {
  install, // 导出 install 方法，支持插件式注册
  MyComponent, // 导出改组件，支持组件单个注册
}
```

通过在 `Vue` 的原型上添加全局方法或属性实现来扩展组件功能，如上所示，在功能组件内部实现时即可通过 `this.$MyPlugin` 来读取配置信息：

```ts
// ./my-component.vue

computed: {
  size () {
    return this.$MyPlugin.size;
  }
}
```

#### 为什么要提供 `install`?

细看 `vue` 源码，可以发现，`Vue.use` 调用后执行了插件提供的 `install` 方法，或者当插件本身是函数的时候，无需提供 `install`。此外在执行前，内置了 `this` 参数，也就是 `Vue` 构造器作为插件初始化的第一个参数：

```ts
import { toArray } from '../util/index';
export function initUse(Vue: GlobalAPI) {
  Vue.use = function (plugin: Function | Object) {
    const installedPlugins = this._installedPlugins || (this._installedPlugins = []);
    if (installedPlugins.indexOf(plugin) > -1) {
      return this;
    }
    // additional parameters
    const args = toArray(arguments, 1);
    args.unshift(this);
    if (typeof plugin.install === 'function') {
      plugin.install.apply(plugin, args);
    } else if (typeof plugin === 'function') {
      plugin.apply(null, args);
    }
    installedPlugins.push(plugin);
    return this;
  };
}
```

### 差异总结

- `Vue.component` 用于注册全局组件，以便在整个应用程序中使用。

- `Vue.use` 用于注册 `Vue.js` 插件，以扩展 `Vue` 的功能和添加全局方法或属性。

- **使用时机不同**：`Vue.component` 通常在组件定义之前使用，而 `Vue.use` 通常在根 `Vue` 实例创建之前使用。

### 使用场景

`Vue.component` 用于全局组件注册。而 `Vue.use` 不仅可以在插件内部通过执行 `Vue.component` 实现组件的批量注册逻辑，额外添加一些自定义处理，提升开发体验，还多用于第三方插件库集成，例如 `VueRouter...`💀💀💀

> 之所以几乎所有的 `UI` 库都支持 `Vue.use` 方式，一是因为可以进行批量全局注册。二是可以添加自定义的组件配置等等...这些都是 `Vue.component` 不具备的。
