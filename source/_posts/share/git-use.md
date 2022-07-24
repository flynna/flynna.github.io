---
title: git 的基本使用
date: 2022-07-24 13:44:42
updated: 2022-07-24 13:44:42
tags:
  - git
  - git指令
categories:
  - 土豆の教程分享
  - 工具使用
---

> Git is a free and open source distributed version control system

### `Git`是用来做什么的?

- 托管代码到远程，分布式托管，避免本机磁盘损坏造成不可挽回的局面。

- 版本控制，可以发布多个版本并且实现在各个版本之间来回穿梭（实现原理：文件快照，每个版本都会有一个文件快照，比直接备份文件快速便捷。因此，Git 仓库又被称为版本库）。

- 团队协作，强大的分支功能，可以快速实现团队协作

<!-- more -->

环境搭建：[-> 戳这里](/share/git-install-and-terminal-config)

---

### 指令使用

#### 配置信息

```bash
git config <scope> <key> <value>
```

#### 初始化项目仓库

区别于克隆，初始化仓库通常是在新建项目时使用。

```bash
# 第一步：初始化仓库
git init

# 第二步：建立本地仓库与远程仓库的关联
git remote add origin [线上仓库地址]
# 如果此命令抛出 fatal: remote origin already exists. 异常，那么继续执行：
git remote rm origin
git remote add origin [线上仓库的SSH地址]
```

`注意：.git 文件不能嵌套（仓库不能嵌套）`

#### 克隆一个项目

克隆一个已经存在的项目到本地。

```bash
git clone [url]
```

#### 一次提交流程

```bash
# 先拉取目标分支的最新代码，拉取之后直接合并。可以简写为 git pull
git pull origin [分支名]
# git pull = git fetch + git merge
# git fetch origin [分支名] ：先拉取目标分支的最新代码，拉取之后由用户决定是否合并

# 将改动的代码文件添加至暂存区
# 也可以 git add [文件名] 来单独添加某一个文件
git add -A

# 将暂存区的文件提交至本地仓库
git commit -m '这次提交的描述信息'

# 将代码推送到远程 如果不是该分支的第一次推送，可以简写为 git push
git push origin [分支名]

# 当然这个过程你也可以查看当前仓库的状态
git status
```

如图：

[![git-use-p1](/images/share/git-use/p1.png)](/images/share/git-use/p1.png)

#### 查看提交日志

```bash
git log
```

#### 存储文件改动

场景：当前工作区被污染，内容未完成，不想直接提交， 需要切到指定分支版本紧急解决其他需求。

```bash
# 存储当前工作区的改动（不存储新文件及 ignore 的文件
git stash

# 存储所有改动
git stash -a

# 指定版本 test 并存储
git stash save 'test'

# 查看当前存储的所有版本
git stash list

# 弹出最新的存储内容至工作区 并删除存储对应的版本的存储
git stash pop

# 应用指定版本的存储内容至工作区 不删除存储 version 对应存储的版本号 0、1、、
git stash apply stash@{version}

# 删除指定的存储版本内容 0 为版本号
git stash drop stash@{0}

# 删除所有的存储
git stash clear
# ...
```

#### 将某次提交合并至当前分支

场景：在某个功能分支`A`上新增了一个功能提交，想在另外一个分支`B`使用，但又不想让另外这个分支`B`合并`A`的其他功能，此时，可以通过`cherry-pick`将`A`的某次提交合并到`B`。

```bash
# 切换到分支 B
git checkout B

# 将分支 A 提交的 asgasgasgnajsgank 内容合并当当前的 B 分支
git cherry-pick asgasgasgnajsgank

# 遇到冲突
# 解决冲突 先打开编辑器解决冲突
# -- 工作区是干净的
git cherry-pick --continue
# -- 工作区不干净
git add xxx
git commit -m ''
git push 或 git push origin B

# 不解决冲突
# -- 退回到 cherry-pick 之前的状态
git cherry-pick --abort
# -- 退出 cherry-pick 但不回到之前的状态
git cherry-pick --quit
```

#### 回退版本

回退到指定版本后，指定版本的后续版本将被删除

```bash
# 查看版本号
git log

# 版本回退至 asgasjgajsga
git reset --hard asgasjgajsga

# 推送更改
# 直接 git push 会报错，因为此时本地的 HEAD 指向版本和远程不一样
git push origin A  || git push -f
```

#### 还原某一次提交

场景：用于回退之前的错误提交或者取消某一次的内容提交，不影响其他的提交内容

```bash
# 取消 asgasgasgjaj 这次提交
git revert -n asgasgasgjaj

# 遇到冲突先解决
# 提交这次 revert
git commit -m 'Revert: 取消 asgasgasgjaj 的提交内容'

# 推送
git push
```

