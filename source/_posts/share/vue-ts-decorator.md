---
title: Vue-Ts-Decorator 类组件装饰器使用
date: 2023-08-22 14:55:07
updated: 2023-08-22 14:55:07
tags:
  - vue
  - 类装饰器
  - decorator
categories:
  - 技术分享
  - Vue
---

### 引言

目前 `vue` 组件的主流开发风格是基于 `ts setup` 语法的 `composition api`，`composition api` 相对于 `options api` 来说，更加灵活，代码更加简洁。

由于历史原因，仍有很多项目技术栈使用的是 `vue2`。本篇文章也旨在介绍 `vue-ts-decorator` 类组件装饰器的使用和优化。

<!-- more -->

### 构建

`vue-ts` 项目构建就不做细讲了，根据 `vue-cli` 提供的选项选择并创建项目，只需要注意在选择 `Use class-style component syntax? (Y/n)` 选择 `Y` 即可。

又或者如果项目是之前并没有使用 `decorator` 的老项目，可以比对和上面脚手架创建的项目的 `package` 依赖，手动安装没有的包即可。

值得注意的库：

> `vue-class-component` (类的装饰器)、 `vue-property-decorator` (类似 `Component Prop Emit...` 等装饰器从这里导出，是基于 `vue-class-component` 扩展的)、此外还有 `vuex-module-decorators` (可选择性安装使用的 `vuex` 库)

目前仍在使用`decorator` 进行开发的大多是老项目，新项目可以尝试直接拥抱 `vue3-ts` 的 `setup`。

### 类组件

在此之前，`.vue` 组件的常见写法是：

```html
<template>
  <div @click="clickHandle">{{ msgSync }}</div>
</template>

<script>
  export default {
    name: 'HelloWorld',
    data() {
      return {
        flag: false,
      };
    },
    computed: {
      msgSync() {
        return this.flag ? 'Hello Boy' : 'Hello World';
      },
    },
    ss: {
      clickHandle() {
        this.flag = !this.flag;
      },
    },
  };
</script>

<style scoped></style>
```

现在我们把它转为类组件实现：

```html
<template>
  <div @click="clickHandle">{{ msgSync }}</div>
</template>

<script lang="ts">
  import { Component, Vue } from 'vue-property-decorator';

  @Component({
    name: 'HelloWorld',
    // 可以传入其他的一些选项 ... 例如 computed/components...
  })
  export default class HelloWorld extends Vue {
    public flag = false;

    public get msgSync() {
      return this.flag ? 'Hello Boy' : 'Hello World';
    }

    // 如果需要设置 set
    // public set msgSync(value: string) {}

    public clickHandle() {
      this.flag = !this.flag;
    }
  }
</script>

<style scoped></style>
```

不难发现两者的区别，`data` 对象的属性变成了类的属性，`ss` 变成了类的方法，`computed` 变成了 `get(set) function`。

当然除了 `class` 导致的一些变化外，在 `@Component` 中，可以传入以前选项式的其他 `options` 配置，例如组件名称、子组件注册，又或者需要使用 `vuex` 提供的一些 `map`方法时：

```ts
import { mapState } from 'vuex';
import { Component, Vue } from 'vue-property-decorator';
import Children1 from '@/components/Children1.vue';

@Component({
  name: 'HelloWorld',
  components: {
    Children1,
    Children2: () => import('@/components/Children2.vue'),
  },
  computed: {
    ...mapState({
      msg: (state) => state.msg,
    }),
  },
})
export default class HelloWorld extends Vue {}
```

### 装饰器

