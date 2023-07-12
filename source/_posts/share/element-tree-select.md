---
title: 基于 elementUI 实现的 treeSelect 组件
date: 2023-07-12 18:52:49
updated: 2023-07-12 18:52:49
tags:
  - TreeSelect
  - UI
categories:
  - 技术分享
  - 功能实现
---

基于 `element` 的 `tree` 和 `select` 组件，实现的一个 `treeSelect` 组件，完整 `props` 和用法参考 `element`.

<!-- more -->

### 模板和样式文件

```html
<template>
  <div class="tree-select-wrapper">
    <div class="mask" v-show="isShowSelect" @click="selectClickHandle"></div>
    <el-popover
      placement="bottom-start"
      :width="width"
      trigger="manual"
      v-model="isShowSelect"
      @hide="popoverHide">
      <el-tree
        class="common-tree"
        :style="style"
        ref="tree"
        :lazy="lazy"
        :data="data"
        :props="defaultProps"
        :show-checkbox="multiple"
        :node-key="nodeKey"
        :check-strictly="checkStrictly"
        :expand-on-click-node="false"
        :check-on-click-node="multiple"
        :highlight-current="true"
        :load="loadChildNodes"
        v-on="$listeners"
        @node-click="handleNodeClick"
        @check-change="handleCheckChange"></el-tree>
      <el-select
        :style="selectStyle"
        slot="reference"
        ref="select"
        :size="size"
        v-model="selectedData"
        :multiple="multiple"
        :disabled="disabled"
        :clearable="clearable"
        :collapse-tags="collapseTags"
        @click.native="selectClickHandle"
        @remove-tag="removeSelectedNodes"
        @clear="removeSelectedNode"
        @change="changeSelectedNodes"
        class="tree-select">
        <el-option
          v-for="item in options"
          :key="item.value"
          :label="item.label"
          :value="item.value"></el-option>
      </el-select>
    </el-popover>
  </div>
</template>

<script lang="ts" src="./index.ts"></script>

<style lang="less" scoped>
  .mask {
    width: 100%;
    height: 100%;
    position: fixed;
    top: 0;
    left: 0;
    opacity: 0;
  }
  .common-tree {
    overflow: auto;
  }
  .tree-select-wrapper {
    /deep/.tree-select .el-select__tags .el-tag .el-tag__close {
      display: none;
    }
    /deep/.tree-select .el-select__tags .el-tag .el-icon-close {
      display: none;
    }
  }
</style>
```

### 源码逻辑

`由于时间原因，代码中的类型不是很准确和完善，自行更正，准确类型参考 elTree 和 elSelect.`~~就是懒~~😥😥😥

