---
title: 基于 Vuepress 的静态网站生成器实践
date: 2023-09-24 20:07:10
updated: 2023-09-24 20:07:10
tags:
  - vuepress
  - 静态网站
categories:
  - 工具学习
  - Vuepress
---

> `VuePress` 由两部分组成：第一部分是一个极简静态网站生成器 (`opens new window`)，它包含由 `Vue` 驱动的主题系统和插件 `API`，另一个部分是为书写技术文档而优化的默认主题，它的诞生初衷是为了支持 `Vue` 及其子项目的**文档需求**

之前自己写了一个简版的 `Vue` 组件库，但是没有写文档，所以就想着动手实际操作并记录下来。

<!-- more -->

### 文档项目搭建

根目录下新增 `docs` 文件夹，作为文档项目的根节点，再初始化：

```bash
mkdir docs && cd ./docs && npm init -y

yarn add -D vuepress
```

添加项目脚本指令：

```json
"scripts": {
  "docs:dev": "vuepress dev .",
  "docs:build": "vuepress build ."
},
```

启动查看效果：

```bash
npm run docs:dev
```

### 文档配置调整

默认的文档搭建完成后，只能通过切换网址路由来跳转各个文档，很不方便...然后看到可以自定义主题，根据主题配置侧边栏、导航栏、主页等等...详见：[https://vuepress.vuejs.org/zh/theme/default-theme-config.html](https://vuepress.vuejs.org/zh/theme/default-theme-config.html)

下面是我配置的案例：

#### 开启主页显示

即项目根路由页面显示配置，仅做参考：

```md
<!-- /docs/README.md -->

---

home: true
heroImage: /favicon.ico
heroText: vui-project
tagline: 轻量、可定制的移动端 Vue 组件库
actionText: 快速上手 →
actionLink: /config.html
features:

- title: 简洁至上
  details: 以 Markdown 为中心的项目结构，以最少的配置帮助你专注于写作。
- title: Vue 驱动
  details: 享受 Vue + webpack 的开发体验，在 Markdown 中使用 Vue 组件，同时可以使用 Vue 来开发自定义主题。

---
```

#### 添加侧边栏、导航栏配置

新建 `.vuepress` 文件夹（请先查看[目录结构说明](https://vuepress.vuejs.org/zh/guide/directory-structure.html)），新建 `config.js`：

```js
// /docs/.vuepress/config.js

module.exports = {
  themeConfig: {
    logo: '/favicon.ico',
    lastUpdated: 'Last Updated',
    sidebar: {
      '/config/': ['/config.html'],
      '/componentsDoc/': [
        'button' /* /componentsDoc/button.html */,
        'text' /* /componentsDoc/text.html */,
      ],
    },
    nav: [
      { text: 'Home', link: '/' },
      { text: 'Guide', link: '/config.html' },
      { text: 'docs', link: '/componentsDoc/button' },
      {
        text: 'Languages',
        items: [
          { text: '简体中文', link: '/language/chinese' },
          { text: '英文', items: '/language/english' },
        ],
      },
      {
        text: 'Github',
        link: 'https://github.com/flynna/vui-project/tree/main/docs',
        target: '_blank',
        rel: 'noopener noreferrer',
      },
    ],
  },
};
```

其中的以 `/` 开头的都是页面可访问路由，需要添加对应的 `.md` 文件。[去看源码](https://github.com/flynna/vui-project/tree/main/docs)

#### 主页效果

[![vuepress-p1](/images/tools/vuepress/p1.png)](/images/tools/vuepress/p1.png)

### 文档实现

文档内容均是由 `.md` 文档编写，遵循 `markdown` 语法：

[![vuepress-p2](/images/tools/vuepress/p2.png)](/images/tools/vuepress/p2.png)

好像缺了点什么？哦对，实时渲染...不说实施编译，至少应该将组件效果呈现出来才对...

#### 添加 `vui-project` 组件库

将之前写的 `UI` 库组件引进来，全局注册...（官方默认说明：`.vuepress/components/*.vue` 的组件会自动全局注册）

说实话，将 `packages/` 的组件实现再在 `.vuepress/components/` 写一次是很冗余的事情...

翻了文档，有以下**解决办法**：

- 1. 添加插件 [@vuepress/plugin-register-components](https://vuepress.vuejs.org/zh/plugin/official/plugin-register-components.html) 配置自定义组件的路径和组件名称。

- 2. 通过添加 `enhanceApp.js` 文件注册第三方库的方式，将 `vui-project` 集成进来。

`but` 由于我的组件库是由 `vue3` 写的，和 `vuepress` 存在兼容性问题，对我都不太适用...所以尝试着查阅了一下，果然有相关的 [issue](https://github.com/vuejs/vuepress/issues/2550)，可以发现，官方尝试着发行新的库 [vuepress-next](https://github.com/vuepress/vuepress-next) 来解决这个问题，但目前还没有正式版...

所以为了测试组件在 `vuepress` 的可用性，我就直接在 `/docs/.vuepress/components/` 下写了一份 `vue2` 的同功能 `VuiText` 组件代码。

#### 效果呈现

[![vuepress-p3](/images/tools/vuepress/p3.png)](/images/tools/vuepress/p3.png)