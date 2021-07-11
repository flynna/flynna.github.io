---
title: css3_4 好用但不常见属性
date: 2021-06-05 16:07:22
tags: [less, css3, css4]
# type: "categories"
categories: [大前端, CSS]
---

### 1. 文本省略（指定行数）

```less
// 常规用法：单行文本省略
.text-ellipsis {
  // 一般不用加
  display: inline-block;
  min-width: 60px;
  // 固定写法
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}

// 指定文本行省略
.loop( @count )when( @count > 0 ) {
  .text-ellipsis-@{count} {
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: @count;
    /*! autoprefixer: off */
    -webkit-box-orient: vertical;
    word-break: break-all;
  }
  .loop((@count - 1));
}

.loop(5);
```

### 2. 浏览器滚动条样式

- 基础使用

```less
.custom-scroll-bar {
  &::-webkit-scrollbar {
    width: 6px;
    height: 6px;
    border-radius: 3px;
  }
  &::-webkit-scrollbar-track {
    border-radius: 3px;
    background-color: #f3f6f8;
  }
  &::-webkit-scrollbar-thumb {
    border-radius: 3px;
    background: #f1f1f1;
  }
}
```

- 通过插件实现

### 3. color-adjust 浏览器颜色调节

`打印样式（部分场景下，需要浏览器保留界面设置的背景边框颜色等样式）`

```less
@media print {
  .template-view {
    // 默认值 economy
    -webkit-print-color-adjust: exact;
  }
}
```

### 4. 数字类型输入框-禁用按钮

`部分场景下 如 type 为 number 的输入框，尾部会出现加减按钮`

```css
/* chrome */
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  /* display: none; <- Crashes Chrome on hover */
  -webkit-appearance: none;
  margin: 0; /* <-- Apparently some margin are still there even though it's hidden */
}

/* firefox */
input[type="number"] {
  -moz-appearance: textfield; /* Firefox */
}
```

### 5. 媒体查询

```css
@media screen||all and (max-width: 600px) {
  /*当屏幕宽度不超过 600 的时候   <600  显示的样式*/
}
@media screen||all and (min-width: 600px) and (max-width: 1000px) {
  /*当屏幕宽度处于600-1000*/
}
@media screen||all and (min-width: 1000px) {
  /*当屏幕宽度>1000*/
}
```