```ts
import { Component, Prop, Vue, Watch } from 'vue-property-decorator';

type Option = { value: string; label: string };
type Node = { key: string; label: string; childNodes?: Node[] };

@Component({})
export default class TreeSelect extends Vue {
  @Prop({ default: () => [] }) public readonly data!: any[];
  @Prop({ default: () => ({}) }) public readonly defaultProps!: any;

  @Prop(Boolean) public readonly multiple?: boolean;
  @Prop(Boolean) public readonly clearable?: boolean;
  @Prop(Boolean) public readonly disabled?: boolean;
  @Prop(Boolean) public readonly lazy?: boolean;
  @Prop(Boolean) public readonly collapseTags?: boolean; // 配置多选时是否将选中值按文字的形式展示
  @Prop(Boolean) public readonly checkStrictly?: boolean; // 显示复选框情况下，是否严格遵循父子不互相关联

  @Prop({ default: 250 }) public readonly width!: number;
  @Prop({ default: 300 }) public readonly height!: number;

  @Prop({ default: 'small' }) public readonly size!: string;
  @Prop({ default: 'id' }) public readonly nodeKey!: string;
  @Prop({ default: () => [] }) public readonly checkedKeys!: string[];

  @Prop(Function) public readonly loadChildNodes!: (node: any, resolve: () => void) => void;

  public isShowSelect = false;

  public options: Option[] = [];
  public selectedData: string | string[] = [];
  public checkedIds = [];
  public checkedData = [];

  public get style() {
    return 'width:' + this.width + 'px;' + 'height:' + this.height + 'px;';
  }

  public get selectStyle() {
    return 'width:' + (this.width + 24) + 'px;';
  }

  @Watch('isShowSelect')
  public onSelect() {
    (this.$refs.select as any).blur();
  }

  @Watch('checkedKeys', { deep: true })
  public onCheckedKeysChange(val: string[]) {
    if (!val) return;
    this.initCheckedData();
  }

  public mounted() {
    this.initCheckedData();
  }

  public selectClickHandle() {
    if (this.disabled) return;
    this.isShowSelect = !this.isShowSelect;
  }

  public setSelectOption(node: Node) {
    const tmpMap: any = {};
    tmpMap.value = node.key;
    tmpMap.label = node.label;
    this.options = [];
    this.options.push(tmpMap);
    this.selectedData = node.key;
  }

  public checkSelectedNode(checkedKeys: string[]) {
    const item = checkedKeys[0];
    (this.$refs.tree as any).setCurrentKey(item);
    const node = (this.$refs.tree as any).getNode(item);
    this.setSelectOption(node);
  }

  public checkSelectedNodes(checkedKeys: string[]) {
    (this.$refs.tree as any).setCheckedKeys(checkedKeys);
  }

  public clearSelectedNode() {
    this.selectedData = '';
    (this.$refs.tree as any).setCurrentKey(null);
  }

  public clearSelectedNodes() {
    // 所有被选中的节点的 key 所组成的数组数据
    const checkedKeys = (this.$refs.tree as any).getCheckedKeys();
    for (let i = 0; i < checkedKeys.length; i++) {
      (this.$refs.tree as any).setChecked(checkedKeys[i], false);
    }
  }

  public initCheckedData() {
    if (this.multiple) {
      if (this.checkedKeys.length > 0) {
        this.checkSelectedNodes(this.checkedKeys);
      } else {
        this.clearSelectedNodes();
      }
    } else {
      if (this.checkedKeys.length > 0) {
        this.checkSelectedNode(this.checkedKeys);
      } else {
        this.clearSelectedNode();
      }
    }
  }

  public popoverHide() {
    if (this.multiple) {
      // 所有被选中的节点的 key 所组成的数组数据
      this.checkedIds = (this.$refs.tree as any).getCheckedKeys();
      // 所有被选中的节点所组成的数组数据
      this.checkedData = (this.$refs.tree as any).getCheckedNodes();
    } else {
      this.checkedIds = (this.$refs.tree as any).getCurrentKey();
      this.checkedData = (this.$refs.tree as any).getCurrentNode();
    }
    this.$emit('popoverHide', this.checkedIds, this.checkedData);
  }

  public handleNodeClick(d: any, node: Node) {
    if (!this.multiple) {
      this.setSelectOption(node);
      this.isShowSelect = !this.isShowSelect;
      this.$emit('change', this.selectedData);
    }
  }

  public handleCheckChange() {
    // 所有被选中的节点的 key 所组成的数组数据
    const checkedKeys = (this.$refs.tree as any).getCheckedKeys();
    this.options = checkedKeys.map((item: string) => {
      // 所有被选中的节点对应的node
      const node = (this.$refs.tree as any).getNode(item);
      const tmpMap: any = {};
      tmpMap.value = node.key;
      tmpMap.label = node.label;
      return tmpMap;
    });
    this.selectedData = this.options.map((item) => {
      return item.value;
    });
    this.$emit('change', this.selectedData);
  }

  public removeSelectedNodes(val: string) {
    (this.$refs.tree as any).setChecked(val, false);
    const node = (this.$refs.tree as any).getNode(val);
    if (!this.checkStrictly && node.childNodes.length > 0) {
      this.treeToList(node).map((item) => {
        if (!item?.childNodes?.length) {
          (this.$refs.tree as any).setChecked(item, false);
        }
      });
      this.handleCheckChange();
    }
    this.$emit('change', this.selectedData);
  }

  public treeToList(tree: Node) {
    let queen: Node[] = [];
    const out = [];
    queen = queen.concat(tree);
    while (queen.length) {
      const first = queen.shift();
      if (first?.childNodes) {
        queen = queen.concat(first.childNodes);
      }
      out.push(first);
    }
    return out;
  }

  public removeSelectedNode() {
    this.clearSelectedNode();
    this.$emit('change', this.selectedData);
  }

  public changeSelectedNodes(selectedData: any) {
    if (this.multiple && selectedData.length <= 0) {
      this.clearSelectedNodes();
    }
    this.$emit('change', this.selectedData);
  }
}
```
