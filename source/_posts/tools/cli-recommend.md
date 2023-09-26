---
title: 一些常用的命令行工具推荐
date: 2023-09-25 11:34:36
updated: 2023-09-25 11:34:36
tags:
  - cli
categories:
  - 工具学习
  - 脚手架
---

罗列日常开发中用到的工具包以及脚手架。持续更新中...

<!-- more -->

### [`@antfu/ni`](https://github.com/antfu/ni)

自动检测项目中使用的包管理器，是 `npm or yarn or pnpm...` 然后自动选择对应的管理器运行。

```bash
npm i -g @antfu/ni

ni # npm install | yarn install ...
ni package # npm install package | yarn add package ...
ni package -D
ni -g package # yarn global add package

nr # npm run
nr dev # npm run dev | yarn dev ...

nlx # npx
nlx package # npx package | yarn dlx ...

nu # npm upgrade | yarn upgrade | yarn up | pnpm update ...
nu -i # yarn upgrade-interactive (Yarn 1) | yarn up -i (Yarn Berry) | yarn up -i (Yarn Berry)

nun # npm uninstall | yarn remove ...

nci # npm ci | yarn install --frozen-lockfile ...

na # npm | yarn ...
```

### [`depcheck `](https://github.com/depcheck/depcheck)

`Depcheck` 是一个工具，用于分析项目中的依赖项，以查看 `package.json` 哪些依赖项无用，以及缺少哪些依赖项。

```bash
npm install -g depcheck

depcheck # 根目录下执行即可 or
depcheck <项目目录>
```

<div class='info'>

常用参数：

- `--skip-missing=[true | false]`：默认 `false`，表示是否检测 `Missing` 的依赖包

- `--ignore-bin-package=[true | false]`：默认 `false`，表示是否忽略包含 `bin` 条目的包

- `--json`：表示所有包的检测结果以 `json` 格式输出，大概就是 `XX` 包在哪些文件使用了，{"包名":`["path1","path2"]`}

- `--ignores="eslint,babel-*"`：表示要忽略的包名称（逗号分隔），比如 `depcheck --ignores="eslint,@babel/*,babel-*"`

- `--ignore-path`：表示要忽略的文件的模式的文件的路径，比如 `depcheck --ignore-path=.eslintignore`

- `--ignore-dirs`：已经弃用，使用 `--ignore-patterns` 替代，表示要忽略的目录名，逗号分隔 `--ignore-dirs=dist,coverage`

- `--ignore-patterns`：表示要忽略的用逗号分隔的模式描述文件，比如 `depcheck --ignore-patterns=build/Release,dist,coverage,*.log`

- `--parsers, --detectors and --specials`：高级的语法使用参考[官方文档](https://github.com/depcheck/depcheck/blob/main/doc/pluggable-design.md)

- `--config=[filename]`：外部配置文件

</div>

外部配置文件：`.depcheckrc` 文件 （`yml/json` 格式），然后直接配置

```yml
ignores: ['eslint', 'babel-*', '@babel/*']
skip-missing: true
```

### [consoleImporter 插件](https://chrome.google.com/webstore/detail/console-importer/hgajpakhafplebkdljleajgbpdmplhie)

当我们需要对某一个三方库进行测试的时候，正常情况需要我们先搭建项目基础结构，然后安装模块在写案例并运行。~~但只是为了测一下功能用法，这样做会显得有些笨重 😅😅😅~~

安装浏览器插件，打开浏览器调试面板 `console`：

```bash
# 浏览器自带指令
$

# 安装 package
$i(package)

# 然后就可以正常使用模块的功能了
```

### 持续更新中...
