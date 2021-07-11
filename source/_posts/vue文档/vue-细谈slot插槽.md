---
title: vue-细谈slot插槽
date: 2021-06-11 09:36:12
tags: [slot, scope, slot-scope]
# type: "categories"
categories: [大前端, Vue]
---

### 普通插槽

`在定义组件时，可以通过 <slot 标签定义插槽的位置及内容 `

```vue
<!-- 基础组件：base-layout -->
<template>
  <div class="base-layout">
    <!-- slot 标签包裹的值 将作为 插槽的默认值显示 -->
    <slot>这里是插槽的默认显示内容</slot>
  </div>
</template>

<!-- 基础组件在父级组件中默认插槽的使用 -->
<template>
  <div class="parent-layout">
    <!-- (推荐)方式一 将插槽内容直接写入标签之间  -->
    <base-layout>作为插槽的值 传入子组件的 slot 内, 可以是组件</base-layout>

    <!-- 方式二：通过给 template 标签 添加 v-slot 指令设置 (作为默认插槽 我感觉这样有点画蛇添足，可以这样写 不推荐)  -->
    <base-layout>
      <template v-slot:default>
        同样，这里的内容也将插入子组件的默认插槽；(注意：v-slot指令只能在
        template 上使用)
      </template>
    </base-layout>

    <!-- (已废弃)方式三 给包裹的元素添加 slot 属性 -->
    <base-layout>
      <p slot="default">这是一段插入子组件 默认插槽的内容</p>
    </base-layout>
  </div>
</template>
```

### 具名插槽

`明白普通插槽过后，我们再来看具名插槽，具名：顾名思义就是在设置 slot 时，绑定 name 属性，区分普通插槽，避免插槽过多时混淆`

```vue
<!-- 首先来看子组件的定义 -->
<!-- base-layout -->
<template>
  <div class="base-layout">
    <!-- 给 slot 设置 name 属性，让 slot “具名” -->
    <slot name="header">定义一个头部插槽，让插入的值显示在组件的头部</slot>
    <div>组件自身的一些其他DOM</div>
    <slot>作为默认插槽</slot>
    <slot name="footer">定义一个尾部插槽，让插入的值显示在组件的尾部</slot>
  </div>
</template>

<!-- 组件被使用 -->
<template>
  <div class="parent-layout">
    <!-- (推荐)方式一 通过 template 标签包裹，并在 template 上使用 v-slot 设置插入的位置 -->
    <base-layout>
      <template v-slot:header>修改后的头部内容</template>
      作为默认的内容插入 default-slot
      <template v-slot:footer>修改后的尾部内容</template>
    </base-layout>

    <!-- (已废弃)方式二 给包裹的元素添加 slot 属性 -->
    <base-layout>
      <p slot="header">修改后的头部内容</p>
      <p slot="footer">修改后的头部内容</p>
    </base-layout>
  </div>
</template>
```

### 作用域插槽

`部分场景下，我们在父组件中，访问并使用子组件，并通过 slot 来自定义组件的 UI 时，希望能够同时访问 子组件内部维护的数据。`

```vue
<!-- 首先来看子组件的定义 -->
<!-- base-layout -->
<template>
  <div class="base-layout">
    <!-- 为了让父组件在使用时，能够使用子组件内部的变量属性，我们需要给 slot 绑定动态 attribute -->
    <slot :user="user">默认文字</slot>
    <slot name="footer"></slot>
  </div>
</template>

<script>
export default {
  data() {
    return { user: { name: "张三" } };
  },
};
</script>

<!-- 父组件中使用 base-layout -->
<template>
  <div class="parent-layout">
    <!-- 方式一：子组件只有匿名插槽时 v-slot 指令才能直接作用域组件本身，scope 支持解构 -->
    <base-layout v-slot="user">
      {{ user.name }} 被插入子组件的 slot 内
    </base-layout>
    <base-layout v-slot="{ name }"> {{ name }} 解构写法 </base-layout>

    <!-- 方式二：除了匿名插槽外，还含有具名插槽，那么 v-slot 就只能作用域 template 上，避免作用域不明确 -->
    <base-layout>
      <!-- 这里写成 v-slot:default="user" 也可以，default 一般省略不写 -->
      <template v-slot="user">
        {{ user.name }} 被插入子组件的默认 slot 内
      </template>
      <template v-slot:footer="user">
        {{ user.name }} 被插入子组件的 name 为 footer 的 slot 内
      </template>
    </base-layout>
  </div>
</template>
```

- 只要出现多个插槽，请始终为所有的插槽使用完整的基于 `<template>` 的语法
- 注意 v-slot 使用场景: 子组件只有 默认插槽时 可以直接作用域组件本身，否则只能作用域 template；
- 注意 v-slot: 、v-slot= 、v-slot:name= 的含义和区别
