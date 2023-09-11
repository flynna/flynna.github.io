---
title: 基于 Husky 和 Commitlint 实现 Git 提交规范校验
date: 2022-08-01 11:07:39
updated: 2022-08-01 11:07:39
tags:
  - husky
  - git-hooks
  - commitlint
categories:
  - 前端工程化
  - 代码规范约束
---

### Before...

`husky`在前端工程化的过程中可以说是必不可少，`why?` 它可以让我们在项目中更加方便的去使用`git hooks`，而非传统意义上的`.git/hooks`下编辑修改。

`git hooks`则是`git`在触发某个重要操作时自动执行的脚本，本文主要围绕`commit-msg and pre-commit`介绍。[了解更多](https://git-scm.com/docs/githooks)

<!-- more -->

### `husky@6↑`工作原理

> `husky@6`放弃了原有的配置方式（创建所有类型的`git hooks`，方便用户设置任何类型钩子都能正常工作）。新版本放弃了使用默认的`.git/hooks`，而是根据`git@2.9`提供的新特性`core.hooksPath`指定了`hooks`所在的目录`.husky/`，实现只添加用户想要的`hooks`

> ~~既然知道了新版本的破坏性变更，以及其原理，那肯定选择用新版本。目前最新版已经到`8.0.1`了.~~

### **新版`husky`使用**

#### 安装`husky`

```bash
yarn add -D husky
```

#### 卸载`husky`

```bash
yarn remove husky
git config --unset core.hooksPath
```

#### 初始化`husky`

```bash
npx husky install
# 或者添加 prepare 脚本命令
npm set-script prepare "husky install"
yarn prepare

# install 到指定的目录
npx husky install .config/husky
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
npx husky add .husky/pre-commit 'npm run lint'
# or 如果你的项目使用了脚手架的 lint，例如 vue-cli-service...
npm set-script lint "vue-cli-service lint"
```

> 钩子里配置的`npm run lint`，请确认你的项目已经包含了`lint`校验配置，并自行安装`devDependencies`。~~只是个栗子，不一定就是做`lint`校验 ❤❤❤~~ 效果图如下：

[![git-precommit-and-commitmsg-hooks-p1](/images/share/git-precommit-and-commitmsg-hooks/p1.png)](/images/share/git-precommit-and-commitmsg-hooks/p1.png)
[![git-precommit-and-commitmsg-hooks-p2](/images/share/git-precommit-and-commitmsg-hooks/p2.png)](/images/share/git-precommit-and-commitmsg-hooks/p2.png)

### `pre-commit`搭配`lint-staged`

上面只是举了一个栗子...`pre-commit`的时候，还可以做得到事情有很多，例如执行测试脚本...

**But...每次触发 pre-commit 都对所有的文件执行 lint，属实是有一点点恶趣味了，这里使用 lint-staged 工具做一点优化.** ~~如果你不介意...可以跳过下面的`lint-staged`😢😢😢~~

#### 安装

```bash
yarn add -D lint-staged
```

#### 配置

在项目的`package.json`中添加：

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

> 提交前会自动顺序执行`cmd`。例如上面配置，会先`eslint --fix`尝试修复，修复成功执行`add`操作，修复失败抛出异常，此时需要手动修复，然后再提交。

修改`.husky/pre-commit`钩子，改用`lint-staged`：

```bash
# 打开 .husky/pre-commit 文件，修改之前生成的 cmd
npx lint-staged

# or 配置 lint-staged 脚本
npm run lint-staged
# {
#   "scripts": {
#     "lint-staged": "lint-staged"
#   }
# }
```

[了解更多`lint-staged`](https://github.com/okonet/lint-staged)

除了在`commit`之前添加代码校验外，我们也可以对提交的`message(-m)`作规范约束。如何做？往下看 ↓

### `commit-msg`搭配`commitlint`

`commit-msg`是`git`提交时校验提交信息的钩子（此时由`husky`来指定），当触发时便会使用`commitlit`来校验。安装配置完成后，想通过`git commit`或者其它第三方工具提交时，只要提交信息（`-m`指定的`message`）不符合规范就无法提交，并提示。

> 约定格式： git commit -m \<type\>[optional scope]: \<description\>

#### 安装

```bash
yarn add -D @commitlint/cli @commitlint/config-conventional
```

#### 配置`commmitlint`

[更多配置项参考](https://github.com/conventional-changelog/commitlint/blob/master/%40commitlint/config-conventional/index.js)

```javascript
// commmitlint.config.js or .commmitlintrc.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  // rules：kv组成的对象，eg. 'name:[0, 'always', 72]'
  // 数组中第一位为 level，可选0,1,2，0为disable，1为warning，2为error
  // 第二位为应用与否，可选 always|never
  // 第三位该 rule 的值
  // rules: {}
};
```

然后添加`husky`的`commit-msg`钩子：

```bash
npx husky add .husky/commit-msg 'npx --no-install commitlint -e'
```

貌似已经很完美了，`emmm~~`但开始对规范使用不熟悉的童鞋，可能不太友好...我下面添加了`commitizen`优化方案使用，不需要可以跳过 😌😌😌

#### 配置`commitizen`

`commitizen`可以让用户通过界面化问答的方式完成提交信息的录入，并由用户决定是否推送（这个过程仅相当于命令`git commit -m 'xxx'`）

```bash
yarn add -D commitizen cz-conventional-changelog
```

配置`commitizen`并添加`commit`为`npm script`：

```json
{
  // ...
  "scripts": {
    // cz 本质就是 commitizen 一段短命令，代替 git commit 生成专业的 commit-message
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

[![git-precommit-and-commitmsg-hooks-p3](/images/share/git-precommit-and-commitmsg-hooks/p3.png)](/images/share/git-precommit-and-commitmsg-hooks/p3.png)

<div class="warning">

> 因为你，我的心脏总是忙个不停

</div>
