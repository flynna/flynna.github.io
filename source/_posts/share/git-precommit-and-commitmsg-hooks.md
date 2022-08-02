---
title: 基于husky和commitlint实现git提交规范校验
date: 2022-08-01 11:07:39
updated: 2022-08-01 11:07:39
tags:
  - husky
  - git-hooks
  - commitlint
categories:
  - 土豆の教程分享
  - 工具使用
---

### Before...

`husky`在前端工程化的过程中可以说是必不可少，`why?` 它可以让我们在项目中更加方便的去使用`git hooks`，而非传统意义上的`.git/hooks`下编辑修改。

`git hooks`则是`git`在触发某个重要操作时自动执行的脚本，本文主要围绕`commit-msg and pre-commit`介绍。[了解更多](https://git-scm.com/docs/githooks)

<!-- more -->

### `husky@6↑`工作原理

> `husky@6`放弃了原有的配置方式（创建所有类型的`git hooks`，方便用户设置任何类型钩子都能正常工作）。新版本放弃了使用默认的`.git/hooks`，而是根据`git@2.9`提供的新特性`core.hooksPath`指定了`hooks`所在的目录`.husky/`，实现只添加用户想要的`hooks`

> ~~既然知道了新版本的破坏性变更，以及其原理，那肯定选择用新版本。目前最新版已经到`8.0.1`了.~~

### **新版`husky`使用**

这里介绍了几种情况下的`husky`使用，本文后续也将统称`场景1、场景2、场景3...`

#### 安装`husky`

```bash
# 场景1：如果是 lerna 管理的 monorepo
yarn add -W -D husky

# 场景2：如果是单项目，那么 cd 到根目录
yarn add -D husky

# 场景3：如果是同个项目对应多端的 monorepo，例如的 vue-client、node-server
# 创建用于校验的 commit 文件夹
mkdir commit && cd ./commit
# 在该文件夹中 install 校验模块及工具，例如：
yarn add -D husky
```

#### 卸载`husky`

```bash
# 场景1、场景2
yarn remove husky
git config --unset core.hooksPath
# 场景3
cd ./commit && yarn remove husky
cd .. && git config --unset core.hooksPath
```

#### 初始化`husky`

**场景 1 and 场景 2：**

```bash
npx husky install
# 或者添加 prepare 脚本命令
npm set-script prepare "husky install"
yarn prepare

# install 到指定的目录
npx husky install .config/husky
```

---

**场景 3：**

先`cd`到创建的`commit`文件夹下：

```bash
cd ./commit
```

再添加`npm`钩子初始化`husky`：

```json
{
  "postinstall": "cd .. && husky install"
}
```

#### 创建`hooks`

> 语法：`husky add <file> [cmd]`

`eg.`

```bash
husky add .husky/pre-commit 'eslint'
```

#### 绕过钩子

```bash
# 添加 --no-verify eg.
git commit -m "test" --no-verify
```

#### 简单举例

##### 添加`pre-commit`钩子：

通过`husky add`钩子添加的`cmd`在创建后作为初始的脚本命令，你可以对其做任何修改。~~意味着创建的时候可以不加 😊😊😊~~

`eg.`当触发`pre-commit`的时候，对代码执行`lint`脚本检查：

```bash
npx husky add .husky/pre-commit 'npx eslint --ext .js,.ts,.vue src'
# or 如果你的项目使用了脚手架的 lint，例如 vue-cli-service...
npx husky add .husky/pre-commit 'npm run lint'
npm set-script lint "vue-cli-service lint"
```

效果图如下：

[![git-precommit-and-commitmsg-hooks-p1](/images/share/git-precommit-and-commitmsg-hooks/p1.png)](/images/share/git-precommit-and-commitmsg-hooks/p1.png)

### `pre-commit`搭配`lint-staged`

上面只是举了一个栗子...`pre-commit`的时候，还可以做得到事情有很多，例如执行测试脚本...

**But...每次触发 pre-commit 都对所有的文件执行 lint，属实是有一点点恶趣味了，这里使用 lint-staged 工具做一点优化.** ~~如果你不介意...可以跳过下面的`lint-staged`😢😢😢~~

#### 安装

```bash
# 场景1：
yarn add -W -D lint-staged
# 场景2、3
yarn add -D lint-staged
```

#### 配置

`场景1`：在`packages/`下的每个模块的`package.json`中添加，`场景2、3`：在各自项目的根目录的`package.json`中添加：

```json
{
  // 配置是一个对象，其中每个值都是要运行的命令，其键要用于此命令的 glob 模式，值（cmd）可以是数组，如果是数组，则按顺序运行命令。
  // 甚至支持配置成函数：(filenames: string[]) => string | string[] | Promise<string | string[]>
  // eg. '**/*.js?(x)': filenames => filenames.map(filename => `prettier --write '${filename}'`)
  "lint-staged": {
    // 例如此配置：通过 eslint 工具检测 修复提交的 .js .vue .ts 的错误项，然后添加到暂存区
    "**/*.{js,vue,ts}": ["eslint --fix", "git add ."]
    // 你也可以只是提交前检测
    // "**/*.{js,vue,ts}": "eslint"
  }
}
```

> 提交前会自动执行`eslint --fix`尝试帮你自动修复，修复成功执行`add`操作，修复失败抛出异常，此时需要手动修复。

然后修改`.husky/pre-commit`钩子，因为我们不需要每次提交都用`eslint`对所有文件检测，而是改用`lint-staged`：

```bash
# 场景2 .husky/pre-commit 文件，修改之前生成的 cmd
npx lint-staged

# 场景1、3：对于同 repo 的多个项目，此时可以分别添加 lint-staged 脚本，然后修改 .husky/pre-commit。注意：下面两个场景的命令均同场景2，是对 .husky/pre-commit 文件的 cmd 修改，而非手动执行

# 场景1: lerna run lint-staged 或者在 lerna 的 root 的 package.json 添加 lint-staged 脚本，然后修改 cmd：
npm run lint-staged
# 场景3：通过 cd 大法，挨个执行 lint-staged 脚本，eg：
cd pc && yarn run lint-staged && cd ../mobile && yarn run lint-staged
```

更多`lint-staged`了解

### `commit-msg`搭配`commitlint`

`commit-msg`是`git`提交时校验提交信息的钩子（此时由`husky`来指定），当触发时便会使用`commitlit`来校验。安装配置完成后，想通过`git commit`或者其它第三方工具提交时，只要提交信息不符合规范就无法提交，并提示。

#### 安装

```bash
# 场景1
yarn add -W -D @commitlint/cli @commitlint/conventional-commit
# 场景2
yarn add -D @commitlint/cli @commitlint/conventional-commit
# 场景3
cd ./commit && yarn add -D @commitlint/cli @commitlint/conventional-commit
```

#### 配置`commmitlint`

```javascript
// commmitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
};
```

然后添加`husky`的`commit-msg`钩子：

```bash
npx husky add .husky/commit-msg 'npx commitlint -E HUSKY_GIT_PARAMS'
# 场景3需要改造一下该 cmd，方便 commitlint 能够正确寻址，非安装
cd ./commit && npx commitlint -E HUSKY_GIT_PARAMS
```

~~这么做其实已经完成了咱想要的效果，我这里添加了`commitizen`优化方案使用，不需要可以跳过。~~

#### 配置`commitizen`

通过界面化问答的方式完成提交信息录入。

```bash
# 场景1
yarn add -W -D commitizen cz-conventional-changelog
# 场景2
yarn add -D commitizen cz-conventional-changelog
# 场景3
cd ./commit && yarn add -D commitizen cz-conventional-changelog
```

配置`commitizen`并添加`commit`为`npm script`：

```json
// package.json
{
  // ...
  "scripts": {
    "commit": "git-cz"
  },
  "config": {
    "commitizen": {
      "path": "cz-conventional-changelog"
    }
  }
}
```

后续`commit`，就可以使用`yarn commit`进行`commit`，其会自动做出如下提示：

[![git-precommit-and-commitmsg-hooks-p2](/images/share/git-precommit-and-commitmsg-hooks/p2.png)](/images/share/git-precommit-and-commitmsg-hooks/p2.png)
