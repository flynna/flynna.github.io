---
title: 在 Windows 中安装和使用 WSL
date: 2022-07-16 15:47:54
updated: 2022-07-16 15:47:54
tags:
  - wsl
  - ubuntu
  - terminal
categories:
  - 环境搭建
  - WSL
---

### `Overview`

`WSL（Windows Subsystem for Linux，全称 Microsoft-Windows-Subsystem-Linux）`是一种由微软开发的在 `Windows` 操作系统上运行 `Linux` 应用程序的兼容性层（模拟 `Linux` 运行环境 | 轻型虚拟机）。

`WSL` 支持多个 `Linux` 发行版，包括 `Ubuntu`、`Debian`、`Kali Linux`（操作系统）。

**注意：在 `WSL` 中，只有一个默认的 `Linux` 发行版版本，即 `Ubuntu`。其他的操作系统可以通过应用商店安装，`WSL` 提供了兼容性支持。**

<!-- more -->

### 安装 `WSL`

打开 `Windows PowerShell`，执行以下命令：

- 安装 `WSL2`(推荐)：

```css
/* 这个指令默认会设置 WSL 版本为 2 */
wsl --install
```

- 安装 `WSL1`(旧版`Windows`)：

```bash
# /online 指定在当前正在运行的操作系统中执行操作
# /enable-feature 启用一个或多个 Windows 功能
# /featurename:Microsoft-Windows-Subsystem-Linux 指定要启用的特定功能，即 WSL
# /all 启用所有依赖于指定功能的子功能
# /norestart 在完成操作后不要重新启动计算机

# 启用 Linux 子系统
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 启用虚拟机
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 下载 Linux 内核更新包 ---> 安装 ---> 设置 WSL 版本号
# 下载子操作系统 （可以在应用商店里下载安装）
# 设置账户密码
```

> `dism.exe` 是 `Windows` 操作系统中的一个命令行工具，全称为 `Deployment Image Servicing and Management (DISM)`。它用于管理和维护 `Windows` 映像文件（如 `Windows` 安装文件和映像文件）以及 `Windows` 组件、驱动程序和软件包。

### 切换 `WSL` 版本

#### 查看 `Linux` 分发版本

```bash
# 进入 wsl
wsl
# 查看
lsb_release -a
```

#### 查看已下载的版本

```bash
wsl -l -v
# eg. Ubuntu Running 2 安装的 Ubuntu 使用的是 WSL 2
```

#### 设置切换版本

```bash
# 默认使用 WSL2
wsl --set-default-version 2
# 切换版本 使用 Ubuntu-20.04 的 WSL 1 版本
wsl --set-version Ubuntu-20.04 1
```

### 安装 `WSL` 的操作系统 `Ubuntu`

- 应用商店安装`Ubuntu`

  下载安装后启动，设置用户名密码，设置完成后会启动 `Ubuntu`，然后执行 `Linux` 程序和命令

- 命令行安装

```bash
# or wsl --install -d Ubuntu-20.04 [这个过程可能需要管理员权限]
wsl --install -d Ubuntu
```

### 启动/退出 `WSL`

```bash
# 启动
wsl
# 退出 关闭终端 | 关机 直接退出 or (↓推荐 exit)
exit
# or 关闭所有正在运行的 WSL 会话 [在非 Ubuntu 环境下执行 wsl 命令, 比如新建窗口]
wsl --shutdown
```

> 首次启动会提示输入用户名和密码

#### 忘记账户密码

```bash
# 进入默认的 WSL 系统
wsl -u root
# 进入指定系统 <名称> ---> Ubuntu
wsl -d <名称> -u root
# 修改密码 <username> 对应忘记密码的账户名
passwd <username>
# 修改成功 退出
exit
```

### 包管理器`sudo`

`sudo` 是 `Linux` 和 `Unix` 操作系统中用于暂时获取管理员权限的命令。它允许用户使用根级别权限执行命令，是发行版的首选包管理器。

#### 更新软件包

> `sudo` 是在 `Unix、Linux` 和类似系统中用于以超级用户或管理员权限执行命令的指令

> 例如 `sudo -u john ls /home/john` 这会以 `john` 用户身份执行 `ls /home/john` 命令，而不是当前用户的身份。

建议**定期更新和升级包**。以管理员权限更新 `Ubuntu` 操作系统的软件包列表:

```bash
# 启动并进入 Ubuntu
wsl
# 更新、升级软件包
sudo apt update && sudo apt upgrade
```

