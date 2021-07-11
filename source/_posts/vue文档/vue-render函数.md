---
title: vue-render 渲染函数
date: 2021-06-06 13:41:25
tags: [render函数, vue-jsx]
# type: "categories"
categories: [大前端, Vue]
---

`理解 render 函数之前，先了解 $createElement，除此之外，render 函数内需使用 jsx 语法完成目标编辑`;

### `$createElement` 使用

- 参数一、 `html`元素名称 or 组件名称(或组件对象) or 异步函数(resolve or return 元素或组件名)
- 参数二(可选)、当前 `$createElement` 的元素传入的 attribute 数据对象
- 参数三(可选)、子元素虚拟`DOM` 数组 or 字符串虚拟 `DOM`
- return `VNode`

`参数二：数据对象详解`

```typescript
{
  /* type: string | object | Array<string | object> */
  class: 'a', // v-bind:class
  /* type: string | object | object[] */
  style: 'color: #fff', // v-bind: style;
  /* type: { [k: string]: any } */
  attrs: { id: 'f' } // 元素属性
  props: { level: 1 } // 传入组件的 prop
  domProps: { innerHTML: '这是html元素额' } // dom 节点的属性
  on: { click: clickHandler } // 事件监听器 同 v-on(不支持 .修饰符)
  nativeOn: {} // 仅用于组件，监听原生事件， 同 vm.$emit 触发的事件
  directives: [{
    name: 'my-custom-directive',
    value: '111'
  }] // 自定义指令数组，内为每个指令属性对象 同 v-my-custom-directive="111"
  /* { name: props => VNode | Array<VNode> } name 为slot名称 props 为 scoped 对象 */
  scopedSlots: { default: (props) => createElement('span', props.text) }
  slot: 'name-of-slot', // 如果组件是其它组件的子组件，需为插槽指定名称
  key: 'myKey',
  ref: 'myRef',
  // 是否支持多个元素具有同一 ref 名，获取时为数组
  refInFor: true,
}
```

`补充：修饰符`
`对于 .passive、.capture 和 .once 这些事件修饰符，Vue 提供了相应的前缀可以用于 on：`

| 修饰符                           | 前缀 |
| -------------------------------- | ---- |
| .passive                         | &    |
| .capture                         | !    |
| .once                            | ~    |
| .`capture.once 或 .once.capture` | ~!   |

如：

```typescript
on: {
  '!click': this.doThisInCapturingMode,
  '~keyup': this.doThisOnce,
  '~!mouseover': this.doThisOnceInCapturingMode
}
```

`对于所有其它的修饰符，私有前缀都不是必须的，因为你可以在事件处理函数中使用事件方法： `

| 修饰符                                      | 处理函数中的等价操作                                                                                            |
| :------------------------------------------ | :-------------------------------------------------------------------------------------------------------------- |
| `.stop`                                     | `event.stopPropagation()`                                                                                       |
| `.prevent`                                  | `event.preventDefault()`                                                                                        |
| `.self`                                     | `if (event.target !== event.currentTarget) return`                                                              |
| 按键： `.enter`, `.13`                      | `if (event.keyCode !== 13) return` (对于别的按键修饰符来说，可将 `13` 改为[另一个按键码](http://keycode.info/)) |
| 修饰键： `.ctrl`, `.alt`, `.shift`, `.meta` | `if (!event.ctrlKey) return` (将 `ctrlKey` 分别修改为 `altKey`、`shiftKey` 或者 `metaKey`)                      |

### render 函数渲染 vue 组件

`vue 组件 使用 render 方法替换 template 模板`

例如 A.vue

- 模板渲染

```vue
<template>
  <div>这是A组件</div>
</template>
<script lang="ts">
import { Component, Vue, Watch, Prop, Emit } from "vue-property-decorator";

@Component({ name: "a", components: {} })
export default class A extends Vue {
  @Prop() public readonly level!: number;
  /* data */
  public name: string = "AGFSASGa";

  /* watch */
  @Watch("name")
  public onNameChange(newName: string) {}

  /* computed */
  public get transformName() {
    return store.state.global.name;
  }
  public set transformName(v: name) {
    store.commit("CHANGE_NAME", v);
  }

  public methodA() {}

  @Emit("cancel")
  public onCancel() {}
}
</script>
```

- 函数渲染：

`jsx在.vue文件渲染:` ([Babel 插件](https://github.com/vuejs/jsx) )

```vue
<script>
function renderFun(h) {
  return <div>这是A组件</div>;
}

export default {
  name: "a",
  components: {},
  props: {},
  data() {
    return { name: "aasgf" };
  },
  computed: {},
  watch: {
    name: {
      handler() {},
      immediate: true,
      deep: true,
    },
    // ...
  },
  methods: {
    methodA() {},
  },
  /* 替代 template */
  render(h) {
    return renderFun.call(this, h);
  },
};
</script>
```

`.js函数渲染vue`

```javascript
import ComponentA from "./ComponentA.vue";

function bindEventListener(that, dataObject) {
  /* 修改父组件 .sync 传入的name属性值 */
  dataObject.on.input = (name) => {
    this.$emit("update:input", name);
  };
}

export default {
  /* h 同 vm.$createElement 接收三个参数 */
  render(h) {
    /* dataObject 数据对象 */
    const dataObject = {
      props: ["name", "age", "config"],
      attrs: {
        // 自身属性
        sex: "男",
        /* props 继续向下传递 */
        name: this.name,
        age: this.age,
      },
      ref: this.config.ref,
    };
    const children = [];

    bindEventListener(this, dataObject);

    return h("ComponentA", dataObject, children);
  },
};
```
