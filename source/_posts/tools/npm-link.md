---
title: 手把手教你使用 npm-link 软链
date: 2022-07-27 13:39:08
updated: 2022-07-27 13:39:08
tags:
  - npm
  - npm-link
categories:
  - 工具学习
  - 包管理器
---

### 浅谈一下

前段时间，开发了一个组件，需要在本地`debug`验证相关功能，有些纠结是用`npm link`还是`npx link`，亦或是`npm install`。。。

有些忘记了相关用法以及带来的影响...觉得还是有必要写一篇文章记录一下过程，以及新的东西。😮‍💨😮‍💨😮‍💨

### `npm link`是干啥的？

[npm 文档](https://docs.npmjs.com/cli/v7/commands/npm-link) 也有说明：为了方便你的迭代测试，`npm link`将在全局的`node_modules`创建一个指向当前自定义的`npm`模块的软链接，并在你需要使用的地方通过`npm link packagename`创建一个从全局安装的`package-name`到`当前文件夹/node_modules`的符号链接。你对自定义包的任何改动都将同步到`link`到的项目中。~~可以理解为 mklink~~

<!-- more -->

### 第一视角带你学习`link`指令

#### 准备好预发布包

```bash
mkdir my-test-component-zhl
cd ./my-test-component-zhl
npm init -y

# 准备入口文件 index
# (() => {
#   console.log('这是我的组件模块');
# })();
```

#### 准备一个用于测试的项目\*

```bash
cd ../
mkdir my-test-project
cd ./my-test-project
npm init -y
```

#### 为预发布包创建软链接

```bash
cd ../my-test-component-zhl
npm link
```

结果预览：

[![npm-link-p1](/images/tools/npm-link/p1.png)](/images/tools/npm-link/p1.png)

链接的位置就是你`nodejs`安装目录下的`node_modules`目录下，指向了当前的组件。

#### 在项目中引用这个预发布包

你会发现，`link`的包并不会在`package.json`的`dependencies`里出现。

```bash
cd ../my-test-project
npm link my-test-component-zhl
```

结果预览：

[![npm-link-p2](/images/tools/npm-link/p2.png)](/images/tools/npm-link/p2.png)
[![npm-link-p3](/images/tools/npm-link/p3.png)](/images/tools/npm-link/p3.png)

实际使用调试：

```javascript
require('my-test-component-zhl');
```

[![npm-link-p4](/images/tools/npm-link/p4.png)](/images/tools/npm-link/p4.png)

#### 调试

修改`my-test-component-zhl`的内容，会同步更新到当前引用的项目中...很方便。

效果：

[![npm-link-p5](/images/tools/npm-link/p5.png)](/images/tools/npm-link/p5.png)

#### 解除`link`

```bash
npm unlink --no-save packagename && npm install
```

查阅了文档，发现`unlink`其实是`uninstall`的别名。[-> 戳这里](https://docs.npmjs.com/cli/v7/commands/npm-uninstall)

当然，如果你不需要`link`该组件包进行测试时，建议你同时也卸载掉`link`到全局的包。如下：

```bash
cd ../my-test-component-zhl
npm unlink [-g]
```

### 说说`npm link`的缺点

好用确实是好用，但是也有几个缺点（槽点）：

<details>
<summary>建立了 link 过后，跨 node 版本使用容易出错</summary>

其实这个在我本机上没有遇到，前面测试的时候，细心的伙伴可能也发现了，`link`的`global`地址并没有带`node`版本信息（并不是在`nvm/nodeversion/node_modules`下，意味着我换一个`node`的版本，软链接仍然是存在且有效的。😮‍💨😮‍💨😮‍💨 如何做到的？~~小伙伴可以自己验证一下是否可行~~

配置`npm`的全局安装位置：

> npm config set prefix xxx

</details>

<details>
<summary>link 失败不会报错并且会回退到直接从 npm 仓库进行安装</summary>

这个确实，如果`link`本地预发布包失败，`npm`会全局安装一个你`link 的 packagename`包，然后再建立软链接，如果`npm-registry`仓库也没有这个包，才会抛异常。💀💀💀 ~~潜在问题，不容易发现，当然你可以通过为自己模块添加私有前缀避免这一问题~~

</details>

<details>
<summary>会有预期之外的二进制可执行文件安装</summary>

`通过 npm uninstall -g packagename`可以同时卸载全局包和它的二进制执行文件。那么根据`unlink`是`uninstall`的别名，可以很容易推出另外一个等价指令：

> npm unlink \[-g\]

</details>

<details>
<summary>不符合预期的软链接删除</summary>

每一次的`npm link`，都是一次**重新建立软链接**的过程，这个过程会取消之前已经链接的包。

如果你想同时保留多个包的软链接，记得同时`link`多个：

> npm link ../packageA ../packageB

</details>

这么一分析，好像也就不是问题了...😂😂😂 不过还是介绍一下另外的两种方式：

### `npm install`替代

使用安装指令，拼上`你自定义的模块的路径地址，可以是相对路径`，这种方式同样是建立软链接，并不是真的将资源下载到`node_modules`，只不过少了个`global`的中间过程。貌似更加方便一点。

> npm install \<package-path\>

如果你不想写入`package.json`，可以带上`--no-save`参数：

> npm install --no-save \<package-path\>

取消的话，可以用`uninstall`：

> npm uninstall package-path

### 扩展【仅做了解】

通过`link`工具避免上面提到的问题。[-> 详见](https://github.com/privatenumber/link)

```bash
npm i -g link
# 方式一
npx link package-path
# 方式二：项目根目录添加 link.config.json 配置文件，再执行
npx link
```
