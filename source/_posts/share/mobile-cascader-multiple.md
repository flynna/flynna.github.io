---
title: Mobile 端的级联选择单多选组件实现
date: 2023-07-12 16:27:49
updated: 2023-07-12 16:27:49
tags:
  - Cascade
  - Mobile-UI
categories:
  - 技术分享
  - 组件封装
---

### `Why?`

级联选择组件在`UI`库中很常见，几乎所有的`UI`库都支持。但是你会发现几乎所有用于 `PC` 端的 `Cascade` 组件都支持对选项多选和搜索，而 `Mobile` 端的 `Cascade` 组件，更多的是以 `picker` 的形式仅支持单选。

局限性大，同时在部分业务场景下，例如表单组件，在 `PC` 和 `Mobile` 端的不同呈现兼容，需要我们不得不支持多选这一要求...

如果使用传统的 `picker` 形式，那么在交互上它不可能实现多选操作，此外如果级联层级过深甚至无法在手机端友好展示，所以我另辟蹊径，参考了目前部分`Mobile-UI` 库对“地址选择器”的交互效果，即：通过 `tab` 渲染不同层级实现选项选择。🤨🤨🤨

<!-- more -->

### 需要准备什么？

由于我的案例中使用了基础的`UI`库组件，所以需要你的项目引入才能使用，例如 `tabs、cell、checkbox`...

当然这些基础组件也可以由你自己去封装，为了节约案例功能的实现成本，我这使用的 `element 和 vant`~~使用 vant 是因为项目里移动端用的 vant 库，同时使用 element 是因为 vant 的 checkbox 没有提供半选状态，这个也可以自己实现~~😅😅😅...

### 实现思路？

- 明确需要接受的 `props` 和向外传播的 `event`

- 明确 `v-model` 变量类型和变更条件，例如点击 `cell` ? 还是点击 `cell-right-slot`?

- 明确当前交互状态，即：用户的每一次点击操作，我们都应该记录下当前操作的选项路径 `currentSelectPath`

- 根据上一步的路径，我们可以很轻松的拿到需要渲染的可选层级 `selectableOptions` 和渲染的 `tabs` ，如果 `currentSelectPath` 为空，则展示根层级即可。至此，我们可以很轻松的实现选项切换这一交互效果和呈现

- 下一步是值绑定，我们需要根据 `currentSelectPath` 和当前点击的 `option` 推断它的完整路径，并 `emit:input` 实现数据双向绑定

- 上面是传统 `Mobile-Cascade` 的交互实现，下面开始优化升级

- 添加计算属性，计算每个选项是否被全选、半选，并展示对应的 `icon` 和被选中样式

- 添加单多选支持，单选时点击叶子节点选中，点击父节点则展开子节点。多选时，仅点击 `icon` 选中所有叶子节点

- 递归计算已选项的`label`，并用于界面呈现

- 支持 `filterable` 对级联选项进行搜索选中，交互参考自 `element`

### 源代码

`代码基于 Vue2 ts decorator 语法实现，可根据自己需要，对应调整技术栈.`

#### 模板

`input 组件用于对已选项进行呈现，你可以切换成 tags。`

`另外该例子仅对核心功能做实现，如有需要你可以添加一些额外的交互和 props`

