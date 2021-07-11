---
title: vue-特殊属性_is_key_ref_介绍
date: 2021-06-08 14:41:12
tags: [is属性, key属性, ref属性]
# type: "categories"
categories: [大前端, Vue]
---

### is

`常用于构建动态组件 或 基于 DOM 内模板的解析`

> 场景 1：作为动态组件 <component is="aaa" /> 的属性，动态切换组件显示
> 场景 2：作为普通 html-dom 的属性，动态指定当前标签渲染的组件 如 li，tr 绑定 is 属性后，将不受特殊 dom 树的影响 如 ul 子级组件 为 li。

### key

`作用于 vue 的虚拟 dom算法，辨识新旧的 nodes 对比 VNodes。`

> 场景 1：最常见的即为 在 v-for 指令 DOM 中使用，以便维护内部组件及子树的状态，提升渲染效率，更加高效的更新虚拟 DOM，避免临时数据混淆。
>
> 场景 2：对于部分组件，我们不希望状态被缓存。如 <component 动态组件，动态从 A -> A 的过程中，（即两个相同的组件切换）可能会存在 model 状态遗留，绑定 key 即可解决。
>
> 场景 3：部分特殊组件，在使用过程中，DOM 需要动态销毁重建。（如我在项目中遇到的 handsontable 基于 hot-column 渲染自定义组件时， 无法通过 api 添加或删除列数据，以及无法通过修改 data 触发 hot-column 组件更新，采用动态更新绑定的 key 值解决这一问题。---当然方法不唯一）。

### ref

`作为元素或组件的引用，存储于父组件的 $refs 对象上。如果在普通 DOM 上使用，则获取的就是 DOM ，如果在子组件上使用，则指向其组件实例。`

> 场景 1：普通 DOM 绑定 ref
>
> 场景 2：组件绑定 ref
>
> 场景 3：v-for 内绑定 ref (this.$refs.xxx 获取到的是 vm 数组)

```vue
<template>
  <p ref="pp">hello</p>
</template>
<script>
export default {
  mounted() {
    console.log(this.$refs.pp.textContent); // hello
  },
};
</script>
```

- 作用于组件，获取组件实例的方法或属性

`通过绑定 ref 到组件本身，使当前组件能访问其子组件内部定义的变量(属性)或者方法。`

```vue
<!-- 如果需要获取 DOM 相关数据或节点 需搭配 await this.$nextTick() 使用 -->
<template>
  <A><B ref="bb"></B></A>
</template>
<script>
export default {
  methods: {
    getBm() {
      console.log(this.$refs.bb.val);
      this.$refs.bb.changeVal("sss");
    },
  },
};
</script>
```
