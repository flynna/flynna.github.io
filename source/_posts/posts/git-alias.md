---
title: git常用alias别名配置
date: 2022-08-18 12:48:13
updated: 2022-08-18 12:48:13
tags:
  - git-alias
categories:
  - 土豆の收藏安利
---

### 点亮你的技巧点

通过更加简洁易记的单词或者短语来标记常用的指令--别名`alias`。它的另外一个不容忽视的好处就是可以尽可能的避免指令输出错误~~

<!-- more -->

### 配置

`eg.`配置`git status`的`status`别名为`st`：

> git config --global alias.st status

添加`--global`针对用户级起作用，如果不加，则只会在当前仓库起作用

除了通过指令进行设置以外，你可以打开`~/.globalConfig`文件，然后编辑录入你想要的`alias`...语法如下：

```
[alias]
  st = status
  xx = xxxx
```

这样就配置好了，例如上面配置的`status`，当你从`terminal`录入`git s`的指令时，会自动使用`git status`来替换当前指令。

有的童鞋要说了，那都这样了 我`git`也想简写，行否？当然是可以滴~~😋😋😋

例如我平时提交代码的`terminal`是`git bash`，我们可以为`git bash`添加`alias`的配置项，辅助识别指令：找到`~/.bash_profile`或者是`prefix/etc/bash.bashrc`，末尾追加：

> `alias g='git'`

现在`g st`就等价`git status`...有木有感觉很爽？既然知道了怎么配，大家根据自己的日常习惯和需要，进行配置即可...

写下我常用的一些`alias`配置，其中有一部分也是官方推荐的配置项： -> [这里](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-Git-%E5%88%AB%E5%90%8D)

| 别名`alias` | 原指                                                                                                                                  |
| :---------: | :------------------------------------------------------------------------------------------------------------------------------------ |
|     `a`     | `add .`                                                                                                                               |
|    `cm`     | `commit -m`                                                                                                                           |
|    `st`     | `status`                                                                                                                              |
|    `co`     | `checkout`                                                                                                                            |
|    `ci`     | `commit`                                                                                                                              |
|    `rci`    | `commit --amend --no-edit`                                                                                                            |
|    `br`     | `branch`                                                                                                                              |
|  `unstage`  | `reset HEAD`                                                                                                                          |
|   `last`    | ` log -1`                                                                                                                             |
|    `lg`     | `log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit` |
|    `pr`     | `pull --rebase`                                                                                                                       |
|    `pl`     | `pull`                                                                                                                                |

例如：`g last` 可以查看最近一条的`git`提交记录，`g rci`可以将暂存区内的文件改动合并至上一次提交，且不生成新的提交记录 ~~有时候修改了内容提交了才发现，可能忘记 format ？ 或者是变量名拼写错误？等等需要重新编辑提交的，但又不想在日志中再生成一条提交记录，就可以使用 rci 并入提交~~，`g unstage`可以取消`add`到暂存区的文件，`lg`可以查看`git`提交日志。
