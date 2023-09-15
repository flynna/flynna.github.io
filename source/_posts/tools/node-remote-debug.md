---
title: 远程调试 NodeJS 程序的几种姿势
date: 2023-07-20 15:51:31
updated: 2023-07-20 15:51:31
tags:
  - debug
  - 远程调试
categories:
  - 工具学习
  - 程序调试
---

### `Before?`

在学习这篇文章之前，请先了解如何在 `vscode` 中调试一个 `Node` 程序，学会基础的调试参数配置和调试技能。详细请参考：[在 VScode 中调试 NodeJS 程序](/share/vscode-debug)

在此基础上，学习如何在 `vscode` 本地调试远程运行中的 `Node` 程序。

<!-- more -->

### 基于 `vscode` 本地进行远程调试

#### 添加 `debug` 脚本命令

在 `package.json` 添加调试端口和脚本命令，如下，添加 `9229 debug` 监听端口：

```json
{
  "scripts": {
    "start:debug": "nest start --debug 0.0.0.0:9229 --watch"
  }
}
```

当程序打包成镜像后，通常会写入启动脚本，在启动脚本中执行`npm run start:prod`。

那么我们需要在这个基础上，通过标识环境变量，区分是否执行`npm run start:debug`

例如：`start.sh` 中，通过 `RUN_ENV = TEST` 启动 `debug 命令`

```sh
#!/bin/sh
if [ "$RUN_ENV" != "" ] && [ $(echo $RUN_ENV | tr [a-z] [A-Z]) == "TEST" ];then
    yarn --production=false
    npm run start:debug
else
    npm run start:prod
fi
```

`ok`，本地项目配置完成...下面看 `rancher` 中又该如何配置？

#### `Rancher` 配置变更

如图：进入工作负载，先添加环境变量 `RUN_ENV`，然后关闭健康检查，方便重新部署时执行上面配置的 `npm run start:debug`.

> 关闭健康检查：在调试模式下，服务可能会进行一些非标准的操作或处于不稳定状态，这可能导致健康检查失败

[![node-remote-debug-p1](/images/share/node-remote-debug/p1.png)](/images/share/node-remote-debug/p1.png)

然后打开服务发现，在高级选项中配置端口映射规则：注意服务端口和主机端口的区别：

> 服务端口：指在 `Rancher` 中定义的服务对外暴露的端口，主机端口：指在运行服务的主机上实际使用的端口

> 当容器内的服务监听特定的端口时，主机端口可以将外部请求映射到容器内的服务

[![node-remote-debug-p2](/images/share/node-remote-debug/p2.png)](/images/share/node-remote-debug/p2.png)

然后重启容器服务，接着处理本地 `vscode` 配置：

#### 配置 `launch.json`

在`.vscode/launch.json`中添加调试配置项，如下所示：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "address": "172.16.148.25",
      "localRoot": "${workspaceFolder}",
      "name": "Attach to Remote - 172.16.148.25",
      "port": 32145,
      "remoteRoot": "/usr/src/app",
      "request": "attach",
      "skipFiles": ["<node_internals>/**"],
      "type": "node",
      "restart": true
    }
  ]
}
```

注意配置的`address、port、request`字段，因为是对远程已经运行中的程序追加调试，所以使用`attach`，`port`对应上面配置服务发现中的主机`port`.

配置好后，记得先执行`npm run build`，`why?`

> 构建生成的 `map` 文件包含代码映射信息的文件，用于帮助调试器在构建后的代码和源代码之间建立映射关系。

最后在本地源代码添加断点(断言、表达式、日志...)等等，启动和运行调试.

### 基于浏览器进行远程调试

- 点击`F12`或者鼠标右键打开控制台，控制台右侧有个设置按钮，点击选择源代码，启用`javascript`源映射

- 访问：`edge://inspect`，`edge://`根据你使用的浏览器决定

- 点击 `Open dedicated DevTools for Node`

[![node-remote-debug-p3](/images/share/node-remote-debug/p3.png)](/images/share/node-remote-debug/p3.png)

- 添加连接

[![node-remote-debug-p4](/images/share/node-remote-debug/p4.png)](/images/share/node-remote-debug/p4.png)

- 添加断点

[![node-remote-debug-p5](/images/share/node-remote-debug/p5.png)](/images/share/node-remote-debug/p5.png)

当用户下次访问该服务时，效果如下：

[![node-remote-debug-p6](/images/share/node-remote-debug/p6.png)](/images/share/node-remote-debug/p6.png)

这种方式虽然也能实现远程调试的效果，但是它是基于`build`后的产物调试的，不方便阅读和定位.
