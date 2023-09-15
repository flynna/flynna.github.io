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

### 源码剖析

#### 思路

- 通过 `window` 监听页面加载，加载完成后向页面追加调试相关的 `DOM`。

- 类似 `log、network` 面板等相关的渲染显示，则是通过重写 `window` 下对应的系统方法，追加一些自定义操作完成。

#### 原理实现

##### 构造函数

入口在 `src/core/core.js`，采用单例模式，只允许有一个 `vconsole` 实例：

```js
// ...
class VConsole {
  constructor(opt) {
    if (!!$.one(VCONSOLE_ID)) {
      console.debug('vConsole is already exists.');
      return;
    }
// ...
```

`vConsole` 面板里的模块则是通过插件的形式集成进去的独立模块：

```js
// built-in plugins
import VConsolePlugin from '../lib/plugin.js';
import VConsoleLogPlugin from '../log/log.js';
import VConsoleDefaultPlugin from '../log/default.js';
import VConsoleSystemPlugin from '../log/system.js';
import VConsoleNetworkPlugin from '../network/network.js';
import VConsoleElementPlugin from '../element/element.js';
import VConsoleStoragePlugin from '../storage/storage.js';
```

##### 加载 `DOM`

监听 `window` 事件，确定加载结束后挂载 `vconsole` 相关 `dom`：

```js
// try to init
let _onload = function () {
  if (that.isInited) {
    return;
  }
  that._render();
  that._mockTap();
  that._bindEvent();
  that._autoRun();
};
if (document !== undefined) {
  if (document.readyState === 'loading') {
    $.bind(window, 'DOMContentLoaded', _onload);
  } else {
    _onload();
  }
} else {
  // if document does not exist, wait for it
  let _timer;
  let _pollingDocument = function () {
    if (!!document && document.readyState == 'complete') {
      _timer && clearTimeout(_timer);
      _onload();
    } else {
      _timer = setTimeout(_pollingDocument, 1);
    }
  };
  _timer = setTimeout(_pollingDocument, 1);
}
```

##### `Log` 模块实现

通过重写 `window.console..` 的相关方法，执行自己的 `printLog` 并记录，最后渲染到 `vconsole` 面板：

```js
// src/log/log.js
// ...
/**
 * replace window.console with vConsole method
 * @private
 */
mockConsole() {
  const that = this;
  const methodList = ['log', 'info', 'warn', 'debug', 'error'];

  if (!window.console) {
    window.console = {};
  } else {
    methodList.map(function (method) {
      that.console[method] = window.console[method];
    });
    that.console.time = window.console.time;
    that.console.timeEnd = window.console.timeEnd;
    that.console.clear = window.console.clear;
  }

  methodList.map(method => {
    window.console[method] = (...args) => {
      this.printLog({
        logType: method,
        logs: args,
      });
    };
  });
  // ...
}
```

当 `vconsole destroy` 后，恢复 `window.console` 方法：

```js
// src/log/log.js
onRemove() {
  window.console.log = this.console.log;
  window.console.info = this.console.info;
  window.console.warn = this.console.warn;
  window.console.debug = this.console.debug;
  window.console.error = this.console.error;
  window.console.time = this.console.time;
  window.console.timeEnd = this.console.timeEnd;
  window.console.clear = this.console.clear;
  this.console = {};

  const idx = ADDED_LOG_TAB_ID.indexOf(this.id);
  if (idx > -1) {
    ADDED_LOG_TAB_ID.splice(idx, 1);
  }
}
```

##### `Network` 模块实现

也是重写了原生的 `window.XMLHttpRequest.prototype.open/send` 等方法，添加了拦截器：

```js
// src/network/network.js
/**
 * mock ajax request
 * @private
 */
mockAjax() {
  let _XMLHttpRequest = window.XMLHttpRequest;
  if (!_XMLHttpRequest) { return; }

  let that = this;
  let _open = window.XMLHttpRequest.prototype.open,
      _send = window.XMLHttpRequest.prototype.send;
  that._open = _open;
  that._send = _send;

  // mock open()
  window.XMLHttpRequest.prototype.open = function() {
    let XMLReq = this;
    let args = [].slice.call(arguments),
  // ...
  // mock send()
  window.XMLHttpRequest.prototype.send = function() {
    let XMLReq = this;
    let args = [].slice.call(arguments),
        data = args[0];
    // ...
```

> 篇幅原因就不完整呈现源码了，另外文档和第三方插件列表摘自：官方文档：[https://github.com/Tencent/vConsole/blob/dev/README_CN.md#L160-L186](https://github.com/Tencent/vConsole/blob/dev/README_CN.md#L160-L186)
