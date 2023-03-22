---
title: github 上如何为开源项目提交 pr?
date: 2022-07-24 18:41:36
updated: 2022-07-24 18:41:36
tags:
  - github
categories:
  - 收藏安利
  - Github
---

本文主要是根据自己在摸索提交`pr (pull-request)`过程程的一些总结。

为了方便描述，约定本文需要`fork`的项目为`test-project`，源仓库地址`regionUser/test-project`，你（提`pr`的用户）的用户名为`userA`。

<!-- more -->

### `fork`原项目

登入自己的`github`账号，进入你需要提`pr`的那个项目下，点击左上角的`fork`。

然后进入你的`Repositories`列表，你会发现名为`test-project`仓库。

### `clone 这个 fork`项目

打开`userA/test-project`（即你`fork`的项目下），`clone`到本地。

> git clone https://github.com/userA/test-project.git

### 建立`upstream`上游链接

这里上游指的是一开始`fork`的那个项目源。

> git remote add upstream https://github.com/regionUser/test-project.git

查看远程仓库信息：

> git remote -v

<div class="error">

> 一定确定`origin`是你自己的地址，`upstream`是远程的地址。

</div>

### 新增、提交变动

这里修改的是你`clone`下来的项目，列举了两种方式。

#### 在`main`分支修改

我这里为了测试，选择就在`main`上修改，且模拟了源项目`main`变更，以及`userA`的多次提交。

```bash
git add .
git commit -m 'userA的第一次提交'
git commit -m 'userA的第二次提交'
git push origin main
```

#### 基于新的分支

创建了一个新的`fix/fs-fix`

```bash
git checkout -b fix/fs-fix
git add .
git commit -m 'userA的第一次提交'
git add .
git commit -m 'userA的第二次提交'
git push origin fix/fs-fix
```

写完后合并到主分支`main`

```bash
git merge fix/fs-fix
git push origin main
```

### 新建`pull-request`

在`github`打开你`fork`的项目。`userA/test-project`，点击`Pull requests` -> `New pul request`新建`pr`，会自动跳转至`regionUser/test-project`下的`compare`，出现下面界面：

[![github-create-pr-p1](/images/posts/github-create-pr/p1.png)](/images/posts/github-create-pr/p1.png)

> `base repositories`源项目仓库，`head repositories`是`fork`的项目仓库(`userA`).

我这里创建的时候出现`check`不通过的提示（如果你没有遇到，就跳过）：

> Can’t automatically merge. Don’t worry, you can still create the pull request.

可以预料到，因为原项目的作者在我们之前`fork`以后，又新增了功能提交。 ~~虽然仍然可以创建成功，但不建议，也不一定会被采纳~~

此时，需要我们对`fork`的项目源码做代码同步。如下：

```bash
# 拉取 upstream 最新代码
git fetch upstream

# (这里不一定是使用 rebase，merge 同样可以，只是一个同步最新代码的一个方式而已)
# 再 rebase，main 是分支名，我这偷了个懒
# 此过程可能会遇到冲突，解决冲突
git rebase upstream/main

# 当然直接使用 git pull upstream branch 也是可以的 === fetch + merge

# 此过程可能会遇到冲突，解决冲突再次 rebase
git status
git add .
git rebase --continue

# rebase （变基）是合并的另外一种方式，不同于merge的是，不会产生旁支和冗余的提交记录。
# 执行后会在 terminal 中打开编辑器交互，完成变基操作（不知道怎么在这个界面操作的，可以看我的另外一篇文章'vi 编辑器学习'）
# 在界面上方列出了需要编辑的所有提交，在每个commit id前的是指令类型(pick)，在Commands中有相关的指令说明。
# p, pick = 保留该commit
# r, reword = 保留该commit，但修改它的提交信息
# e, edit = 保留该commit，在合并该请求时暂停
# s, squash = 保留该commit，合并到前一个提交中
# f, fixup = 类似于squash，但抛弃提交它的提交信息
# x, exec = 执行shell命令
# d, drop = 丢弃该commit

# 解决掉所有冲突 推送此次合并，添加 -f 强制推送
git push -f origin main
```

按上面流程操作完了以后，重新创建`pull-request`，`check`错误已经消失：

[![github-create-pr-p2](/images/posts/github-create-pr/p2.png)](/images/posts/github-create-pr/p2.png)

`填写pull-request 的 title，你也可以添加 comment`，完成后，你可以在`regionUser/test-project`的`Pull requests`下看到你新提交的`pr`.效果如下：

[![github-create-pr-p3](/images/posts/github-create-pr/p3.png)](/images/posts/github-create-pr/p3.png)

至此，`pr`创建就完成了，等待原项目作者审核后合并。

### 原仓库作者视角

[![github-create-pr-p4](/images/posts/github-create-pr/p4.png)](/images/posts/github-create-pr/p4.png)

选择合并的方式，再次点击确认即可完成合并。

### 更新`fork`项目

当你的提交被采纳以后，原项目`regionUser/test-project`会新增一个合并的提交记录，为了保持一致，需要在`fork`的本地仓库进行更新：

```bash
git pull --rebase upstream [branch]
# 或者
git fetch upstream
git merge upstream/main

# 更新推送至远程
git push origin main
```

### 其他问题

#### 如何再次提交`pr`?

`answer`：重复上面新建`pull-request`的流程，值得注意的是：记得先同步主分支代码，保证与`fork`的原项目代码一致，否则你会继续遇到冲突。

#### 提交`pr`后，我`fork`的仓库可以删除吗?

`answer`：在`pr`被采纳前，不能删除，否则该`pr`会自动关闭；在采纳后是可以删除掉的，因为代码已经被合并至主项目里了（存在于主仓库的`Pull requests`记录也会被删除）。

<div class="success">

> 世事千帆过，前方终会是温柔和月光。

</div>
