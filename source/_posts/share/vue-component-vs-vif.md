---
title: Vue Component 动态组件和 v-if 控制组件两者区别？
date: 2023-09-15 14:59:36
updated: 2023-09-15 14:59:36
tags:
  - vue
  - component
  - v-if
categories:
  - 技术分享
  - Vue
---

其实两者从定义上讲区别还是挺明显的：

> `v-if`：用于条件性地显示或隐藏一个元素，适用于简单的条件渲染。

> `component`：通过 `:is` 属性实现组件的动态切换，适用于更复杂的条件渲染场景，可以根据条件加载不同的组件

那为什么要单独拧出来说呢？是因为**两者在特殊情况下是具有一致性**的..😟😟😟

<!-- more -->

### 相同点

仅从切换单个组件渲染的角度来讲：`component` **动态组件可以理解为就是 `v-if` 控制显示的语法糖**。

_试想一个场景，当我们需要从很多个组件中挑选一个进行渲染的时候，使用 `v-if` 就会显得特别的臃肿，这个时候切换到 `component`渲染，可以增加代码的可读性和可维护性_

例如：

```html
<template>
  <div>
    <MyComponentA v-if="showComponentName === 'MyComponentA'">A</MyComponentA>
    <MyComponentB v-if="showComponentName === 'MyComponentB'">B</MyComponentB>
    <MyComponentC v-if="showComponentName === 'MyComponentC'">C</MyComponentC>
    <MyComponentD v-if="showComponentName === 'MyComponentD'">D</MyComponentD>
    <MyComponentE v-if="showComponentName === 'MyComponentE'">E</MyComponentE>
    <MyComponentF v-if="showComponentName === 'MyComponentF'">F</MyComponentF>
  </div>
</template>

<script>
  import MyComponentA from './MyComponentA.vue';
  import MyComponentB from './MyComponentB.vue';
  import MyComponentC from './MyComponentC.vue';
  import MyComponentD from './MyComponentD.vue';
  import MyComponentE from './MyComponentE.vue';
  import MyComponentF from './MyComponentF.vue';

  export default {
    data() {
      return {
        showComponentName: 'MyComponentA',
      };
    },
    components: {
      MyComponentA,
      MyComponentB,
      MyComponentC,
      MyComponentD,
      MyComponentE,
      MyComponentF,
    },
  };
</script>
```

是不是看着都挺累....

这个时候我们就可以使用 `component` 组件来渲染，这样代码看起来就简洁多了：

```html
<template>
  <div>
    <component :is="showComponentName"></component>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        showComponentName: 'MyComponentA',
      };
    },
    components: {
      MyComponentA: () => import('./MyComponentA.vue'),
      MyComponentB: () => import('./MyComponentB.vue'),
      MyComponentC: () => import('./MyComponentC.vue'),
      MyComponentD: () => import('./MyComponentD.vue'),
      MyComponentE: () => import('./MyComponentE.vue'),
      MyComponentF: () => import('./MyComponentF.vue'),
    },
  };
</script>
```

为什么说这种模式下，`component` 可以理解为 `v-if` 的语法糖？

> **当通过 `is` 切换 `component` 组件显示时，对比 `v-if` 页面渲染和钩子函数的执行情况，两者是一致的。**

除此之外：

### 不同点

- `v-if` 旨在控制单个元素的渲染，**并未限制元素是否为独立组件**

- `v-if` 适合简单的渲染场景，使用起来也更加的灵活，而 **`component` 局限于多选一的情况**

- 一个是指令，一个是组件，两者实现和实际用法不一样，只有在特定情况下两者是一致的。
