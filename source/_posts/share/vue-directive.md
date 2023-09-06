---
title: vue-directive 指令的自定义封装案例
date: 2023-08-15 18:19:34
updated: 2023-08-15 18:19:34
tags:
  - vue
  - 指令
  - directive
categories:
  - 技术分享
  - Vue
---

### 了解更多

一个自定义指令由一个包含类似组件生命周期钩子的对象来定义.

> [自定义指令： https://vuejs.org/guide/reusability/custom-directives.html](https://vuejs.org/guide/reusability/custom-directives.html)

下面是在 `vue` 中可能用到的一些自定义指令实现：

<!-- more -->

### 容器遮罩

部分组件场景需要给元素添加遮罩层，禁止用户点击。

```ts
import Vue from 'vue';

Vue.directive('mask', {
  bind(el) {
    const maskDiv = document.createElement('div');
    const position = getComputedStyle(el).position;

    if (!['relative, absolute'].includes(position)) {
      el.style.position = 'relative'; // 给父元素设置定位
    }

    Object.assign(maskDiv.style, {
      height: '100%',
      width: '100%',
      position: 'absolute',
      top: '0',
      left: '0',
      background: 'transparent',
      zIndex: 1,
    });
    el.appendChild(maskDiv);
  },
});
```

### 按钮权限

在实际项目中，有很多根据用户权限进行展示的操作按钮，比如某些按钮只有管理员才能操作，那么我们可以通过自定义指令来实现。

在这个指令中，接受的参数就是当前按钮的所需权限和用户已有权限(也可直接从 store 中获取用户已有权限)，如果用户权限中包含按钮所需权限，则显示按钮，否则不显示。

```ts
import Vue from 'vue';

Vue.directive('permissions', {
  bind(el, banding) {
    const { permission, userPermission } = banding.value as {
      permission: string;
      userPermission: string[];
    };

    // 实际需要根据 permission 的结构和类型进行判断处理，我这里示例给出的 string
    const isPermission = userPermission.includes(permission);

    if (!isPermission) {
      await Vue.nextTick();
      el.parentNode?.removeChild(el);
    }
  },
});
```

### 空状态文字显示

部分组件容器在没有获取到值，或者没有子组件的时候，需要向用户展示一个空状态的文字，那么我们可以通过自定义指令来实现。

```ts
import Vue from 'vue';
import type { CSSProperties } from 'vue/types/jsx';

interface Empty {
  content: string;
  visible: boolean;
}

function createEmptySpanCtx(content: string, className: string) {
  const emptyEl = document.createElement('span');

  emptyEl.innerText = content;
  emptyEl.className = className;

  Object.assign(emptyEl.style, {
    position: 'absolute',
    top: '50%',
    left: '50%',
    transform: 'translate(-50%, -50%)',
    fontSize: '14px',
    color: '#909399',
    background: 'transparent',
  } as CSSProperties);

  return emptyEl;
}

function triggerEmptyEl(containerEl: HTMLElement, bindValue: Empty) {
  const emptyClassName = 'empty-span';
  const emptyEls = containerEl.querySelectorAll(`.${emptyClassName}`);

  if (bindValue.visible && !emptyEls.length) {
    const emptyEl = createEmptySpanCtx(bindValue.content, emptyClassName);

    containerEl.appendChild(emptyEl);
  } else if (!bindValue.visible && emptyEls.length) {
    emptyEls.forEach((el) => {
      containerEl.removeChild(el);
    });
  }
}

Vue.directive('empty', {
  bind(el: HTMLElement, binding: { value: Empty }) {
    const position = getComputedStyle(el).position;

    if (!['relative, absolute'].includes(position)) {
      el.style.position = 'relative';
    }

    triggerEmptyEl(el, binding.value);
  },
  update(el: HTMLElement, binding: { value: Empty }) {
    triggerEmptyEl(el, binding.value);
  },
});
```