#### 安装/删除软件包

```bash
# 以管理员身份安装软件包
sudo apt install [package-name]
# 删除包
sudo apt remove [package-name]
```

### 文件存储和内存占用

既然 `WSL` 提供了虚拟机环境，那么在 `Ubuntu` 安装的文件又去了哪里，内存又是怎么分配的？

> `Ubuntu` 文件大小可能会随着使用情况而变化，因此建议定期检查并管理 `Ubuntu` 的文件占用空间，以确保磁盘空间充足。

```bash
# 进入 wsl 启动 Ubuntu
wsl
# du [选项] [目录或文件] 查看文件内存占用 du -s[总计]-h[可读性] [目录未指定时统计当前目录]
du -sh

#  只统计 /path 前两层子目录的磁盘占用
du -h --max-depth=2 /path
```

- `Ubuntu` 文件存储在 `Windows` 文件系统的虚拟磁盘文件（通常为 VHD 或 VHDX 格式 `\\wsl$\Ubuntu\`）。`\\wsl$` 是一个特殊的 `UNC (Uniform Naming Convention)` 路径，可用于访问 `WSL` 的文件系统。

  默认情况下存储在 `Windows` 用户目录下的隐藏目录中，具体路径为 `%LOCALAPPDATA%\Packages\<DistroName>\LocalState\` ~~不建议去动它，避免造成不可预知的错误~~

- `WSL` 会将文件映射到 `Ubuntu` 文件系统中，使得 `Ubuntu` 中也可以访问和操作这些文件。`据我自己测试，是在 /mnt 文件夹下，eg. /mnt/c/...`

### 目录结构

| name               | 含义                                                                                       |
| ------------------ | :----------------------------------------------------------------------------------------- |
| `/bin`             | 包含了可执行的二进制文件，用于存放系统和用户级的基本命令和工具。                           |
| `/boot`            | 包含了系统引导相关的文件，如内核文件、引导加载程序和引导配置文件等。                       |
| `/dev`             | 包含了设备文件，用于与系统中的设备进行交互，如硬盘、终端、USB 设备等。                     |
| `/etc`             | 包含了系统级的配置文件，如网络配置、用户账户配置、服务配置等。                             |
| `/home`            | 包含了用户的个人目录，每个用户都有一个对应的目录，用于存放用户的个人文件、配置文件和设置。 |
| `/lib` 和 `/lib64` | 分别包含了系统和用户级的共享库文件，用于支持程序运行时的依赖关系。                         |
| `/media`           | 用于挂载可移动介质，如光盘、USB 设备等。                                                   |
| `/opt`             | 用于安装第三方软件的目录，通常由用户自行安装的软件会被安装到该目录下。                     |
| `/proc`            | 包含了内核和进程相关的虚拟文件系统，用于查看系统运行时的信息和状态。                       |
| `/root`            | 超级用户（root）的个人目录，通常只有超级用户才有权限访问。                                 |
| `/sbin`            | 包含了系统级的可执行二进制文件，用于存放系统管理和维护的工具。                             |
| `/srv`             | 用于存放系统服务相关的数据文件，如网站的数据文件、FTP 服务器的文件等。                     |
| `/tmp`             | 用于存放临时文件和目录，通常在系统重启时会被清空。                                         |
| `/usr`             | 用于存放用户级的应用程序、库文件、头文件、文档等。                                         |
| `/var`             | 包含了系统运行时产生的变化的文件，如日志文件、数据库文件、缓存文件等。                     |

> 如果需要查看文件内容

### 文件操作指令

一些常用的文件操作指令：

```bash
# 创建文件夹
mkdir [文件夹名称]

# 创建空文件 touch test.txt
touch [文件名]
# 删除文件 -r[删除所有文件] -f[强制删除，不提示]   rm -rf *
rm [文件名]
# 移动 | 重命名文件 文件 -> 文件夹 || 文件夹 -> 文件夹[已存在]，为移动，否则就是重命名
mv [旧文件] [文件 | 文件夹路径]
# 复制文件 同上解释
cp [旧文件] [新文件名 | 路径]

# 创建并写入内容到文件
echo "Hello, World!" > file.txt

# 查看文件内容
cat [文件名]
# 分页查看内容
less [文件名]
more [文件名]

# 使用文本编辑器编辑文件 --- 详细可见另外一篇文章 【vi、vim 编辑器学习】
vi/vim [文件名]
nano [文件名]
```
