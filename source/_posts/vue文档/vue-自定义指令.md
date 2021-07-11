---
title: vue-directive 自定义指令
date: 2021-06-05 16:07:22
tags: [v-指令, directive]
# type: "categories"
categories: [大前端, Vue]
---

### 指令参数介绍

- `el`：指令所绑定的元素，可以用来直接操作 DOM。

- ```
  binding
  ```

  ：一个对象，包含以下 property：

  - `name`：指令名，不包括 `v-` 前缀。
  - `value`：指令的绑定值，例如：`v-my-directive="1 + 1"` 中，绑定值为 `2`。
  - `oldValue`：指令绑定的前一个值，仅在 `update` 和 `componentUpdated` 钩子中可用。无论值是否改变都可用。
  - `expression`：字符串形式的指令表达式。例如 `v-my-directive="1 + 1"` 中，表达式为 `"1 + 1"`。
  - `arg`：传给指令的参数，可选。例如 `v-my-directive:foo` 中，参数为 `"foo"`。
  - `modifiers`：一个包含修饰符的对象。例如：`v-my-directive.foo.bar` 中，修饰符对象为 `{ foo: true, bar: true }`。

- `vnode`：Vue 编译生成的虚拟节点。

- `oldVnode`：上一个虚拟节点，仅在 `update` 和 `componentUpdated` 钩子中可用。

`举栗项目中 实际运用到的场景`

### 1. 按钮自定义文案

`config.ts`

```typescript
export interface ButtonTextSetting {
  action:
    | "saveDraft"
    | "submit"
    | "pass"
    | "refuse"
    | "rollback"
    | "tempSave"
    | "dataExport";
  type: string;
  defaultText: string;
  text: string;
  visible: boolean;
}

export const DefaultBtnSettings: ButtonTextSetting[] = [
  {
    action: "saveDraft",
    type: "info",
    defaultText: "存至草稿箱",
    text: "",
    visible: true,
  },
  {
    action: "submit",
    type: "primary",
    defaultText: "提交",
    text: "",
    visible: true,
  },
  {
    action: "pass",
    type: "primary",
    defaultText: "通过",
    text: "",
    visible: true,
  },
  {
    action: "refuse",
    type: "danger",
    defaultText: "拒绝",
    text: "",
    visible: true,
  },
  {
    action: "rollback",
    type: "warning",
    defaultText: "回退",
    text: "",
    visible: true,
  },
  {
    action: "tempSave",
    type: "plain",
    defaultText: "暂存",
    text: "",
    visible: true,
  },
  {
    action: "dataExport",
    type: "plain",
    defaultText: "打印数据",
    text: "",
    visible: true,
  },
];
```

`@/src/directives/buttonText.ts`

```typescript
import {
  ButtonTextSetting,
  DefaultBtnSettings,
} from "@/components/processNodeSetting/components/ButtonTextConfig";
import { deepClone } from "@/utils";
import Vue from "vue";

Vue.directive("button-text", {
  async bind(el, binding) {
    const value = deepClone(binding.value) as {
      action: ButtonTextSetting["action"];
      buttonTextSettings: ButtonTextSetting[];
    };

    const { action, buttonTextSettings } = value;
    const defaultBtnSettings = DefaultBtnSettings;
    let btnConfig = buttonTextSettings.find(
      (btnSetting) => btnSetting.action === action
    );
    // 节点上没有配置则读默认配置
    if (!btnConfig) {
      btnConfig = defaultBtnSettings.find(
        (btnSetting) => btnSetting.action === action
      );
    }

    if (!btnConfig) {
      return console.warn("没有找到按钮配置");
    }

    if (!btnConfig.visible) {
      await Vue.nextTick();
      el.parentNode?.removeChild(el);
      return;
    }

    // 当指令用于 el-button 时
    if (el.classList.contains("el-button")) {
      const span = el.querySelector("span");
      span && (span.innerText = btnConfig.text || btnConfig.defaultText);

      // 修改按钮的class
      el.classList.remove("el-button--default");
      el.classList.add(`el-button--${btnConfig.type}`);
    } else {
      // 用于移动端的按钮时( 前提：需要保证移动端的按钮必须包含一个classname为custom-text的span子元素 )
      const span = el.querySelector(".custom-text") as HTMLSpanElement;
      span && (span.innerText = btnConfig.text || btnConfig.defaultText);
    }
  },
});
```

`具体使用：`

```vue
<!-- buttonTextSettings 参数同 config -->
<el-button
  v-button-text="{ buttonTextSettings, action: 'saveDraft' }"
  type="info"
  plain
>存至草稿箱</el-button>
```

### 2. 按钮权限控制

`顾名思义，通过判断登录用户的权限，控制按钮是否应该渲染`

```typescript
import Vue from "vue";
import { DirectiveBinding } from "vue/types/options";

export interface PermissionFlag {
  [index: string]: boolean | undefined;
  isDefaultRole: boolean;
  isAppManager: boolean;
  isAppCreator?: boolean;
  isFormManager?: boolean;
  isFormMember?: boolean;
  isOwner?: boolean;
}

Vue.directive("permissions", {
  async bind(el, banding) {
    await permissionDirective_combined(el, banding);
  },
});

async function permissionDirective_combined(
  el: HTMLElement,
  banding: DirectiveBinding
) {
  // 是否是系统管理员 是则不屏蔽
  const isDefaultRole = !!store.getters.isSysManager;
  if (isDefaultRole) return;

  // 当前用户的权限信息
  const permissions = store.getters.permissions;

  // permission 按钮自身权限的 action 及 module, permissionFlag (非配置的权限信息，如是否为应用创建者)
  const { permission, permissionFlag } = banding.value as {
    permission: string;
    permissionFlag?: Partial<PermissionFlag>;
  };
  // 当前需要完成权限控制的按钮  action 及 module
  const [module, action] = permission.split(":");

  const accredited = permissionCheck(module, action, permissions);

  const flag = permissionFlag ? ObjectPropsOrOperation(permissionFlag) : true;

  if (!(accredited && flag)) {
    await Vue.nextTick();
    el.parentNode?.removeChild(el);
  }
}

function ObjectPropsOrOperation<T extends Partial<PermissionFlag>>(obj: T) {
  let res = false;
  Object.keys(obj).forEach((item) => {
    res = res || Boolean(obj[item]);
  });
  return res;
}
```

### 3. 自定义遮罩指令

`部分组件场景需要给元素添加遮罩层，禁止用户点击`

```typescript
import Vue from "vue";

Vue.directive("mask", {
  bind(el) {
    // 父元素没有定位属性设置为 relative
    const position = getComputedStyle(el).position;
    if (!["relative, absolute"].includes(position)) {
      el.style.position = "relative";
    }
    const maskDiv = document.createElement("div");
    Object.assign(maskDiv.style, {
      height: "100%",
      width: "100%",
      position: "absolute",
      top: "0",
      left: "0",
      background: "transparent",
      zIndex: 1,
    });
    el.appendChild(maskDiv);
  },
});
```
