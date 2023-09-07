---
title: 由 Notification 引起的奇葩问题
date: 2023-09-07 11:24:20
updated: 2023-09-07 11:24:20
tags:
  - bug
  - notification
categories:
  - 踩坑记录
  - Vue
---

### 吐槽

一大早到公司上班，就被告知有一个看似无厘头的 `bug` 需要解决？

原话是：在电脑浏览器上能够正常访问？甚至是打开调试工具，模拟手机进行调试也可以访问，无任何异常信息...`but...`，当同样的链接放到手机上时，不能正常访问服务，页面一片空白？

其实要解决 `bug`，只要能发现原因所在，就算解决一半了。

然而...`目前只有..猜测？程序员怎么能靠猜呢？`，是不是也得`断点调试debugger`一套流程走下来...

`ok，如果你真的这么做了，你会发现...嘿嘿 毫无收获...😉😉😉` 因为它不抛异常，也不报错，连控制台都没有任何输出

只有复现出问题的场景才能定位问题了，然而场景就是...`手机真机浏览器无法访问...网页模拟手机可以访问`

`why?` 因为手机上不会提示网页异常消息，也不能打开调试控制台，怎么办呢？总不能再装个虚拟机吧...有点搞复杂了...

正常情况下，如果不能找到直接发现异常，那只能通过切换版本来根据版本变更确定是那次提交出的问题。

<!-- more -->

### 分析原因

听上去有些无厘头，甚至我和大家开始的想法一样，无从下手进行调试，我寻思...

- 环境问题：生产环境和开发环境下的不同配置导致编译后的不同表现。

- 版本问题：之前的版本手机上是能够正常访问的，中间发布了很多个版本，部分版本没有进行测试，需要定位到具体的版本和出错时间。

- 代码异常：网页白屏很大程度上都是由于程序出错导致的运行中断引起的，需要重点排查。

- 兼容性问题：`PC` 甚至模拟手机调试都能够正常访问，切没有任何异常提示和报错，而部分手机浏览器又可以正常访问，大概率是浏览器兼容性引起的。

通过本地项目 `run` 起来在自己手机上运行后，排除了环境的影响，当前版本最新代码确实有这个问题存在，代码异常正常情况下是无法编译成功的，那就应该是兼容性问题了。~~当时的我很自信的这样认为的 😂😂😂~~

### 定位问题

因为懒，不想一个个版本切换去找哪次提交产生的问题，也没有很好的思路，就在手机浏览器上刷新了几次链接，无意间发现有一瞬间的错误消息提示~~就一瞬间...零点几秒的出现时间，还是刷新后偶现的...无语 😅😅😅~~，然后就白屏了，根本就看不清错误消息。

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
