---
title: git 使用过程中的踩坑记录
date: 2022-07-31 16:32:50
updated: 2022-08-14 21:47:45
tags:
  - bug
  - git
categories:
  - 踩坑记录
  - Git
---

> 用于记录由于`git`的不规范使用`or`其他问题导致的**错误和奇怪现象**，及其实际解决方案汇总。

<!-- more -->

### 分支信息不同步

> 用户`A`删除了本地及远程的某分支`fz`后，用户`B`执行`git branch -a`依旧能够看到该分支.

尽管用户`B`执行`git pull`拉取并合并修改，用户`B`仍然能够看到被删除`origin/fz`分支.那么此时应该怎么解决？

```bash
# 执行下面命令 --- 删除远程不存在，但本地通过指令仍能看到的某 origin/xxx 分支
git remote prune origin

# 如果用户 B 显示本地也还存在 fz 分支，需要再删除本地的 fz（先切到另外一个非 fz 分支，当然不删除也不影响）
git branch -d fz
```

### 拒绝合并不相关的历史

`refusing to merge unrelated histories: `出现这个问题的最主要原因还是在于本地仓库和远程仓库实际上是独立的两个仓库。（例如你在本地初始化了一个仓库，并提交了一些东西，然后与远程仓库建立关联，此时推送就会抛这个异常）直接`clone`的方式在本地建立起远程`github`仓库的克隆本地仓库就不会有这个问题。解决办法：

```bash
git pull origin master --allow-unrelated-histories
```

### 新建分支首次提交错误

`There is no tracking information for the current branch`，此时需要在推送的时候建立关联，根据提示完成即可：

```bash
# (分支名称)，建立关联关系即可
git branch --set-upstream-to=origin/xxx
```

### 推送被拒绝

`Updates were rejected because a pushed branch tip is behind its remote.`推送的分支提示位于其远程提交的后面。比如别人提交了，你并没有拉取最新代码，直接`push`就会存在该提示。

> 建议：每次`push`之前最好先`pull`拉取并合并代码，如果有冲突就解决冲突，然后再执行`push`，可以避免很多问题。

> 解决办法：先`pull`拉取最新代码合并，然后再提交。**或者**: 添加 `-f` 参数强制推送。

### 文件名大小写不敏感

这个在另外一篇`git指令使用`中也提到过，`windows`环境下`git`对大小写识别不敏感，解决办法：

> 方案一：重命名为另外一个名字，然后提交（一定要提交），然后再改回你要修改的大、小写名称。（不仅麻烦，而且还会多产生一条无关的提交）

> 方案二：使用`git mv [file] [newfile]`指令修改。

### 行尾结束符统一

问题描述及解决方案详见[-> git 环境搭建](/envConstruct/git-install-and-terminal-config)

### HttpRequestException encountered

使用`Git`下载或者更新代码时出现`fatal：HttpRequestException encountered`提示信息，但是它又不会影响 Git 的正常使用

> 解决办法：更新`Windows`的`git`凭证管理器，打开链接，下载安装即可 [https://github.com/Microsoft/Git-Credential-Manager-for-Windows/releases/tag/v1.14.0](https://github.com/Microsoft/Git-Credential-Manager-for-Windows/releases/tag/v1.14.0)

### 持续更新中...
