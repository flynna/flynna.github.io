---
title: hexo的服务搭建和简单使用
date: 2022-07-08 15:31:02
updated: 2022-07-08 15:31:02
tags:
  - hexo-cli
  - hexo
categories:
  - 工具学习
  - Hexo
---

### What is Hexo?

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。[详见文档](https://hexo.io/docs/)

<!-- more -->

### 开始搭建你的`Hexo`服务

#### 安装脚手架

> npm install -g hexo-cli
> npm install hexo

#### 初始化你的博客项目

> hexo init <blogName>
> cd blogName
> yarn or npm i

#### 配置你的博客信息

`_config.yml`即为博客的配置信息文件，详见[配置](https://hexo.io/zh-cn/docs/configuration) 。

#### 主要文件夹说明

`scaffolds`文件夹可以配置你的文章模板，即每次 hexo 新建文章都会根据此文件夹配置建立文件。

`source`就是你的文章相关的资源文件。

`themes`主题文件夹，根据主题来生成对应静态页面。

#### 启动服务

在`package.json`中可以发现程序的启动指令。

> hexo server [hexo s] [yarn server]

在`terminal`中找到服务地址，默认`http://localhost:4000/`,打开浏览器即可浏览你的博客

### 根据你的口味配置想要的主题模板

- 打开`github`，搜索`hexo-theme`，根据`star`的数量排个序，挨个去看效果，选择自己心仪的主题

- `cd themes && git clone xxx`，将找到的主题文件资源克隆到`themes`文件夹下

- 根据主题的相关配置项提示进行配置，重启服务即可看到效果。(每个主题可能配置方式不一样，大同小异)

### 开启你的博客之路

> 新建文章，`layout`对应`draft、post、page(草稿source/_drafts、文章source/_posts、页面source)`

```bash
hexo new [layout] <title>
```

其他辅助参数：

```
-p, --path	自定义新文章的路径
-r, --replace	如果存在同名文章，将其替换
-s, --slug	文章的 Slug，作为新文章的文件名和发布后的 URL
```

`eg`. 我想在`_posts/share`文件夹下创建一片`a.md`的文章，且文章的页面`title`为‘这是 A’

```bash
hexo new post -p share/a '这是A'

# 创建一篇草稿
hexo new draft ...
# 发布
hexo publish [layout] <title>
```

> 其他常用指令说明：

```bash
# 生成静态文件，在 public 文件夹下
hexo generate
hexo g

# 清除缓存文件 (db.json) 和已生成的静态文件 (public)
hexo clean

# 部署网站 -g, --generate	部署之前预先生成静态文件
hexo deploy

# 草稿发表，移动到 source/_posts 文件夹
hexo publish [layout] <filename>
```

### [将`Hexo`部署到`GitHub Pages`](https://hexo.io/zh-cn/docs/github-pages)
