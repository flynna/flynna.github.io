---
title: 基于 lerna 管理 monorepo 的一次实践记录
date: 2022-07-26 20:25:37
updated: 2022-07-27 14:28:32
tags:
  - lerna
  - monorepo
categories:
  - 土豆の教程分享
  - 工具使用
---

### `lerna`是什么？

> Lerna is a fast modern build system for managing and publishing multiple JavaScript/TypeScript packages from the same repository.

[`lerna and nx`的其他博客文章](https://blog.nrwl.io/nx/home)

### 为什么要用`lerna`?

`官方解释：`

> Lerna 在 repo 中链接不同的项目，因此它们可以相互导入，而无需向 NPM 发布任何内容

> Lerna 对任意数量的项目运行命令，它以最有效的方式、以正确的顺序执行它，并且可以将其分布在多台机器上

> Lerna 管理您的发布流程，从版本管理到发布再到 NPM，它提供了多种选项来确保可以适应任何工作流程

结合工作中的实际体验，谈谈我的看法：

> 公司一般都有自己的 npm 私库，有很多的 package 相互之间存在或多或少的关联。随着业务需求的不断升级，我在维护这些 package 的时候就碰到了一个很操蛋的问题：比如我改了 packageA 然后发布，那么所有用到了 packageA 的其他 package，也要跟着升级 packageA 的依赖，然后再发布，过程显得很是繁琐，此时如果你的 package 是通过 lerna 管理，就可以很好的避免这个问题。

<!-- more -->

---

### 全局安装`lerna`

`tips: 注意 lerna@5.1.0 前后区别.`

> npm i -g lerna@4.0.0

### `lerna`使用实践

#### 初始化`git`仓库

```bash
# 初始化一个 utils 的 lerna 项目
mkdir utils && cd utils
git init
git remote add origin [xxx]
```

配置`.gitignore`

```
node_modules
lib
dist
logs
coverage
*/.config/*
```

#### `lerna`初始化

`lerna`有两种发布模式，固定模式`fixed`（默认）、独立模式`independent`。两者的区别：

> 固定模式：lerna init 通过 lerna.json 里的版本进行统一的版本管理。其中的任何一个包的改动都会导致所有的 packages 的版本号进行升级。

> 独立模式：lerna init \-\-independent 允许使用者对每个包单独改变版本号。每次发布的时候，针对所有有更新的 package 询问需要升级的版本号。（基于自身的 package.json 版本号）这种情况下，lerna.json 的版本号不会变化， 默认为 independent

~~建议使用独立模式...~~ 下面开始初始化过程（附效果图）：

> lerna init \-\-independent

[![lerna-p1](/images/share/lerna/p1.png)](/images/share/lerna/p1.png)

#### 完善`lerna.json`配置

指定命令使用的`client`，默认是`npm`，可以是`yarn`.修改`lerna.json`，添加：

> "npmClient": "yarn"

指定使用`yarn workspaces`模式，并在`lerna`中开启，同时在根目录下的`package.json`中添加`workspaces`：（开启后`lerna bootstrap`命令由`yarn install`代理，等价于在`workspace`的根目录下执行`yarn install`，只有顶层有一个`node_modules`），**这么做是因为 yarn 本身提供了较 lerna 更好的依赖分析与 hoisting 的功能**：

> "useWorkspaces": true

> "workspaces": \["packages/\*"\]

指定忽略发布的不必要的更新（比如`readme.md`）：

```json
// 方式一：在`lerna.json`中添加：
"ignoreChanges": ["**/*.md", "**/*.test.ts", "**/*.e2e.ts", "**/fixtures/**", "**/test/**", "**/__test__/**"]

// 方式二：在指定的命令下添加：
"command": {
  "publish": {
    "ignoreChanges": ["ignored-file", "*.md"]
  },
  "bootstrap": {
    "ignore": "component-*",
    "npmClientArgs": ["--no-package-lock"]
  }
}
```

##### `lerna.json`的配置参数说明

```json
// version：当前库的版本
// npmClient：允许指定命令使用的client， 默认是 npm， 可以设置成 yarn
// useWorkspaces：使用 yarn workspaces 模式
// ignoreChanges：一个不包含在 lerna changed/publish 的 glob 数组。使用这个去阻止发布不必要的更新，比如修复 README.md
// command.publish.ignoreChanges：可以指定那些目录或者文件的变更不会被 publish
// command.publish.registry：设置自定义的 npm 代理（比如使用公司自己搭建的私服）
// command.publish.conventionalCommits：lerna version 会自动决定 version bump 和生成 CHANGELOG 文件
// command.publish.message：一个 publish 时的自定义 commit 信息
// command.bootstrap.ignore：指定不受 bootstrap 命令影响的包
// command.bootstrap.npmClientArgs：指定默认传给 lerna bootstrap 命令的参数
// command.bootstrap.scope：指定那些包会受 lerna bootstrap 命令影响
// packages：指定包所在的目录
```

~~`ok`，准备工作做完了 😃😃😃~~

---

#### 创建`package`

这个过程可以通过手动创建，也可以通过命令生成：

```bash
cd ./packages
mkdir package-a package-b package-c
# 分别进入这三个目录初始化成包 ...
cd package-a && npm init -y
```

`or`：

```bash
# name：包名，loc 位置 [可选，不指定时默认就是 workspaces[0] 所指位置]
lerna create < name > [loc]
# 例如：
lerna create package-a
```

目录结构：

```
├── lerna.json
├── package.json
└── packages
    ├── packageA
    │   ├── index.js
    │   └── package.json
```

#### 安装依赖

##### 安装共用的`dependencices`or`devDependencices`

设置`root`的依赖，通常为一些开发工具. `eg: typescript、eslint、babel...`

```bash
yarn add -W -D typescript [-W -D == --ignore-workspace-root-check --dev]
# 卸载
yarn remove -W typescript
```

添加所有的`package`依赖（不包含`root`，而是在各自的`package.json`）：

```bash
lerna add lodash -D
```

##### 给指定的 packageA 安装依赖`A`模块

```bash
# 推荐
lerna add A packages/packageA

# 或者指定 --scope
lerna add A --scope=packageA
lerna add A --scope=packageA --dev

# 或（这种方式安装的 A 如果是当前工作区的开发模块，需要带上版本号）
yarn workspace packageA add A

# 安装 A 到指定前缀为 prefix- 的包
lerna add A packages/prefix-*

# 安装 A 到所有名为 packageA 包中
lerna add A **/packageA
```

##### `workspace`各`package`之间的依赖

`packages`下的各个包之间，也可以相互依赖，例如`moduleA`依赖了`moduleB`，而`moduleB`又依赖了`moduleC`。使用的方式同上，相差不大（基于软链接`symlink`实现）：

```bash
lerna add moduleB packages/moduleA
lerna add moduleC --scope moduleB
```

#### `lerna bootstrap`安装所有包的依赖

在`lerna`中，执行**默认的`bootstrap`命令**会在每个`package`下安装各自`package.json`中的依赖。

当你使用`yarn workspace，并在lerna中开启该功能时`，`lerna bootstrap`将由`yarn install`代理，等价体现为在`workspace`的根目录下执行`yarn install`.

```bash
lerna bootstrap
# 效果等价于
lerna link + yarn install
```

##### 优势

> 工作区的模块包之间可以相互依赖，并在你发布升级对应包的时候，自动检测其他依赖该模块的包。

> 相比 yarn link，这种方式只影响你工作区的依赖树，而不会污染全局。

> node_modules 统一安装，生成单一 lock 文件，方便 yarn 更好的管理并构建依赖。

#### 卸载依赖

> lerna exec -- <command> \[..args\] # 在所有包中运行该命令

可以依据这个命令来实现指定模块的包卸载，`eg`：

```bash
lerna exec --scope=packageA  yarn remove A # 将 packageA 包下的 A 卸载
lerna exec -- yarn remove A # 将所有包下的 A 卸载
```

#### 清理依赖包

快速删除所有模块中的`node_modules`文件夹。

```bash
lerna clean
```

[更多的 `bootstrap`细节](https://lerna.js.org/docs/features/bootstrap)

#### 列出工作区所有的`package`

如果与你文夹里面的不符，进入那个包运行`yarn init -y`解决.

```bash
lerna list
# or
lerna ls
```

#### 列出需要`publish`更新的包

```bash
lerna changed
```

> lerna 每次发布都会为对应的版本打 TAG，变动检测其实是依据 `git diff --name-only v版本`收集变动信息.

> `lerna diff`查看自上次发布以来的所有包或者指定包的 git diff 变化。

#### `lerna run`执行脚本

通过`lerna run xx`执行脚本时，`lerna`会先检测符合条件（含有`xx`命令）的`package`，然后再各自内部执行`npm run xx`.

```bash
lerna run test
# 区别于普通项目之处在于各个package之间存在相互依赖，如packageB只有在packageA构建完之后才能进行构建，否则就会出错，这实际上要求我们以一种拓扑排序的规则进行构建。
lerna run --stream --sort build
```

区别于`yarn workspaces run`：

> yarn workspaces run 执行 xx 指令时，必须所有的包都含有该 xx 命令，否则在执行过程中会抛出异常.

#### 发布

```bash
# 执行该命令后，会根据检测出的有变动的 package 提示你选择对应要升级到的新版本号
lerna publish
# 或者默认选项全部选择 Yes，并根据 commit 信息自动升级版本号
lerna publish -y
```

##### 建议

> 不要自己手动为`lerna`管理的仓库添加`tag`，防止 `package`变更检测异常，导致无法正常升级发布

> 尽量只在一个分支上发布，避免多个分支同时进行且生成相同版本

> 每次`publish`之前，先`commit`代码，保证工作区是干净的

> 确保你的`npm`账号是登录状态，否则会发布失败。`npm whoami`查看状态，可以指定`registry`

> 确保你的`package`设置了正确的`npm registry`地址，且发布的包与已存在的包之间不会存在冲突

> 按顺序执行`lerna bootstrap -> lerna run --stream --sort build（我在单个 package 中定义了一些 npm scripts，例如 prepublishOnly 钩子来执行 build，然而 npm-client 使用 yarn 后，这些钩子似乎并没有按预期进行工作，导致最终 publish 失败。通过 lerna run build 可以触发所有 package 执行 build，如果无此需求可以跳过该步骤）-> lerna publish.`

### 问题记录

#### 关于`--scope`的说明

> 不管是安装还是卸载，`--scope=packageA`中的`packageA`均是指的具体包名，而非`path`，而另外一种方式`packages/packageA`则是指具体路径.

#### 包名带有`scope`的发布?

形如`@xxx/xx`，在你的子包（具体要发布的那个包）的`package.json`中添加`publishConfig.access`字段：

```json
{
  // ...
  "publishConfig": {
    // ...
    "access": "public"
  }
}
```

##### 为什么`@`前缀包添加了`access`仍然抛出了`403`异常?

`@`符号后面的是你注册`npm`账户时的`username`，请确保该`scope`与你账户一致. ~~（前往官网注册你的账户，可以看到更加明确的错误提示）😂😂😂~~

#### 如何生成`changeLog.md`文件?

`lerna.json`中添加：

```json
{
  // ...
  "command": {
    // ...
    "publish": {
      // ...
      "allowBranch": "master", // 只在 master 分支执行 publish
      "conventionalCommits": true // 生成 changelog 文件
    }
  }
}
```

#### 如何只发布指定文件至`npm`?

> 添加 .npmignore 规则同 .gitignore，添加 .gitignore 也是可以的

> 或者配置当前包的 package.json 的 files 字段指定

#### 如何将预先存在的独立包收集到`lerna`管理的仓库中?

将带有提交历史记录的包导入 `packages/<directory-name>`. 保留原始提交作者、日期和消息。另外：如果您要在新的 lerna 存储库上导入外部存储库，请记住至少有一次提交。[-> 戳这里](https://github.com/lerna/lerna/tree/main/commands/import#readme)

```bash
lerna import <path-to-external-repository>
```

#### 发布失败后怎么重新发布?

运行`lerna publish`如果中途有包发布失败，再运行`lerna publish`的时候，因为`Tag`已经打上去了，所以不会再重新发布包到`NPM`.

先清空当前工作区的文件修改（主要是上一次发布时，每个`package`自身的`package.json`会修改`gitHead`，发布成功会被重置，失败后无法正常重置，需要手动放弃修改），然后：

> 运行 `lerna publish from-git`，会把当前标签中涉及的`NPM`包再发布一次，PS：不会再更新`package.json`，只是执行`npm publish`

`or`：

> 运行 `lerna publish from-package`，会把当前所有本地包中的`package.json`和远端`NPM`比对，如果是 NPM 上不存在的包版本，都执行一次`npm publish`
