---
title: v-for在特定场景下会失去响应式
date: 2022-12-26 15:45:40
updated: 2022-12-26 15:45:40
tags:
  - bug
  - vue
  - unresponsive
categories:
  - 踩坑记录
  - Vue
---

### 问题描述

前段时间，实现一个表单字段提交校验功能时碰到的一个有趣的问题...`v-for`的元素居然失去响应式了？`what...`~~开始怀疑人生 🙄🙄🙄~~

代码大致如下：

<!-- more -->

```html
<template>
  <div class="actionNodeValidate">
    <el-button icon="el-icon-edit" @click="addFieldCheckCondition">添加字段校验</el-button>
    <template v-for="(item, idx) in checkConditions">
      <el-row :key="idx" type="flex" justify="space-between" class="mt-5">
        <el-col :span="21"><el-input readonly :value="item.conditions" /></el-col>
        <el-col :span="1">
          <el-button icon="el-icon-edit" @click="editFieldConditions(idx)" />
        </el-col>
        <el-col :span="1">
          <el-button type="danger" icon="el-icon-delete" @click="delFieldConditions(idx)" />
        </el-col>
      </el-row>
    </template>
    <el-dialog title="校验通过条件配置" append-to-body :visible.sync="visible">
      <FormularPattern
        v-model="currentConf.conditions"
        @close="visible = false"
        @confirm="setFieldCondition"
      />
    </el-dialog>
  </div>
</template>

<script lang="ts">
  import { Component, PropSync, Vue } from 'vue-property-decorator';
  import FieldCheckConditions from './FieldCheckConditions';
  import FormularPattern from './FormularPattern/index.vue';

  @Component({ name: 'ActionNodeValidate', components: { FormularPattern } })
  export default class ActionNodeValidate extends Vue {
    @PropSync('conditions')
    public checkConditions!: FieldCheckConditions[];

    public visible = false;

    public conditionIdx = -1; // 标记当前的配置项索引

    public currentConf: FieldCheckConditions = new FieldCheckConditions();

    public addFieldCheckCondition() {
      this.currentConf = new FieldCheckConditions();
      this.conditionIdx = -1;
      this.visible = true;
    }

    public delFieldConditions(idx: number) {
      this.checkConditions.splice(idx, 1);
    }

    public editFieldConditions(idx: number) {
      this.conditionIdx = idx;
      this.currentConf = { ...this.checkConditions[idx] };
      this.visible = true;
    }

    public setFieldCondition() {
      if (this.conditionIdx === -1) {
        this.checkConditions.push({ ...this.currentConf });
      } else {
        this.checkConditions.splice(this.conditionIdx, 1, { ...this.currentConf });
      }
      this.visible = false;
    }
  }
</script>
```

> 对于数组类型值的修改使用的`push、splice`，~~当然也可以是`$set`~~。看起来似乎并没有什么毛病...

### 问题分析

通过`dev-tools`分析，发现`v-for`绑定的值是实时变更了，且组件也监听到变化了，但界面就是没渲染...

[![v-for-unresponsive-p1](/images/bugs/v-for-unresponsive/p1.png)](/images/bugs/v-for-unresponsive/p1.png)
[![v-for-unresponsive-p2](/images/bugs/v-for-unresponsive/p2.png)](/images/bugs/v-for-unresponsive/p2.png)

那就从源码分析...`vue`在更新`dom`元素时,会调用`updateChildren`方法：

[![v-for-unresponsive-p3](/images/bugs/v-for-unresponsive/p3.png)](/images/bugs/v-for-unresponsive/p3.png)
[![v-for-unresponsive-p4](/images/bugs/v-for-unresponsive/p4.png)](/images/bugs/v-for-unresponsive/p4.png)

> 进行 `diff` 比对，采用'就地复用'原则，减小开销，`eg. data数据 从 [1] -> [0,2,3,1,4,5] 过程中，进过比对，需要在1这个dom之前插入 0,2,3`，比对完成执行`addVnodes`函数。

```javascript
function addVnodes(parentElm, refElm, vnodes, startIdx, endIdx, insertedVnodeQueue) {
  for (; startIdx <= endIdx; ++startIdx) {
    createElm(vnodes[startIdx], insertedVnodeQueue, parentElm, refElm, false, vnodes, startIdx);
  }
}
```

我这调试的是`push`追加`currentConf`功能，`debug startIdx/endIdx（parentEle的childrenIndex）`均为 1.(不难发现，在上述例子中，`v-for 的父元素同 el-button<添加字段校验>（index = 0），均为 div.actionNodeValidate`，故`startIdx`为 1).

继续看`createElm`函数执行过程中发生了什么...

