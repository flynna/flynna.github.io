---
title: 在 VScode 中调试 NodeJS 程序
date: 2023-07-19 16:15:53
updated: 2023-07-19 16:15:53
tags:
  - debug
categories:
  - 技术分享
  - 程序调试
---

`代码调试在开发工程中必不可少`.

通常情况下我们会通过 `Chrome` 开发者工具在浏览器里进行调试，或者在代码里写入 `console` 语句打印输出观察变更。除此之外呢，就是在开发工具中对源代码断点调试。

本篇文章就以 `nodejs` 为例，介绍在 `vscode` 中调试代码的操作方法，具体参数配置可参考 [官方文档](https://code.visualstudio.com/docs/editor/variables-reference)

<!-- more -->

### 调试的作用

- 程序出现 `bug`，找出现 `bug` 的原因

- 学习源代码，理清代码的逻辑

### 调试需要满足的条件？

任何程序语言，在 `vscode` 中调试，大多需要满足下面三个条件：

- 搭建相应的编程语言的开发环境。例如 `node` 环境搭建可参考：[Nvm 安装及 Node 环境搭建](/envConstruct/nvm-and-node-install)

- 安装相应编程语言用于调试的扩展。（`vscode`内置了很多程序语言的 `debugger` 插件，例如 `JavaScript Debugger`）

- 配置 `launch.json` 程序

由于 `vscode` 对 `node` 的支持特别友好，所以无需我们单独去安装扩展和配置 `launch.json`

### 创建程序并开始调试-`Demo`

在此之前，请先确保 `node` 环境正常.

#### 创建一个程序

```bash
# 创建调试程序项目文件夹
mkdir debug-demo
cd debug-demo

# 创建并写入程序入口文件和代码
vi test.js
# 写入程序代码....这里省略代码部分

# 用 vscode 打开项目，并测试是否正常运行
code .
node test.js
```

#### 添加断点

断点位置在**代码行号的左侧，鼠标覆盖上去有个红色的点**，点击，红色点常亮表示断点添加成功

#### 进入`Debug` 并执行 `Run`

如下图所示，选择调试器 `Nodejs` 开始调试，或者键盘键入 `F5` 启动。这种方式是最简单的一种，使用了 `vscode` 内置的调试配置，不会生成 `launch.json`.

[![vscode-debug-p1](/images/share/vscode-debug/p1.png)](/images/share/vscode-debug/p1.png)

#### 调试面板功能介绍和使用

启动调试后，面板会包含：代码区、调试工具栏、调试输出面板、左侧的调试工具栏等等，具体功能介绍如下图所示：

[![vscode-debug-p2](/images/share/vscode-debug/p2.png)](/images/share/vscode-debug/p2.png)

### 自定义调试 `launch.json` 配置介绍

#### 组成部分

- 指定语言类型

- 指定调试类型 `launch` or `attach`

- 远程调试...

#### 创建 `launch.json`

可以手动点击 `vscode` 左侧的调试按钮后，再点击左侧工具栏出现的 `创建 launch.json 文件`，选择 `Nodejs` 调试器后，会在项目根目录生成 `.vscode/launch.json` 文件，如图所示

> `.vscode` 是 `vscode` 的配置文件夹

[![vscode-debug-p3](/images/share/vscode-debug/p3.png)](/images/share/vscode-debug/p3.png)

右下角的 `添加配置` 按钮，可以很方便的为我们自动化添加各种调试预配置。

#### `configurations` 配置

我们需要重点关注 `configurations` 字段，`version` 可以忽略.

`configurations` 是一个数组对象，因为一个项目可能存在多种调试模式，比如调试开发时调试，也可以是远程调试.又或者 `launch 和 attach 两种模式调试`

每个调试配置项是一个对象，包含了基础的 `type(指定编译环境 node、go、php...)、request(调试模式：launch、attach)、name(自定义配置项名称)`.和 [其他配置项](https://code.visualstudio.com/docs/editor/variables-reference)

##### 调试模式选择

**根据不同调试模式选择其他的配置项**内容，注意两种调试模式的区别：

> **`launch` 是启动程序并添加调试器，`attach` 是在已经运行中的程序添加调试器**。

通俗的讲，`launch` 是先将程序重新启动，然后给程序搭配调试器并开始调试，意味着不需要再手动执行 `npm run start` 了，如果此前已经启动了，可能还会存在端口冲突... 另外：只执行 `npm run start` 是不会停在断点位置的.

`attach` 是在已经运行的程序基础上追加调试器，这种模式需要我们手动先 `npm run start` 把程序跑起来，然后再 `F5` 启动调试。所以后者这种模式通常也用于进行远程调试

##### 配置项示例

需要注意的是，使用 `attach` 调试模式，需要 `Run` 启动程序时添加一个 `debug` 的监听端口(需要将 `9229` 端口对外开放)，`package.json` 如下：

```json
"scripts": {
  "start": "nest start",
  "start:dev": "nest start --watch",
  // 默认的 debug ip 地址为 127.0.0.1，即本机ip。 但是在 docker 容器中，本机 ip 只对自己生效。所以在启动 debug 的时候，需要将地址指定为：0.0.0.0
  "start:debug": "nest start --debug 0.0.0.0:9229 --watch",
}
```

项目中具体配置要根据实际需求进行调整，我只是简单列举一些常用配置：更多请[参考官网](https://code.visualstudio.com/docs/editor/variables-reference)

> `start:debug` 指令添加 `9229` 端口，映射到下面第一个调试配置：

> `0.0.0.0` 是路由表的默认路由，表示整个网络，即网络内的所有主机

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "attach",
      "name": "Attach attach localhost",
      "port": 9229,
      "skipFiles": ["<node_internals>/**"],
      "timeout": 10000,
      "restart": true
    },
    {
      "address": "172.16.148.220",
      "localRoot": "${workspaceFolder}",
      "name": "Attach to Remote - 172.16.148.220",
      "port": 30229, // 9229 映射到主机服务对外发现的端口
      "remoteRoot": "/usr/src/app", // 是 docker 容器里面代码存放的位置
      "request": "attach",
      "skipFiles": ["<node_internals>/**"],
      "type": "node",
      "restart": true
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Nest Framework",
      "args": ["${workspaceFolder}/src/main.ts"],
      "runtimeArgs": ["--nolazy", "-r", "ts-node/register/transpile-only"],
      "sourceMaps": true,
      "cwd": "${workspaceRoot}",
      "outputCapture": "std",
      "stopOnEntry": false
    }
  ]
}
```

当我们配置了类似如上多个调试项后，点击 `vscode` 左侧调试图标，可以发现启动运行和调试的按钮旁边多了个下拉选项(展示的调试配置项名称)，我们可以选择其中配置的某一个调试配置并启动调试.
