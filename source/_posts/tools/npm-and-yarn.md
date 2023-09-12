---
title: npm 以及 yarn 的指令使用
date: 2022-07-26 18:26:11
updated: 2022-07-26 18:26:11
tags:
  - npm
  - npx
  - yarn
categories:
  - 工具学习
  - Npm/Yarn
---

> ### `npm，npx`概念及指令简单介绍

`tips：相关配置指令在环境配置章节`

`npm`是随`node`一起安装的包管理器。主要作用是用于发布和下载程序包的`CLI(命令行)`工具，以及托管`javascript程序包的在线存储库`。

不同于`npm`，`npx`的`x`可理解为`eXecute`，主要作为命令行的寻址等辅助功能。`npx xxx`时，`npx`会先看`xxx`在`$PATH`是否存在，如果没有，则会找当前目录的`node_modules`，如果还是没有，那么会先下载安装这个`xxx`再执行。

<!-- more -->

#### 常用安装指令说明

安装使用 i or install。`卸载时使用 uni or uninstall，就不另外说明了`。

```bash
# 安装 npm 包 i 为 install 简写
# 不带任何参数：临时安装到 node_modules，不会添加到 devDependencies 和 dependencies
npm i [package]

# 安装指定版本or tag的包
npm install package@version
npm i package@tag

# 安装最新版本的包
npm i package@latest

# 安装时的一些参数说明：
# --save-dev or -D or --dev or -S-D 表示安装依赖到 devDependencies
npm i package -D

# -S or --save 表示安装依赖到 dependencies
npm i package --save

# 全局安装
npm i --global package
```

#### 其他指令及附带配置说明：

> npm help 查看全部指令

```bash
# 清除缓存
npm cache clean --force

# config 别名 c
# 设置全局安装位置
npm config set prefix 'C:\Users\xx\AppData\Roaming\npm'

# 设置缓存路径，默认在 c 盘
npm config set cache "C:\Users\xx\AppData\Roaming\npm_cache"

# 其他 npm 配置，格式均同上，set 设置，get 读取，delete删除，例如：
npm config get registry
# 当然可以添加限制参数，表示是本地配置、全局配置、用户配置
npm config --global set registry xxx

# 查看配置
npm config ls -l

# 版本号 -v or --version
npm -v

# 项目初始化 -y 参数可选
npm init -y

# 执行某个某个脚本命令 例如(package.json 的 scripts)
npm run [name]

# 登录到 npm，可以添加登录参数,如下登录到xxx npm仓库，一般用于公司内部npm私服
npm login
npm login --registry=xxx
# 没有账号，添加账号登录。同理可设置 registry
npm adduser

# 查看当前登录信息
npm whoami

# 查看一个 npm 包的最新版本/所有版本
npm view package version/versions
```

补充： `npm link`详见：[手把手教你使用 npm-link 软链](/share/npm-link)，`npm publish`详见：[如何发布一个 npm-package?](/share/npm-publish)

其他指令：[-> 详见 v7](https://docs.npmjs.com/cli/v7/commands/npm-install)

> tips: npm config 读取的内容是有优先级的，项目下的 `.npmrc` -> 用户级`~/.npmrc` -> 全局`安装目录/etc/.npmrc`（稍不注意就会存在你配置的 global registry 不生效...）

> ### `yarn`常用指令介绍

> yarn help 查看全部指令

大部分指令与`npm`大同小异，主要介绍一些项目常用的：

```bash
#初始化项目 (避免使用Git bash)
yarn init

# 安装依赖
# add 参数说明 --dev or -D 安装到 devDependencies，不加参数则安装到 dependencies
yarn add [package]
yarn add [package]@[version]
yarn add [package]@[tag]
yarn add [package] --dev

# 升级依赖的包版本
yarn upgrade [package]
yarn upgrade [package]@[version]
yarn upgrade [package]@[tag]

# 安装指定包的最新版本
# eg. yarn add ioredis@latest
yarn add [packageName]@latest

# 全局安装
yarn global add xx

# 移除依赖
yarn remove [package]

# 执行某个脚本命令 run 可以省略
yarn run [scriptName]
```