```javascript
function createElm(vnode, insertedVnodeQueue, parentElm, refElm, nested, ownerArray, index) {
  if (isDef(vnode.elm) && isDef(ownerArray)) {
    // This vnode was used in a previous render!
    // now it's used as a new node, overwriting its elm would cause
    // potential patch errors down the road when it's used as an insertion
    // reference node. Instead, we clone the node on-demand before creating
    // associated DOM element for it.
    vnode = ownerArray[index] = cloneVNode(vnode); // 拷贝当前需要添加的 vNode
  }

  vnode.isRootInsert = !nested; // for transition enter check
  if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
    // 添加的vnode子组件创建，这里是 null，返回undefined
    return;
  }

  var data = vnode.data;
  var children = vnode.children; // []
  var tag = vnode.tag;
  if (isDef(tag)) {
    // vue-component-48-ElRow
    if (process.env.NODE_ENV !== 'production') {
      if (data && data.pre) {
        creatingElmInVPre++;
      }
      if (isUnknownElement$$1(vnode, creatingElmInVPre)) {
        warn(
          'Unknown custom element: <' +
            tag +
            '> - did you ' +
            'register the component correctly? For recursive components, ' +
            'make sure to provide the "name" option.',
          vnode.context,
        );
      }
    }

    vnode.elm = vnode.ns
      ? nodeOps.createElementNS(vnode.ns, tag)
      : nodeOps.createElement(tag, vnode); // document.createElement
    setScope(vnode);

    /* istanbul ignore if */
    {
      createChildren(vnode, children, insertedVnodeQueue); // children is [].
      if (isDef(data)) {
        invokeCreateHooks(vnode, insertedVnodeQueue); // 创建相关 hooks
      }
      insert(parentElm, vnode.elm, refElm); // 执行 insert
    }

    if (process.env.NODE_ENV !== 'production' && data && data.pre) {
      creatingElmInVPre--;
    }
  } else if (isTrue(vnode.isComment)) {
    vnode.elm = nodeOps.createComment(vnode.text);
    insert(parentElm, vnode.elm, refElm);
  } else {
    vnode.elm = nodeOps.createTextNode(vnode.text);
    insert(parentElm, vnode.elm, refElm);
  }
}
```

可以看到，在`createElm`函数中，执行了`拷贝vNode -> createComponent -> document.createElement -> createChildren -> hooks挂载 -> 最终执行insert函数`...

```javascript
function insert(parent, elm, ref$$1) {
  if (isDef(parent)) {
    // div.actionNodeValidate
    if (isDef(ref$$1)) {
      // el-dialog__wrapper.node-validate-dialog ---> v-for 的相邻元素(插入位置index+1) ---> (因为v-for没有wrapper，所以这里是 .node-validate-dialog)
      // parent(div.actionNodeValidate) !== body
      if (nodeOps.parentNode(ref$$1) === parent) {
        // 比对插入位置节点与其相邻元素的父节点是否一致，一致则直接在相邻元素前插入新节点，不一致时没有做任何处理
        nodeOps.insertBefore(parent, elm, ref$$1);
      }
      // 不一致... 我们的代码执行到这  G了...
    } else {
      // 没有相邻vNode，直接在 parentEle 追加 children
      nodeOps.appendChild(parent, elm);
    }
  }
}
```

> `insert`函数比对了'插入位置'与其'相邻`vNode`'(**插入位置`index+1`的`vNode`节点**)的父节点是否一致，一致则在其'相邻节点'前插入新的节点(`insertBefore`)。~~值得注意的是，如果相邻`vNode`的父节点与当前`parentEle`不相同时，没有做任何处理...😮😮😮~~
> 如果相邻元素不存在，那么就直接追加(`appendChild`).

最终发现`nodeOps.parentNode(ref$$1) === parent`条件为`false`，所以页面并没有渲染.`why？`为什么父节点会不一致呢？

`el-dialog`添加了`append-to-body`属性后，其父节点变成了`body`，所以代码执行到`insert，parentNode(div.actionNodeValidate) !== body`，比对后发现`parent`发生变更，不执行更新。导致页面上呈现的效果是`dom`没渲染，而响应式数据已经更新。~~这也是为什么调试时`nextTick`内执行`forceUpdate`也无法重新渲染的根本原因~~

找到了原因，那么解决方案自然也就有了（**避免插入的位置相邻`vNode`的`parent`不会变更**就`ok`啦 😁😁😁）

### `hack`解决方案

#### 给绑定了`v-for`的元素添加一层父级的`wrapper`节点

`eg.`

```html
<template>
  <div class="actionNodeValidate">
    <div>...</div>
    <div>
      <template v-for="(item, idx) in checkConditions"> ... </template>
    </div>
    <el-dialog append-to-body :visible.sync="visible"> ... </el-dialog>
  </div>
</template>
```

#### 调整绑定了`append-to-body`属性的`dom`位置，避免与`v-for`元素直接相邻

`eg.`

```html
<template>
  <div class="actionNodeValidate">
    <div>...</div>
    <template v-for="(item, idx) in checkConditions"> ... </template>
    <div>...</div>
    <el-dialog append-to-body :visible.sync="visible"> ... </el-dialog>
  </div>
</template>
```

#### 移除`append-to-body`属性

`eg.`

```html
<template>
  <div class="actionNodeValidate">
    <div>...</div>
    <template v-for="(item, idx) in checkConditions"> ... </template>
    <el-dialog :visible.sync="visible"> ... </el-dialog>
  </div>
</template>
```
