---
title: 移动端调试神器之 vConsole
date: 2023-09-15 10:11:04
updated: 2023-09-15 10:11:04
tags:
  - debug
  - vConsole
categories:
  - 工具学习
  - 程序调试
---

前面开发移动端功能的时候，发现调试并不是很方便，主要体现在部分兼容性问题和体验上，浏览器模拟的手机并不能完全替代。

部分功能只有上线后，才能在手机上发现错误，然后机缘巧合下发现了腾讯开源的 `vConsole` 工具。

> [`vConsole`](https://github.com/Tencent/vConsole/blob/dev/README_CN.md) 一个轻量、可拓展、针对手机网页的前端开发者调试面板。和框架无关，可以在任何地方进行使用。（微信小程序的官方调试工具）

<!-- more -->

### 特性介绍

- 查看 `console` 日志

- 查看网络请求

- 查看页面 `element` 结构

- 查看 `cookie、localStorage、sessionStorage`

- 查看系统版本相关信息

- 执行命令行脚本

- 自定义插件

可以理解为就是浏览器 `DevTools` 的一个缩略版。

### 安装及初始化

#### `cdn` 方式

```html
<!-- 安装 -->
<script src="https://unpkg.com/vconsole@latest/dist/vconsole.min.js"></script>
<!-- 使用 -->
<script>
  // VConsole 默认会挂载到 `window.VConsole` 上
  var vConsole = new window.VConsole();
</script>
```

相关资源：

> `https://unpkg.com/vconsole@latest/dist/vconsole.min.js` > `https://cdn.jsdelivr.net/npm/vconsole@latest/dist/vconsole.min.js`

#### `npm` 方式

```bash
npm install vconsole
```

```ts
import VConsole from 'vconsole';

const vConsole = new VConsole();

// 完成调试后，可销毁 vConsole
vConsole.destroy();
```

> `TIPS:` **VConsole 实例化后，项目里所有的 `console.log` 语句以及错误信息、网络请求等等，都默认会在 `vConsole` 面板进行展示，无需手动额外操作** ~~有个前提就是 `vConsole` 完成挂载之后的日志才会在 `vConsole` 面板显示 😲😲😲~~

### 使用

在 `vConsole` 模块成功引入后，会在页面的右下角出现 `vConsole` 按钮，可以展开/收起调试面板。**注意在生产环境屏蔽该功能**。

支持原生 `console` 的几乎大部分功能和语法。`eg:`

```ts
import VConsole from 'vconsole';

const vConsole = new VConsole();

// 测试打印信息均会展示在 vConsole 面板中
console.log({ name: 'hello', age: 18 }); // 支持 object 和 array 等类型变量打印
console.log([{ name: 'hello', age: 18 }]);
console.debug('hello world ', 'hhh'); // 支持多参数打印
console.error('hello world');
console.warn('hello world');
console.info('hello world');
console.table([{ name: 'hello', age: 18 }]); // 不支持
console.dir(document.body); // 不支持
console.time('foo');
console.timeEnd('foo');
console.clear();
// ...
```

效果如下图所示：

[![v-console-p1](/images/tools/v-console/p1.png)](/images/tools/v-console/p1.png)

---

本文旨在介绍该工具在移动开发过程中带来的便利，方便后续有此需求是进行调试。详细文档和插件实现以及其原理见官方文档：

### 文档

#### `vConsole` 本体

初始化 `option` 配置见如下：公共属性及方法

- [使用教程](./doc/tutorial_CN.md)
- [公共属性及方法](./doc/public_properties_methods_CN.md)
- [内置插件：属性及方法](./doc/plugin_properties_methods_CN.md)

#### 自定义插件

- [插件：入门](./doc/plugin_getting_started_CN.md)
- [插件：编写插件](./doc/plugin_building_a_plugin_CN.md)
- [插件：Event 事件列表](./doc/plugin_event_list_CN.md)

### 第三方插件列表

- [vConsole-sources](https://github.com/WechatFE/vConsole-sources)
- [vconsole-webpack-plugin](https://github.com/diamont1001/vconsole-webpack-plugin)
- [vconsole-stats-plugin](https://github.com/smackgg/vConsole-Stats)
- [vconsole-vue-devtools-plugin](https://github.com/Zippowxk/vue-vconsole-devtools)
- [vconsole-outputlog-plugin](https://github.com/sunlanda/vconsole-outputlog-plugin)
- [vite-plugin-vconsole](https://github.com/vadxq/vite-plugin-vconsole)

---

> 文档和第三方插件列表摘自：官方文档：[https://github.com/Tencent/vConsole/blob/dev/README_CN.md#L160-L186](https://github.com/Tencent/vConsole/blob/dev/README_CN.md#L160-L186)
