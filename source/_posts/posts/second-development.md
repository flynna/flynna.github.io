---
title: 基于开源项目二次开发的解决方案
date: 2023-10-19 11:48:47
updated: 2023-10-19 11:48:47
tags:
  - 二次开发
categories:
  - 收藏安利
  - Github
---

### `Before...`

针对开源项目进行二次开发，在日常开发中已经很常见了。它可以为我们提供很多便利：

- 定制需求：开源项目通常是通用性的，为了满足特定组织或项目的需求，可能需要定制功能、界面。

- 功能扩展：有时候，开源项目的功能不足以满足特定用例的需求，你可以扩展和增强现有功能。

- 修复错误和漏洞：开源项目可能存在错误和漏洞，当然你也可以提 `PR` 修复，贡献社区。

- 整合与互操作：在实际应用中，可能需要将多个开源项目整合在一起，以构建更大的系统。

- 性能优化：你可以对开源项目进行性能优化，以满足特定的性能需求，例如提高响应时间或减少资源消耗。

<!-- more -->

了解了二次开发的必要性，下面我们开始准备工作：

### 准备

准备一个远程仓库，用于存放二次开发后的代码。~~当然，这个仓库可以是公司私服~~

克隆开源项目到本地，并创建自己的开发分支。为了方便后期维护，建议将开源项目的版本号作为分支名。

```bash
git clone https://github.com/ElemeFE/element.git

cd element

# 基于 2.15.13 版本创建自己的开发分支
git checkout -b my2.x v2.15.13
```

修改当前 `remote` 名称，并将新的 `remote` 指向自己的远程仓库。

```bash
# 查看现在的 origin
git remote -v

# 改名
git remote rename origin element

# 添加新的 remote，关联到自己的远程仓库
git remote add origin https://github.com/flynna/el-dev.git

# 添加远程跟踪，与本地分支和新的远程分支之间的关联
git branch --set-upstream-to=origin/my2.x
```

推送代码到远程

```bash
git push origin my2.x

# 设置了 --set-upstream-to 后，可以简化
git push
git pull
```

准备已就绪，现在二次开发的项目仓库和配置就搭建好了。你可以写一些自己的代码然后推送（此时推送到的是自己的远程仓库）

### 更新

当开源项目更新时，如果你有需要，你可以将它更新的代码合并到自己开发的分支中，然后再推送到创建的远程仓库。

更新开源项目远端库的 `tags`：

```bash
git fetch element --tags
```

合并新的功能分支（`tag`）到当前开发分支（假设有新的资源 `v2.15.14`）：

```bash
git merge refs/tags/v2.15.14
```

> **注意：`tags` 数据在 `fetch` 下来后是不分远端库的（即使它有多个远端库），所以这里可以直接合并，日常开发应该注意避免相同 `tag` 导致冲突**

推送更新资源到自己的远程仓库：

```bash
git push
```

过程如图：

[![second-development-p1](/images/posts/second-development/p1.png)](/images/posts/second-development/p1.png)

[![second-development-p2](/images/posts/second-development/p2.png)](/images/posts/second-development/p2.png)