常规用法和全部装饰器详见 [https://github.com/kaorun343/vue-property-decorator](https://github.com/kaorun343/vue-property-decorator)

~~很无语子...之前我在使用的时候，貌似并没有发现 `Ref 和 VModel` 的装饰器 🤐🤐🤐~~

官方文档其实已经把装饰器用法说的很详细了，我补充一些注意的点：

#### 类型为 `Boolean` 的 `Prop`

形如：

```ts
export default class HelloWorld extends Vue {
  @Prop() public readonly checked!: boolean;
}
```

在我们在使用装饰器的时候，由于 `ts` 的类型约束，即如上：`checked!: boolean`，很容易使我们忽略掉 `@Prop()` 自身需要的 `Type`，如果没有指定类型，那么在形如下面的方式传入时会导致不可预知的异常：

```html
<CustomComponent checked></CustomComponent>
<!-- 等价于 -->
<CustomComponent checked=""></CustomComponent>
```

原因就是没有指定 `Prop` 的类型。能不能就像上面这种方式传入值呢？当然可以，但需要指定 `Prop` 类型，即：

```ts
export default class HelloWorld extends Vue {
  @Prop(Boolean) public readonly checked!: boolean;

  // 或者
  // @Prop({ type: Boolean }) public readonly checked!: boolean;
}
```

只有这样，才能拿到正确的 `Prop` 值，如下：

```html
<CustomComponent checked></CustomComponent>
<!-- 等价于 -->
<CustomComponent :checked="true"></CustomComponent>
```

> 如果你在使用的时候，都是严格按照 `:checked="true"` 这种方式传递，那么不写 Type 也不会影响

#### `PropSync 和 ModelSync` 的优势

在推出装饰器之前，如果需要对 `Prop` 的值进行修改，我们需要在每一次变更的时候 `emit` 一个事件，然后在父组件中修改：

```ts
export default {
  props: {
    checked: {
      type: Boolean,
      default: false,
    },
    methods: {
      changeChecked(val: boolean) {
        this.$emit('changeChecked', val);
      },
    },
  },
};
```

针对这种需求，官方提供了 `.sync` 修饰符，即在传入 `prop` 的时候添加 `.sync` 修饰符，如下：

```html
<CustomComponent :checked.sync="checked"></CustomComponent>
```

在组件中如果需要修改 `checked` 的值，只需要 `emit('update:' + propName, value)` 即可，如下：

```ts
{
  methods: {
    changeChecked(val: boolean) {
      this.$emit('update:checked', val);
    },
  },
}
```

这样确实已经解决了父组件在收到事件过后不用再赋值的问题，但是 `PropSync 和 ModelSync` 的出现，还能继续省略优化我们的代码：

```html
<input v-model="myValue"></input>

<script lang="ts">
  import { Vue, Component, ModelSync } from 'vue-property-decorator';

  export default class HelloWorld extends Vue {
    @ModelSync('value', 'input') public myValue!: string;
  }
</script>
```

其父组件：

```html
<HelloWorld v-model="value" />
```

你应该也发现了，`ModelSync PropSync` 的出现，让我们在子组件中可以直接修改 `Model 和 Prop` 而不需要再手动执行 `emit`：

```ts
export default class HelloWorld extends Vue {
  @PropSync('checked') public myChecked!: boolean;

  public changeChecked(val: boolean) {
    // 会同步修改父组件传入的 checked 等价 $emit('update:checked', val);
    this.myChecked = val;
  }
}
```

#### `VModel` 装饰器

和 `ModelSync` 装饰器作用差不大多，我理解下来其实就是**省略了子组件添加计算属性作为 `VModel` 的过程**，让我们得益于直接操作 `VModel` 传入的 `Prop`.

#### `Ref` 装饰器

这个我之前使用的时候确实没有看到，应该是官方后面补充的，但这并不影响它的实用性。下面我就沿用上面的一个例子来说明：

`HelloWorld` 组件实现：

```ts
export default class HelloWorld extends Vue {
  @PropSync('checked') public myChecked!: boolean;

  public message = 'hello world';

  public changeMessage() {
    this.message = 'hello vue';
  }

  public changeChecked(val: boolean) {
    // 会同步修改父组件传入的 checked 等价 $emit('update:checked', val);
    this.myChecked = val;
  }
}
```

在我们需要操作子组件，调用子组件的 `API`， 或者获取子组件的属性和方法的时候，我们可以为组件绑定 `ref` 属性，如下：

```html
<template>
  <HelloWorld ref="hello"></HelloWorld>
</template>

<script lang="ts">
  import { Vue, Component } from 'vue-property-decorator';
  import HelloWorld from 'HelloWorld.vue';

  @Component({
    name: 'Parent',
    components: {
      HelloWorld,
    },
  })
  export default class Parent extends Vue {
    public setHelloWorldMessage(v: string) {
      this.$refs.hello.changeMessage(v);
    }
  }
</script>

<style></style>
```

但通常，我们需要为 `hello` 指定类型，否则在 `ts` 环境下会报错：

```ts
public setHelloWorldMessage(v: string) {
  (this.$refs.hello as HelloWorld).changeMessage(v);
}
```

此时，我们可以使用 `Ref` 装饰器来指定类型，避免过多的 `as` 语句：

```ts
import { Vue, Component } from 'vue-property-decorator';
import HelloWorld from 'HelloWorld.vue';

@Component({
  name: 'Parent',
  components: {
    HelloWorld,
  },
})
export default class Parent extends Vue {
  // 当然你也可以设置 ref 别名
  @Ref() readonly hello!: HelloWorld;

  public setHelloWorldMessage(v: string) {
    this.hello.changeMessage(v);
  }
}
```

#### `Mixins` 不推荐

`Mixins` 由于使用较少和局限性，~~更不推荐使用`mixins`的功能~~：

`建议更多地使用组件组合、插槽和Vuex等更可控的方式来处理代码复用和共享状态.`

> 缺陷：命名冲突、隐式依赖、代码复杂性增加、耦合度增加、维护困难、不利于组件重用、混合的顺序问题....

#### 其他

还有`Prop/Watch/Emit`装饰器都很适用， 除此之外剩下的几个装饰器也都有各自的使用场景，由于没有什么比较凸出的优化点，就不一一阐述了，详细可看官方文档.
