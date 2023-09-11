---
title: 突发 node 环境以及相关指令异常
date: 2023-09-11 14:36:48
updated: 2023-09-11 14:36:48
tags:
  - bug
  - nvm
  - nvm_symlink
categories:
  - 踩坑记录
  - NVM
---

嘿嘿~，就是这么神奇，早上来公司写代码，由于周五走的时候并没有关掉服务，所以来了我就像往常一样工作。新的一周开始了...美滋滋 😚😚😚

一些不可描述的原因，编辑器很卡，~~公司真的该换得电脑了 😶😶😶~~ 我重启了 `vscode`，执行 `nr start` 想着先把程序跑起来。

`md`，`command not found? wtf?` 我不可能没装这个工具的呀，难道是我切了其他版本自己忘记了？，试着运行了一下 `node -v`

😋😋😋 什么是 `tm` 的惊喜...`node command not found` ~~大哥 不要搞我啊...~~

<!-- more -->

### 分析问题

既然现在不能用了，也不能怨天尤人了，先看看是不是 `nvm` 的问题，执行 `nvm ls` 看看...试着切换一下版本，或者重装一下`node`，如图：

[![nvm-symlink-not-working-p1](/images/bugs/nvm-symlink-not-working/p1.png)](/images/bugs/nvm-symlink-not-working/p1.png)

果然不出我所料 😎😎😎，不行...😭😭😭

难道是...全局 `nodejs` 可执行文件出问题不可用了？去瞅了一眼（**nvm 创建的软连接，将版本与全局可执行 node 相关联，也是环境变量里配置的 NVM_SYMLINK 的地址**）...就剩个空文件夹了 ~~连最基本的 node.exe 都没有，这还咋运行？废了~~

毕竟是通过 `nvm` 安装的 `node` 环境，也就不存在单独的 `nodejs` 环境变量配置了...

**`NVM_SYMLINK` 地址很重要，它指定了当前全局的 `node` 可执行环境**，至于为什么这个文件夹会突然坏掉不可用了，`emm...` 我也很疑惑...只能先把这个问题解决了

### 解决方法

- 删掉之前的 `symlink` 的文件夹，并在 `nvm` 下新建一个 `nodejs` 空文件夹

- 重新建立软连接（额...说的不太准确，应该是修改环境变量 `NVM_SYMLINK`，值就是上一步新创建的 `nodejs` 文件夹路径），如下图：

[![nvm-symlink-not-working-p2](/images/bugs/nvm-symlink-not-working/p2.png)](/images/bugs/nvm-symlink-not-working/p2.png)

- 设置完成后，关闭所有终端，重新打开`cmd or bash...`

- `ok`，重新绑定了，但是 `nodejs` 文件夹还是空的呀，此时执行 `node -v` 肯定还是 `command not found`

- 重新切入你使用的 `node` 版本，执行 `nvm use [version]`,（如果有需要可新安装指定的版本），如下图：

[![nvm-symlink-not-working-p3](/images/bugs/nvm-symlink-not-working/p3.png)](/images/bugs/nvm-symlink-not-working/p3.png)

- 可以看到，此时 `node` 和 `npm` 相关指令都能重新正常使用了 ~~欣慰啊 😯😯😯~~ （**有可能之前 `-g` 装的 `global packages` 会丢失，自行判断是否需要再次安装**）

- 最后一步，验证之前的猜想...找到第一步创建的 `nodejs` 文件夹并打开，可以发现里面多了很多内容。（其实也就是你 `nvm use [version]` 对应的 `version` 文件夹里的内容，也从侧面说明了 `symlink` 能够正常工作了）

[![nvm-symlink-not-working-p4](/images/bugs/nvm-symlink-not-working/p4.png)](/images/bugs/nvm-symlink-not-working/p4.png)

> 总会有一些奇奇怪怪的 `bug`伴随你我，日记一笔，以备后用.
