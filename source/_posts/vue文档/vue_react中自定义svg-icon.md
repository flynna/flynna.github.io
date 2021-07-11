---
title: vue_react中自定义svg-icon
date: 2021-06-08 15:51:00
tags: [svg-icon]
# type: "categories"
categories: [大前端, Vue]
---

### vue 中定义 svg-icon

#### svg-icon 组件

```vue
<template>
  <div
    v-if="isExternal"
    :style="styleExternalIcon"
    class="svg-external-icon svg-icon"
    v-on="$listeners"
  />
  <svg v-else :class="svgClass" aria-hidden="true" v-on="$listeners">
    <title v-if="title">{{ title }}</title>
    <use :xlink:href="iconName" />
  </svg>
</template>

<script>
// doc: https://panjiachen.github.io/vue-element-admin-site/feature/component/svg-icon.html#usage
function isExternal(path) {
  return /^(https?:|mailto:|tel:)/.test(path);
}

export default {
  name: "SvgIcon",
  props: {
    iconClass: {
      type: String,
      required: true,
    },
    className: {
      type: String,
      default: "",
    },
    title: {
      type: String,
      default: null,
    },
  },
  computed: {
    isExternal() {
      return isExternal(this.iconClass);
    },
    iconName() {
      return `#icon-${this.iconClass}`;
    },
    svgClass() {
      if (this.className) {
        return `svg-icon ${this.className}`;
      }
      return "svg-icon";
    },
    styleExternalIcon() {
      return {
        mask: `url(${this.iconClass}) no-repeat 50% 50%`,
        "-webkit-mask": `url(${this.iconClass}) no-repeat 50% 50%`,
      };
    },
  },
};
</script>

<style scoped>
.svg-icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}

.svg-external-icon {
  background-color: currentColor;
  mask-size: cover !important;
  display: inline-block;
}
</style>
```

#### 全局注册

` 默认情况下，Svg Icon 组件注册在`[@/icons 中](https://github.com/PanJiaChen/vue-element-admin/blob/master/src/icons/index.js#L6)`，可以在项目的任何地方使用。所有图标都可以在`[@/icons/svg 中找到](https://github.com/PanJiaChen/vue-element-admin/tree/master/src/icons/svg)`您可以自行添加或删除图标，图标会自动导入，无需手动操作。 `

`@/icons/index.js`

```javascript
import Vue from "vue";
import SvgIcon from "@/components/SvgIcon"; // svg component
// register globally
Vue.component("svg-icon", SvgIcon);

const req = require.context("./svg", false, /\.svg$/);
const requireAll = (requireContext) =>
  requireContext.keys().map(requireContext);
requireAll(req);
```

`main.ts` 中引入

> import '@/icons';

#### [文档](https://panjiachen.github.io/vue-element-admin-site/feature/component/svg-icon.html#usage)

- 使用: icon-class 名称默认为 svg 对应文件名

> ```html
> <!-- 从网址导入 <svg-icon icon-class="https://xxxx.svg" /> -->
> <svg-icon icon-class="password" class-name="custom-class" />
> ```

- 设置 svg-icon 颜色

> 默认值为 父级元素的 color，如果需要改变，直接更改父级 color 或自身 fill 属性

---

### react 中定义 svg-icon

`使用时导入即可`

```tsx
import React, { FC } from "react";

interface SvgIconProps {
  iconClass: string;
  title?: string;
  className?: string;
  style?: {
    [key: string]: string | number;
  };
  otherOptions?: {
    [key: string]: string | CallableFunction;
  };

  /**
   * click 点击事件
   * @return {*}  {(boolean | void)} 返回值为 true 时，阻止默认行为及事件冒泡
   * @memberof SvgIconProps
   */
  onClick?(): boolean | void;
}

const SvgIcon: FC<SvgIconProps> = ({
  iconClass,
  title,
  className,
  onClick,
  ...otherProps
}) => {
  return (
    <svg
      className={className ? `svgIcon ${className}` : "svgIcon"}
      aria-hidden="true"
      onClick={(e) => {
        if (onClick) {
          const isStop = onClick();
          if (isStop) {
            e.preventDefault();
            e.stopPropagation();
          }
        }
      }}
      {...otherProps}
    >
      {title && <title title={title}>{title}</title>}
      <use xlinkHref={`#${iconClass}`} />
    </svg>
  );
};

export default SvgIcon;
```
