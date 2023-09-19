---
title: Vscode 配置 GitBash 作为默认的 Terminal
date: 2022-07-17 21:02:05
updated: 2022-07-17 21:02:05
tags:
  - vscode
  - terminal
categories:
  - 环境搭建
  - Vscode
---

将 `Git Bash`设置为 `VSCode` 的默认终端，能够为我们开发带来很多好处：

- 一致性: 如果你在命令行中已经熟悉使用 `Git Bash` 或类似的 `Unix-like` 命令行工具，将其设置为 `VSCode` 的默认终端可以保持工作环境的一致性，无需切换不同的终端。

- 强大的命令行支持: `Git Bash` 提供了强大的命令行支持，可以执行各种 `Git` 命令和其他命令行任务，无需离开 `VSCode` 界面。

- 集成 `Git:` 使用 `Git Bash` 作为默认终端可以确保与 `Git` 版本控制工作流的紧密集成。你可以轻松地提交、推送、拉取和解决冲突，同时观察 `Git` 操作的输出。

- 自定义性: `Git Bash` 和 `VSCode` 都支持丰富的自定义选项。你可以根据自己的偏好配置终端的外观和行为。

...

<!-- more -->

### `VsCode Terminal`配置为默认`git-bash`

- 找到`Terminal > Integrated > Profiles: Windows`，选择在`settings.json`中编辑。

- 添加`GItBash`子项（注意首字母大写，我配置的时候小写没有生效，不知道是不是版本问题导致的）修改`path`，如下图。

- 配置默认的`terminal`为`GitBash`, 方式一：设置中搜索`Default Profile: Windows`，选择`GitBash`，方式二：直接在`settings`中添加下面配置。

[!['vscode-terminal'](/images/envConstruct/vscode-terminal-config/p1.png)](/images/envConstruct/vscode-terminal-config/p1.png)

### 配置指令的`alias`

打开文件`bash.bashrc`，在安装目录下的`Git/etc/bash.bashrc`。

末尾添加：

```bash
alias blog='cd /e/flynn/flynna.github.io'
alias serve='yarn serve'
# ...
```