```html
<template>
  <div class="mobile-cascader">
    <template v-if="readonly">
      <span v-if="selectedLabel">{{ selectedLabel }}</span>
      <span>暂无内容</span>
    </template>
    <template v-else>
      <el-input
        class="cascade-input"
        readonly
        :value="selectedLabel"
        :placeholder="placeholder"
        @focus="showPicker = true">
        <template v-slot:suffix>
          <i
            v-if="selectedLength"
            class="el-icon-circle-close"
            style="position: relative; top: 4px; right: 4px"
            @click="innerValue = []" />
        </template>
      </el-input>
      <van-popup
        v-model="showPicker"
        :lazy-render="false"
        position="bottom"
        get-container="body"
        round
        closeable
        class="popover"
        :style="{ height: '486px' }">
        <template v-if="innerOptions.length">
          <div class="title p-16">
            {{ title }}
            <i class="selected">(已选择 {{ selectedLength }} 项)</i>
          </div>
          <el-input placeholder="搜索选项" prefix-icon="el-icon-search" v-model="optionFilterKey" />
          <SelectorList
            v-show="optionFilterKey"
            v-model="filterValue"
            :options="filterOptions"
            :multiple="multiple" />
          <van-tabs
            v-show="!optionFilterKey"
            v-model="activeTab"
            class="p-6"
            color="#02A7F0"
            title-active-color="#02A7F0"
            title-inactive-color="#000">
            <!-- FIX: value is not unique -->
            <van-tab
              v-for="(tab, tabIdx) in selectableOptions"
              :title="getTabTitle(tab, tabIdx)"
              :key="`${getTabPrePath(tabIdx)}_${tabIdx}`">
              <van-cell-group class="option-wrapper" :border="false">
                <van-cell
                  v-for="(option, optionIdx) in tab"
                  :key="`${tabIdx}-${optionIdx}-${option.value}`"
                  :title="option.label"
                  :title-class="{ selectedCell: checkSelected(option, tabIdx) }"
                  clickable
                  @click="optionClickHandle(option, tabIdx)">
                  <template v-slot:right-icon>
                    <el-checkbox
                      v-if="showableCheckbox(option, tabIdx)"
                      :value="getOptionSelectedStatus(option, tabIdx).selectAll"
                      :indeterminate="getOptionSelectedStatus(option, tabIdx).indeterminate"
                      class="check-box"
                      @click.native.stop
                      @change="optionSelectChange(option, tabIdx)" />
                    <van-icon
                      name="arrow"
                      v-if="option.children && option.children.length"
                      class="arrow-icon"
                      size="16" />
                  </template>
                </van-cell>
              </van-cell-group>
            </van-tab>
          </van-tabs>
        </template>
        <van-empty
          v-else
          description="暂无数据"
          :image="require('@/assets/images/empty-image-default.png')" />
      </van-popup>
    </template>
  </div>
</template>
```

#### 核心逻辑

