---
title: vue-自动化注册全局组件
date: 2021-06-10 17:25:02
tags: [自动化注册]
# type: "categories"
categories: [大前端, Vue]
---

### 获取组件

- `require.context`
- `params`： directory {String} -读取文件的路径、 `useSubdirectories` {Boolean} -是否遍历文件的子目录、`regExp {RegExp}` -匹配文件的正则
- return: function - 此函数挂载了三个属性
  - resolve {Function} -接受一个参数 request,request 为 test 文件夹下面匹配文件的相对路径,返回这个匹配文件相对于整个工程的相对路径
  - keys {Function} -返回匹配成功模块的名字组成的数组
  - id {String} -执行环境的 id,返回的是一个字符串,主要用在 `module.hot.accept`

> 语法: `require.context(directory, useSubdirectories = false, regExp = /^.//)`;

`示例 获取该路径下所有的 .vue文件`

```typescript
const componentsContext = require.context("./customized", true, /\.vue$/);
```

### 枚举 `componentsContext`，完成组件全局注册

`main.ts` 中 `import '当前自动化文件'`

```typescript
import Vue from "vue";
import { kebabCase } from "lodash";

componentsContext.keys().forEach((component) => {
  const componentConfig = componentsContext(component);

  // 兼容 import export 和 require module.export 两种规范
  const ctrl = componentConfig.default || componentConfig;

  if (ctrl.prototype instanceof Vue) {
    // 读取 vue 组件name属性 || 文件名
    const compName = kebabCase(ctrl.options.name || ctrl.name);

    // 完成注册
    Vue.component(compName, ctrl);
  }
});
```



------

`其他附带配置(案例中使用到，实际使运用中不需要)`

------



### 注册并设置组件配置属性(略)

`场景： 自动化注册自定义组件，同时支持对组件配置， 在引入组件时，读取配置设置组件的相关属性，更加灵活(用于表单自动化构建)`

- 基于配置的组件注册 需要`Meun配置`组件提供 `getConfig` 方法
- 单个组件需配套 提供用于渲染的 组件 A，以及用于配置组件相关属性的 `Meun-A` 组件

`interface`

```typescript
import Vue from "vue";
export interface PluginConfig {
  config: any;
}
export default abstract class PluginConfiguratorBase extends Vue {
  protected abstract getConfig(): PluginConfig;
}
```

`./customized/a/menu.vue` a 组件的配置组件

```vue
// ...
<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import PluginConfiguratorBase, { PluginConfig } from "./PluginBase";

@Component({ name: "a-menu" })
export default class AMenu extends PluginConfiguratorBase {
  protected getConfig(): PluginConfig {
    /* 可以任意追加自定义的配置项 */
    return {
      __vModel__: "",
      __config__: { title: "多选框", description: "", defaultValue: [] },
    };
  }
}
</script>
```

```typescript
// 存储自定义组件的基础配置
const customizedComponents = [];
// ...
if (ctrl.prototype instanceof Vue) {
  // ...
  if (ctrl?.options?.methods?.getConfig) {
    const config = ctrl.options.methods.getConfig() as PluginConfig;
    customizedComponents.push(config);
  }
  // ...
}
export customizedComponents;
```
