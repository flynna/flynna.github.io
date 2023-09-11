---
title: Git 环境搭建
date: 2022-07-16 19:12:54
updated: 2022-07-16 19:12:54
tags:
  - git
  - git-bash
categories:
  - 环境搭建
  - Git
---

> `Git` 是一个分布式版本控制系统，用于跟踪文件和代码的变化，协作开发以及管理项目的版本。**是支持版本管理、多人协作、分支管理、轻量级、开源，并拥有丰富的生态系统**

<!-- more -->

### 工具下载

`傻瓜式安装，选择盘符- 注意安装过程中会提示是否将bash加入右键菜单，勾选`

[windows 下载](https://git-scm.com/download/win)

[mac 下载](https://git-scm.com/download/mac)

### 全局配置

#### 配置当前`git`用户信息

```bash
git config --global user.name [git提交显示的用户名]
git config --global user.email [git提交用户所属邮箱]

# 查看系统配置
git config --system --list || git config --system -l

# 查看用户全局配置
git config --global --list || git config --global -l

# 查看当前仓库配置
git config --local --list || git config --local -l
```

#### 配置`git`规范

<div class="primary">

> 换行符约束（问题记录）

</div>

多人协作项目，如果其他人用的时`linux`，你用的`windows`，会存在行尾结束符问题扰乱协作，因为`windows`使用回车和换行两个字符来结束一行。

配置`core.autocrlf`可以避免这个问题。

参数说明：

> true：默认值，拉取（签出）代码的时候，LF 会被转换成 CRLF。大家也应该发现了，如果源代码中是 LF，那么提交的时候肯定会遇到错误冲突 LF would be replaced by CRLF.

> input：提交（签入）时把 CRLF 转换成 LF。同理，当远程仓库源代码使用的是 CRLF，那么提交也会遇到错误冲突 CRLF would be replaced by LF.

> false：取消此功能，即签入签出代码时，不做任何处理。源代码是 LF，那么提交就是 LF，源代码是 CRLF，那么提交也就是 CRLF。所以，当遇到上面两种错误冲突时，设置为 false 通常可以避免。

如果项目中没有对源文件的换行符作出规定，**源代码使用的是 LF，设置`autocrlf=input`, 源代码使用的是 CRLF，设置`autocrlf=true`**。

当我用`windows`电脑`git clone`代码的时候，若我的`autocrlf`(在`windows`下安装`git`，该选项默认为`true`)为`true`，那么文件每行会被自动转成以`CRLF`结尾，若对文件不做任何修改，`pre-commit`执行`eslint`的时候就会提示你删除`CR`。

项目仓库中一般是`Linux`环境下提交的代码，文件换行符默认是以`LF`结尾的(工程化需要，统一标准)，如果你是`windows`用户，设置为`false`最稳妥。~~毕竟不可能每个项目都去配一次，而不同的项目完全有可能是不同的环境和换行符，设置为 false 可以避免这个问题~~

```bash
git config --global core.autocrlf false
```

#### 配置密钥对

生成公钥和私钥，用于上传代码时的安全验证

```
在git bash里执行命令ssh-keygen 一路回车，就可以生成密钥对，默认密钥对是存放在(/c/Users/[主机用户名]/.ssh/) 。
这个目录下有两个文件， .pub就是公钥，另外一个是私钥到线上（gitlab或其他平台）
打开设置->安全设置->ssh公钥，把本地的公钥文件全选复制进来，输入登录密码，就配置成功了。
```

#### 配置`https.proxy`代理

```bash
git config --global https.proxy http:\127.0.0.1:xxx
```
