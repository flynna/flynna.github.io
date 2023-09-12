---
title: 由 Notification 引起的奇葩问题
date: 2023-09-07 11:24:20
updated: 2023-09-07 11:24:20
tags:
  - bug
  - notification
categories:
  - 踩坑记录
  - ElementUI
---

一大早到公司上班，就被告知有一个看似无厘头的 `bug` 需要解决？

> 原话是：在电脑浏览器上能够正常访问？甚至是打开调试工具，模拟手机进行调试也可以访问，无任何异常信息...`but...`，当同样的链接放到手机真机上时，不能正常访问服务，页面一片空白？

其实要解决 `bug`，只要能发现原因所在，就算解决一半了。

然而...`目前只有..猜测？程序员怎么能靠猜呢？`，是不是也得`断点调试debugger`一套流程走下来...

`ok，如果你真的这么做了，你会发现...嘿嘿 毫无收获...😉😉😉` 因为它不抛异常，也不报错，连 `vConsole` 都没能让它在控制台留下任何异常输出.

> `vConsole:` 一个轻量、可拓展、针对手机网页的前端开发者调试面板。同样的，目前也是微信小程序的官方调试工具

<!-- more -->

### 定位问题

听上去有些无厘头，甚至我和大家开始的想法一样，无从下手进行调试，我寻思...

> 网页白屏很大程度上都是由于程序出错导致的运行中断引起的，需要重点排查。
>
> 再加之 `PC` 甚至模拟手机调试都能够正常访问，且没有任何异常提示和报错，而部分手机浏览器又可以正常访问，更大概率是浏览器兼容性引起的。

因为懒，不想一个个版本切换去找哪次提交产生的问题，也没有很好的思路，就在手机浏览器上访问本地服务~~先连上内网 wifi~~，刷新了几次链接，无意间发现有一瞬间的页面错误消息提示~~就一瞬间...零点几秒的出现时间，还是刷新后偶现的...无语 😅😅😅~~，然后就白屏了，根本就看不清错误消息。

没办法，我就打开了手机录屏，复现了，然后就在录制的视频里找关键帧...发现了这样的一句错误信息：

> `can not find variable Notification eval code...`

有错误消息，接下来就好办多了，根据提示，定位到文件，原文~~原文？`bug`文才对 😒😒😒~~如下：

```ts
import Vue from 'vue';
import { InfiniteScroll, Loading, Message, MessageBox } from 'element-ui';

Vue.use(Loading.directive);
Vue.use(InfiniteScroll);

Vue.prototype.$loading = Loading.service;
Vue.prototype.$msgbox = MessageBox;
Vue.prototype.$alert = MessageBox.alert;
Vue.prototype.$confirm = MessageBox.confirm;
Vue.prototype.$prompt = MessageBox.prompt;
Vue.prototype.$notify = Notification;
Vue.prototype.$message = Message;

...
```

找到文件后，一眼就看到了 `Notification`，这里并没有从 `element-ui` 导入...

### 解决问题

问题本身并不难解决和发现，但是当程序不抛异常，又或者哪个结构比较深的文件也出现如上的用法，挨个版本改动对比，会让你找到怀疑人生...

由于没有导入，编辑器此时把 `Notification` 当做是 `web Notification` 了。这也是为什么编辑器没有报错，能正常打包...~~心中顿时一万个曹尼玛奔腾而过 🙄🙄🙄~~

查阅文档后，发现了该 [web Notification](https://developer.mozilla.org/zh-CN/docs/Web/API/Notification) 在 `android webview` 和部分浏览器存在兼容性问题。

大多时候，其实兼容性并不会影响程序运行。本文场景是在入口文件全局挂载 `$notify`，而由于 `Notification` 的局限性，部分浏览器环境下没有该 `API` 识别不了，会当做是未声明的变量，从而抛出异常中断程序运行。

解决的方法很简单，正常导入就可以了：

```ts
import Vue from 'vue';
import { Notification } from 'element-ui';

Vue.prototype.$notify = Notification;
```

> **本文旨在提示 `element-ui Notification` 和 `web Notification` 两者共同存在，在编码过程中容易忽视混淆，而编辑器并不会给出相应提示，后续出现问题也会很难排查定位**
