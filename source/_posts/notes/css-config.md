---
title: 界面化配置自定义的组件css样式
date: 2022-12-26 14:24:08
updated: 2022-12-26 14:24:08
tags:
  - cssTree
  - 自定义样式
categories:
  - 土豆の学习笔记
  - 大前端
---

### 了解[css-tree](https://github.com/csstree/csstree)

`CSSTree is a tool set for CSS: fast detailed parser (CSS → AST), walker (AST traversal), generator (AST → CSS) and lexer (validation and matching) based on specs and browser implementations. The main goal is to be efficient and W3C spec compliant, with focus on CSS analyzing and source-to-source transforming tasks.`

### 为什么使用它？

最近项目上有个需求：提供一个界面化的`css`代码编辑器，管理员可以输入`css`代码配置修改表单/组件的一些默认样式，从而针对不同业务需求实现不同的界面呈现效果。

为了避免样式的全局污染，需要对配置的样式添加默认前置导航，通过属性选择器约束样式的生效范围。如果通过`string`的一些方法粗暴的对样式进行替换`or`正则匹配，有很大的局限性，同时也不符合规范。

思考再三，找到了`css -> ast -> css.`方案，并通过`css-tree`的`API`实现。

~~根据产品业务的区别，如果只是想实现样式自定义，不需要通过代码配置达到更强的交互，那么可以内置`class`，编写默认的一些`style`，用户通过在界面选择主题来实现该效果。参考百度[amis](https://github.com/baidu/amis)~~

<!-- more -->

### 源码实现

#### 给组件唯一标识

> 可以给组件的`wrapper-dom`添加具有唯一标志性的属性，用于约束自定义样式生效的区间范围.

```html
<!-- 给组件 wrapper 绑定具有唯一标识的 id -->
<template>
  <div class="form-item_container" :id="field.__vModel__">
    <RenderWrapper v-bind="$attrs" v-on="$listeners" />
  </div>
</template>
```

#### `helper`实现

> 编写用于操作`css-code`的工具函数，给自定义的`css-code`添加内置的属性选择器前缀。

```typescript
import { clone, generate, parse, walk } from 'css-tree';

export const styleHelper = {
  addClassPrefix(prefix: string, context: string) {
    const orig = parse(context); // css -> ast
    const ast = clone(orig); // clone ast node.

    walk(ast, (node, item) => {
      // 仅对选择器的首个 selector 节点添加前缀  eg.  .form .item {} -> .prefix .form .item {}
      if (!item?.prev) {
        if (node.type === 'TypeSelector') {
          // eg. input div li...
          node.name = `.${prefix} ${node.name}`;
        }

        if (node.type === 'ClassSelector') {
          // eg. .el-input .customCls...
          node.name = `${prefix} .${node.name}`;
        }
        // other process...
      }
    });

    return generate(ast); // ast -> css
  },

  loadCss(id: string, cssContent: string) {
    const styleEle = document.getElementById(`style_${id}`);

    styleEle?.remove(); // 热更新，避免重复性追加 style 节点.
    // or if (styleEle) return;

    const style = document.createElement('style');
    const head = document.getElementsByTagName('head')[0];

    style.id = `style_${id}`;

    const styleCode = styleHelper.addClassPrefix(`form-item_container[id=${id}]`, cssContent);

    style.appendChild(document.createTextNode(styleCode));
    head.appendChild(style);
  },
};
```

#### `config-editor`实现

> 用于配置`css`的编辑器组件.~~偷了个懒，如果你需要更友好的实现，可以添加`css`语法提示~~

```html
<template>
  <el-dialog
    title="自定义样式"
    class="codemirror_dialog"
    :destroy-on-close="true"
    :close-on-click-modal="false"
    :visible.sync="customStyleVisible"
  >
    <VueCodemirror v-model="innerValue" :options="options" />
    <template v-slot:footer>
      <el-button @click="customStyleVisible = false">取 消</el-button>
      <el-button type="primary" @click="onConfirmHandle">确 定</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts">
  import { Component, ModelSync, PropSync, Vue, Watch } from 'vue-property-decorator';

  @Component({
    name: 'StyleConfig',
    components: {
      VueCodemirror: async () => {
        const { codemirror } = await import(/* webpackChunkName: "codeMirror" */ 'vue-codemirror');
        return codemirror;
      },
    },
  })
  export default class StyleConfig extends Vue {
    @PropSync('visible')
    public customStyleVisible!: boolean;

    @ModelSync('value', 'input')
    public customStyle!: string;

    public innerValue = '';

    public options = {
      tabSize: 2,
      mode: 'text/css',
      theme: 'default',
      styleActiveLine: true,
      matchBrackets: true,
      placeholder:
        '请输入组件自定义css样式... eg.\n\n.el-input { color: #f00; } \n\n input { border: none; }',
    };

    @Watch('value', { immediate: true })
    public setContentValue(v: string) {
      this.innerValue = v;
    }

    public async created() {
      await this.initCodemirror();
    }

    public async initCodemirror() {
      await import(/* webpackChunkName: "codeMirror" */ 'codemirror/mode/css/css.js');
      await import(/* webpackChunkName: "codeMirror" */ 'codemirror/addon/display/placeholder.js');
      await import(
        /* webpackChunkName: "codeMirror" */ 'codemirror/addon/selection/active-line.js'
      );
      await import(/* webpackChunkName: "codeMirror" */ 'codemirror/addon/edit/matchbrackets.js');
      await import(/* webpackChunkName: "codeMirror" */ 'codemirror/lib/codemirror.css');
    }

    public onConfirmHandle() {
      this.customStyleVisible = false;
      this.customStyle = this.innerValue;
    }
  }
</script>
```

#### `parser`实现

> 在表单`parser`组件`created`钩子`loadCss`，将自定义的样式代码加载到`html`.

```typescript
import { styleHelper } from './util/index';

public created() {
  const { code, css } = field.__config__;

  styleHelper.loadCss(code, css);
}
```
