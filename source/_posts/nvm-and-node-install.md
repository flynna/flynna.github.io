---
title: Nvm安装及Node环境搭建
date: 2022-07-13 22:38:09
updated: 2022-07-13 22:38:09
tags:
  - nvm
  - node
  - nrm
  - npm
  - yarn
categories:
  - 土豆の教程安利
---

> ### `nvm`下载及安装

下载地址：[链接](https://github.com/coreybutler/nvm-windows/releases)

`下载exe的package文件，选择自定义的安装盘符，傻瓜式安装即可，环境变量会自动添加。`

安装完成后，配置`node`及`nvm`的`mirror`地址 (不配置会导致下载`node`后，在下载`npm`过程中可能会发生错误，需要手动下载安装`npm`)。配置如下：

```bash
# 方式一：手动添加到配置文件中：nvm安装目录/settings.txt 末尾追加
node_mirror: https://npm.taobao.org/mirrors/node/
npm_mirror: https://npm.taobao.org/mirrors/npm/

# 方式二：通过指令修改
nvm node_mirror [url]
nvm npm_mirror [url]
```

> ### 安装`node`以及`npm`

`打开windows terminal，并以管理员身份运行(避免权限问题导致安装过程抛出异常)`

```bash
# 显示所有可以下载的 NodeJs 版本：
nvm list available

# 安装指定版本的 NodeJs (选择版本号安装) 你可以安装多个版本
nvm install 14.5.0

# or 直接安装最新版(不建议)
nvm install latest

# 列出你以及安装的 NodeJs 版本，下面一行指令同 nvm list
nvm list installed

# 使用某个版本号的 NodeJs (从你安装的版本中选择)
nvm use 14.5.0
```

检测`node及npm`是否成功安装：出现版本号代表安装成功

```bash
node -v
npm -v
```

如果`nvm`没有配置`mirror`，或者其他原因导致你的`npm`不能正常下载安装。可以通过上面配置的方式，重新安装`node`，会自动匹配对应的`npm`下载安装。当然你想手动安装并配置`npm`，也是 ok 的，如下介绍手动配置方式：(正常安装请跳过)

- 根据`nvm`下载安装`node`时，抛出的异常信息得知`npm`的下载地址：[下载 npm](https://github.com/npm/cli/releases/tag/v6.14.11)

- 将`npm`解压到`nvm/v指定版本/node_modules/`下，将`npm`解压文件夹重命名为`npm`

- 将`npm/bin/`文件下的`npm`及`npx`相关文件拷贝到`nvm/v指定版本/`下

- 在当前目录打开终端，输入`npm -v`，出现版本号表示安装成功，如果换了文件夹后提示指令不存在，可在环境变量`path`内添加当前路径

> ### `nrm`的安装及切换镜像源

`nrm`提供了快速切换镜像源的方案，如果通过`npm`指令配置，则无需安装

```bash
# 全局安装
npm i(install) nrm -g

# 检测是否安装成功
nrm --version

# 列出可用的镜像源
nrm ls

# 测试镜像地址速度
nrm test

# 切换到指定镜像
nrm use [镜像名称]
```

> ### `npm`全局配置及`yarn`安装

- 安装`yarn`

```bash
# [下载镜像安装](https://classic.yarnpkg.com/en/docs/install#windows-stable)

# npm 安装
npm install --global yarn

# [2.x 安装使用](https://yarnpkg.com/getting-started/install)
```

- 配置`npm`及`yarn`的`registry`：

```bash
# 默认源
npm config set registry https://registry.npmjs.org
yarn config set registry https://registry.yarnpkg.com

# 配置到内网环境
npm config --global set registry http://172.xxx/repository/npm/

# 配置淘宝镜像
# (截止22-5-31)
npm config set registry https://registry.npm.taobao.org
yarn config set registry https://registry.npm.taobao.org
# new
npm config set registry https://registry.npmmirror.com

# 配置源，且同步安装使用 cnpm
npm install -g cnpm --registry=https://registry.npmmirror.com

# 如果命令行修改失败 - 手动修改配置 c:/用户/user/.npmrc 文件
registry=http://172.xxx/repository/npm/

# 其他镜像
CHROMEDRIVER_CDNURL="https://registry.npmmirror.com/-/binary/chromedriver"

SASS_BINARY_SITE="https://registry.npmmirror.com/-/binary/node-sass"

PUPPETEER_DOWNLOAD_HOST="https://registry.npmmirror.com/-/binary"

NODEJS_ORG_MIRROR="https://registry.npmmirror.com/-/binary/node"

NVM_NODEJS_ORG_MIRROR="https://registry.npmmirror.com/-/binary/node"
```

- 设置全局安装及缓存路径

`配置使用 set，查看使用 get。除了指令设置配置项外，也可手动添加配置，文件位置：~/.npmrc ~/.yarnrc`

```bash
# 清除缓存
npm cache clean --force
yarn cache clean

# 设置全局安装位置
npm config set prefix 'C:\Users\xx\AppData\Roaming\npm'
yarn config set global-folder "D:\yarn\yarnDate"

# 设置缓存路径，默认在 c 盘
npm config set cache "C:\Users\xx\AppData\Roaming\npm_cache"
yarn config set cache-folder e:\YarnCache

# 查看配置信息 --global or -l
npm config ls
yarn config list
```