```ts
import type { CascaderOption } from 'element-ui/types/cascader';
import { cloneDeep } from 'lodash';
import { Component, ModelSync, Prop, Vue } from 'vue-property-decorator';]
// 这个是对选项搜索后，展示拉平后的选项列表
import SelectorList from './selectorList.vue';

const ROOT = '_ROOT_';
const SPLIT_CHAR = '|';

@Component({
  name: 'MobileCascade',
  components: {
    SelectorList,
  },
})
export default class MobileCascade extends Vue {
  @ModelSync('value', 'input', { type: Array }) public innerValue!: string[] | string[][];

  @Prop() public readonly innerOptions!: CascaderOption[];

  @Prop(String) public readonly placeholder?: string;
  @Prop(String) public readonly title?: string;

  @Prop(String) public readonly readonly!: boolean;
  @Prop({ default: false, type: Boolean }) public readonly multiple!: boolean;

  public activeTab = 0;
  public showPicker = false;
  public optionFilterKey = '';

  public currentSelectPath: string[] = [];

  /* ---------- filter v-model start --------- */
  public get filterValue() {
    return this.innerValue2Value(this.innerValue);
  }

  public set filterValue(v) {
    this.innerValue = this.value2InnerValue(v);
  }
  /* ------ end ---------- */

  public get selectedLabel() {
    const innerVal = this.multiple ? this.innerValue : [this.innerValue];
    return innerVal.map((v) => this.getOptionPathLabel(v as string[]).join('/')).join('、');
  }

  public get selectedLength() {
    return this.multiple ? this.innerValue.length : this.innerValue?.length ? 1 : 0;
  }

  public get selectableOptions() {
    return [ROOT, ...this.currentSelectPath]
      .map((p, idx, arr) => this.getOptionChildrenByPath(arr.slice(0, idx + 1)))
      .filter((o) => !!o.length);
  }

  public get filterOptions() {
    const flatterOption = this.getFlatterOptions(this.innerOptions);
    return flatterOption.filter((o) => o.label.includes(this.optionFilterKey));
  }

  public mounted() {
    this.currentSelectPath = (
      this.multiple ? this.innerValue[0] ?? [] : this.innerValue
    ) as string[];
  }

  public value2InnerValue(val?: string[]) {
    const tv = val?.map((v) => v.split(SPLIT_CHAR)) ?? [];
    return this.multiple ? tv : tv[0] ?? [];
  }

  public innerValue2Value(v: string[] | string[][]) {
    const tv = this.multiple ? v : v.length ? [v] : [];
    return tv.map((o) => (o as string[]).join(SPLIT_CHAR));
  }

  public getOptionChildrenByPath(path: string[]) {
    return path.reduce(
      (prev, current) =>
        current === ROOT ? prev : prev.find((o) => o.value === current)?.children ?? [],
      this.innerOptions,
    );
  }

  public getOptionPathLabel(path: string[]) {
    let options = this.innerOptions;
    return path.reduce((labels, val) => {
      const opt = options.find((o) => o.value === val);
      options = opt?.children ?? [];
      return [...labels, opt?.label ?? ''];
    }, [] as string[]);
  }

  public getTabPrePath(tabIdx: number) {
    return this.currentSelectPath.slice(0, tabIdx).join(SPLIT_CHAR);
  }

  public getOptionPath(option: CascaderOption, tabIdx: number) {
    const prePath = this.getTabPrePath(tabIdx);
    return prePath ? prePath + SPLIT_CHAR + option.value : option.value;
  }

  public getTabTitle(tab: CascaderOption[], tabIdx: number) {
    return tab.find((o) => o.value === this.currentSelectPath[tabIdx])?.label ?? '---';
  }

  public showableCheckbox(option: CascaderOption, tabIdx: number) {
    return (
      this.multiple ||
      (!option.children?.length && this.getOptionSelectedStatus(option, tabIdx).selectAll)
    );
  }

  public checkSelected(option: CascaderOption, tabIdx: number) {
    const optionPath = this.getOptionPath(option, tabIdx);
    const value = this.innerValue2Value(this.innerValue);
    return value.some((p) => p.startsWith(optionPath));
  }

  public async setCurrentPath(option: CascaderOption, tabIdx: number, skip = true) {
    this.currentSelectPath = this.currentSelectPath.slice(0, tabIdx).concat([option.value]);

    // update tab after tabs(dependences currentSelectPath) rerender.
    if (option.children?.length && skip) {
      await this.$nextTick();
      this.activeTab = tabIdx + 1;
    }
  }

  public async optionClickHandle(option: CascaderOption, tabIdx: number) {
    await this.setCurrentPath(option, tabIdx);

    // single selection and leaf node, click the cell to automatically select
    if (!this.multiple && !option.children?.length) {
      this.optionSelectChange(option, tabIdx);
    }
  }

  public getOptionSelectedStatus(option: CascaderOption, tabIdx: number) {
    const prePath = this.getTabPrePath(tabIdx);
    const leafValues = this.findAllLeafNodesPath(option, prePath);
    const value = this.innerValue2Value(this.innerValue);

    const selectAll = leafValues.every((p) => value.includes(p));
    const indeterminate = !selectAll && leafValues.some((p) => value.includes(p));

    return { selectAll, indeterminate };
  }

  // option is a leaf node, it's includes itself. and only one
  public findAllLeafNodesPath(root: CascaderOption, prePath = '') {
    const values: string[] = [];
    const queue: Array<CascaderOption & { prePath: string }> = cloneDeep([{ ...root, prePath }]);

    while (queue.length > 0) {
      const node = queue.shift()!;
      const path = node.prePath ? `${node.prePath}${SPLIT_CHAR}${node.value}` : node.value;

      if (node?.children) {
        queue.push(...node.children.map((n) => ({ ...n, prePath: path })));
      } else {
        values.push(path);
      }
    }
    return values;
  }

  public async optionSelectChange(option: CascaderOption, tabIdx: number) {
    if (!this.multiple && !!option.children?.length) {
      return;
    }

    await this.setCurrentPath(option, tabIdx, false);

    const prePath = this.getTabPrePath(tabIdx);
    const leafValues = this.findAllLeafNodesPath(option, prePath);

    let value = this.innerValue2Value(this.innerValue);

    const isCheckedAll = leafValues.every((p) => value.includes(p));
    const newChecked = !isCheckedAll;

    if (!this.multiple) {
      // single-selecting，the clicked option is always selected
      this.innerValue = this.value2InnerValue(leafValues);
    } else {
      leafValues.forEach((p) => {
        if (newChecked && !value.includes(p)) {
          value.push(p);
        } else if (!newChecked && value.includes(p)) {
          value = value.filter((o) => o !== p);
        }
      });
      this.innerValue = this.value2InnerValue(value);
    }
  }

  public getFlatterOptions(opts: Array<CascaderOption & { preValue?: string; preLabel?: string }>) {
    const cloneOpt = cloneDeep(opts);
    const options: Array<{ label: string; value: string }> = [];

    while (cloneOpt.length > 0) {
      const node = cloneOpt.shift()!;

      const optVal = node.preValue ? `${node.preValue}${SPLIT_CHAR}${node.value}` : node.value;
      const optLabel = node.preLabel ? `${node.preLabel}/${node.label}` : node.label;

      if (node.children) {
        cloneOpt.push(
          ...node.children.map((n) => ({ ...n, preValue: optVal, preLabel: optLabel })),
        );
      } else {
        options.push({ label: optLabel, value: optVal });
      }
    }

    return options;
  }
}
```

