---
title: element-ui 自定义挂载message
date: 2021-05-16 15:05:00
tags: [element-ui]
# type: "categories"
categories: [大前端, Vue]
---

### 全局修改 $message 的默认配置项

`Element 为 Vue.prototype 添加了全局方法 $message。因此在 vue instance 中可以采用本页面中的方式调用 Message。`

`目的：在不逐一单独配置的情况下，全局覆盖并生成默认条件，如：offset 偏移...`

- plugins/element.ts

```javascript
import ElementUI, { Message } from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
import { ElMessageOptions, MessageType } from 'element-ui/types/message';
import Vue from 'vue';

/* 全局配置引用组件 size 及 modal 的 zIndex */
Vue.use(ElementUI, { size: 'mini', zIndex: 1999 });
/* 若按需引入， 则需单独挂载 config */
// Vue.prototype.$ELEMENT = { size: 'mini', zIndex: 1999 }

/* 修改默认配置 --- defaultOptions */
function CustomMessage(options: string | ElMessageOptions) {
  const defaultOptions = { offset: 120 };
  const transformOpt = typeof options === 'string' ? { message: options } : options;
  Message({ ...defaultOptions, ...transformOpt });
}

/* message 自身的属性方法 */
(['success', 'warning', 'info', 'error'] as MessageType[]).forEach((type) => {
  (CustomMessage as any)[type] = (options: string | ElMessageOptions) => {
    const transformOpt = typeof options === 'string' ? { message: options } : options;
    CustomMessage({ ...transformOpt, type });
  };
});

CustomMessage.prototype = Message.prototype;
/* 重新挂载 $message */
Vue.prototype.$message = CustomMessage;
```

### message-box 组件提示 message

```javascript
export function messageBox(
  vm: Vue,
  message: string,
  option?: {
    icon?: string,
    type?: MessageType,
    description?: string,
  }
) {
  ElementUI.Message({
    message: vm.$createElement("message-box", {
      props: {
        icon: option?.icon,
        type: option?.type ?? "success",
        message,
        description: option?.description,
      },
    }),
    customClass: "custom-message",
    offset: 120,
    duration: 2000,
  });
}
```

### 自定义 $message-vNode

`目的：部分场景中，我们需要有不同类型及样式的提示弹窗。`

```javascript
import A from './A.vue';

// ...
public showMessage() {
  const render = this.$createElement;
  this.$msgbox({
    title: 'title',
    message: render('A'),
    distinguishCancelAndClose: true,
    showCancelButton: true,
    confirmButtonText: '确认1',
    cancelButtonText: '确定2',
  })
}
```
