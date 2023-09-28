---
title: 基于浏览器书签栏实现可执行脚本
date: 2023-09-27 15:12:32
updated: 2023-09-27 15:12:32
tags:
  - 书签栏
  - 浏览器
categories:
  - 技术分享
  - 脚本
---

偶然的一次机会，发现浏览器地址栏里可以执行 `javascript` 代码，~~诶...直接发现新大陆了这是 🤩🤩🤩~~

基于这个特性，可以实现一些有趣的功能脚本，比如：表单自动填写、粘贴复制、快速测试...

> **除了通过书签的方式执行脚本外，你还可以将脚本内容封装到[油猴里](extension://dhdgffkkebhmkfjojejmpbldmpobfkfo/options.html#url=&nav=dashboard)，作为插件在指定域名下自动启用**。~~它俩各有各的优缺点和使用场景，这里不对油猴做详细介绍。~~

<!-- more -->

### 实现方案

在浏览器地址栏执行 `javascript` 脚本，需要遵循特定的 `URL Scheme` 协议开头，用以提示浏览器执行代码。

常用的格式：`javascript:xxx`

<div class="info">

> `javascript:` 是协议前缀，`xxx` 是有效的可执行 `js` 代码。
>
> 由于 `URL` 限制，脚本最好是不要有空格，或者你可以用小括号将代码扩起来，将其看做是一个整体
>
> 除此之外，如果你不想在执行脚本后页面发生跳转，应该避免代码的最后的返回值是字符串...如果你不清楚代码的返回值是什么，可以将脚本代码放入调试控制台执行一次。

</div>

### 简单测试

```bash
javascript:alert('Hello World!')
```

将上面代码粘贴到浏览器地址栏，回车会弹出 `Hello World!` 提示框。

```bash
# 脚本返回值是 string，页面会发生跳转
javascript:a='1'
# 不会跳转，因为返回值是数字
javascript:a='1';b=2
```

既然 `javascript:` 后面跟的是可执行代码，那么也就意味着我们可以将代码放入立即执行函数中，通过函数调用的方式来执行。当然这也是我推荐的一种方式：

> 形如： `javascript:(function(item) {})(args)`

`ok`，知道怎么用了，我们可以将这些脚本设置为书签，书签地址即为我们写的脚本内容。下面是一些之前用到的脚本记录：

### 脚本记录

#### `CSDN` 免登陆复制

方案一：

```js
javascript: (function () {
  window.oncontextmenu = document.oncontextmenu = document.oncopy = null;
  [...document.querySelectorAll('body')].forEach((dom) => (dom.outerHTML = dom.outerHTML));
  [...document.querySelectorAll('body, body *')].forEach((dom) => {
    [
      'onselect',
      'onselectstart',
      'onselectend',
      'ondragstart',
      'ondragend',
      'oncontextmenu',
      'oncopy',
    ].forEach((ev) => dom.removeAttribute(ev));
    dom.style['user-select'] = 'auto';
  });
})();
```

方案二：

修改文档设计模式

```js
document.designMode = 'on';
```

#### 持续更新中...