#### **分支管理**

##### 分支查看

```bash
# 查看本地分支列表
git branch 或 git branch -a

# 查看所有分支
git branch -l
```

##### 分支创建

```bash
# 创建分支 A
git branch A

# 指定以 asgasgl 这次提交的内容 创建的分支名 A
git branch A asgasgl

# 创建并切换至分支 B
git checkout -b B
```

##### 分支切换

```bash
# 切换至分支 B
git checkout B
```

##### 分支更新

场景：远程分支被删除，但自己本地显示远程分支存在

```bash
# 更新远程分支
git remote update origin --prune

# 查看需要清理的本地分支
git remote prune origin --dry-run

# 删除本地存在，但远程不存在的分支
git remote prune origin -n
```

##### 分支删除

```bash
# 删除本地分支 B, 需要先切到一个非 B 的分支 A
git checkout A
git branch -d B

# 删除远程分支 B
git push origin --delete B
```

##### 分支合并

```bash
# 功能分支的代码推送至远端
git push origin feature/xxx

# 切换至主分支
git checkout develop

# 拉取主分支最新代码
git pull origin develop || git pull

# 合并功能分支至 develop
git merge feature/xxx

# 遇到冲突 --- 丢弃 不合并了
git merge --abort
```

#### 为某次提交打标签

标签分为**轻量标签（`lightweight`）**与附注标签 **（`annotated`）**

##### 标签查看

```bash
# 列出所有标签
git tag

# 按照通配符列出标签需要 -l 或 --list 选项（下面通配符指 v1.0.0 开头的所有标签）
git tag -l 'v1.0.0*'

# 查看具体某一个标签（v1.0.0）信息和与之对应的提交信息
git show v1.0.0
```

##### 标签创建

```bash
# -a 创建附注标签（v1.0.0） -m 指定了一条将会存储在标签中的信息
git tag -a v1.0.0 -m "my version 1.4"

# 创建轻量标签（v1.0.0），不包含任何其他信息
git tag v1.0.0

# 针对历史提交记录的某一次的提交，创建标签（v1.0.0）
git tag -a v1.0.0 9fceb02xxx
```

##### 标签共享（推送远程）

注意需要显式的推送，不能简写为`git push`.

```bash
# 将标签 v1.0.0 推送到远程，方便其他人共享
git push origin v1.0.0

# 将所有标签推送至远程
git push origin --tags
```

##### 标签删除

```bash
# 删除本地标签 v1.0.0
git tag -d v1.0.0

# 删除远程的一个标签 git push <remote> :refs/tags/<tagname>
git push origin :refs/tags/v1.0.0
# 或者
git push origin --delete v1.0.0
```

##### 检出标签

在标签创建完成后，你在上线之前又做过多次修改。这时你可以检出你之前那个准备发布的版本（打标签的版本）进行部署。

`Git`中不能真的检出一个标签，因为他们并不能像分支一样来回移动。如果你切换到某个标签后又提交了新东西，那么这些新的提交将不属于任何分支，并且无法访问，除非通过确切的提交哈希才能访问。

那么**如果真的需要在某一个标签版本修复一些内容：那么推荐的做法是基于这个标签版本创建一个新的分支。**如下：

```bash
# git checkout -b [分支名] [标签名]
git checkout -b version1 v1.0.0
```

<div class="success">

> 说了这么多指令，不用刻意去记，孰能生巧。另外：安利 vscode 的插件 Git Graph、GitLens — Git supercharged，所有的`git`指令都能通过界面操作。可以看一下我另外一篇文章： [vscode 的使用，设置分享以及插件推荐](/posts/vscode-setting-and-plugins)

</div>

### 代码提交规范

每一次提交代码时都会写`commit message`，如果书写风格不统一，十分不利于阅读和维护。推荐格式：

> type(scope) : subject

`type`：指`commit`的类别，此项必填。

- `feat`: 新功能

- `fix`: 修复 bug

- `docs`: 文档改变

- `style`: 代码格式改变

- `refactor`: 某个已有功能重构

- `perf`: 性能优化

- `test`: 增加测试

- `build`: 改变了 build 工具 如 grunt 换成了 npm

- `revert`: 撤销上一次的 commit

- `chore`: 构建过程或辅助工具的变动

`scope`：说明`commit`影响的范围，此项选题

`subject`：`commit`的简短描述，不超过`50`个字符

> tips：仅作为一个推荐的规范格式，并不会对你的 commit-message 做任何校验，自觉遵守即可；另外如果你的团队有自己的提交规范，请优先以团队提交规范为主。