#### 样式文件

```less
.popover {
  .title {
    font-size: 16px;
    height: 48px;
    line-height: 48px;

    .selected {
      margin-left: 10px;
      font-size: 14px;
      color: #427daf;
      font-style: normal;
    }
  }

  .check-box {
    padding: 0 5px 0 10px;
  }

  .option-wrapper {
    height: 350px;
    overflow-y: auto;
  }

  .selectedCell {
    color: #409eef;
  }

  .arrow-icon {
    position: absolute;
    right: 5px;
    top: 14px;
  }
}

.p-6 {
  padding: 0 6px !important;
}
.p-16 {
  padding: 0 16px !important;
}

/deep/.cascade-input {
  input {
    text-overflow: ellipsis;
  }
}

/deep/.van-cell:active {
  background: #f2f3f5;
}

/deep/.van-tab {
  padding: 0 10px !important;
  margin: 0 5px !important;
  flex: 0 0 auto !important;
}
```

#### `SelectorList` 组件源码

```html
<template>
  <div class="select-group">
    <van-checkbox-group>
      <van-cell-group>
        <van-cell
          v-for="item in options"
          clickable
          :key="item.value"
          :title="item.label"
          :title-class="{ selectedCell: computeShow(item) }"
          @click="handleClick(item)">
          <template #right-icon>
            <van-checkbox :name="item" ref="checkboxes">
              <template #icon>
                <van-icon
                  v-show="computeShow(item)"
                  name="success"
                  color="#000"
                  style="background: #fff; border: none" />
              </template>
            </van-checkbox>
          </template>
        </van-cell>
      </van-cell-group>
    </van-checkbox-group>
  </div>
</template>

<script lang="ts">
  import { Component, ModelSync, Prop, Vue } from 'vue-property-decorator';

  @Component({ name: 'selector-list' })
  export default class SelectorList extends Vue {
    @ModelSync('value', 'input') public innerValue!: string[];

    @Prop() public readonly options?: Array<{ label: string; value: string }>;

    @Prop(Boolean) public readonly multiple!: boolean;

    public handleClick(opt: { label: string; value: string }) {
      let innerVal = this.innerValue ? [...this.innerValue] : [];

      if (!this.multiple) {
        innerVal = [opt.value];
      } else {
        const idx = innerVal.findIndex((o) => o === opt.value);

        if (idx >= 0) {
          innerVal.splice(idx, 1);
        } else {
          innerVal.push(opt.value);
        }
      }
      this.innerValue = innerVal;
    }

    public computeShow(opt: { label: string; value: string }) {
      return this.innerValue.some((o) => o === opt.value);
    }
  }
</script>

<style lang="less" scoped>
  .select-group {
    height: 400px;
    display: flex;
    flex-direction: column;
    overflow: auto;

    -webkit-overflow-scrolling: touch;

    .selectedCell {
      color: #409eef;
    }

    .van-cell {
      font-size: 14px;
      height: 35px;
      padding: 5px 16px;
    }
  }
</style>
```

### 最终效果

[![mobile-cascader-multiple-p1](/images/share/mobile-cascader-multiple/p1.png)](/images/share/mobile-cascader-multiple/p1.png)
