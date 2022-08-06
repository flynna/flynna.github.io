---
title: vscode的使用，设置分享以及插件推荐
date: 2022-07-14 18:48:37
updated: 2022-08-07 20:17:31
tags:
  - vscode
  - plugins
  - 设置
categories:
  - 土豆の收藏安利
---

### 前言

> 工欲善其事，必先利其器

前端开发工具有很多，类似`HBuilder、WebStorm、Visual Studio Code...`等，其中`vscode`以其轻量且强大的代码编辑功能和丰富的插件生态系统，独受前端工师的青睐。

<!-- more -->

### 编辑器汉化

- 方式一：`Ctrl+Shift+P`，输入`configure language`回车，在打开的语言配置文件中将`en-us`修改为`zh-cn`，`Ctrl+S`保存设置，重启生效。

- 方式二：插件安装。打开扩展搜索`Chinese (Simplified) (简体中文) Language Pack`安装，重启生效。

---

### vscode 推荐的用户设置

左下角设置图标，点击，选择设置，搜索设置，设置以下内容(推荐)。或者直接`Ctrl+Shift+P`搜索`setting.json`，直接在文件中编辑修改，界面化操作的设置最终也会同步到该文件中。

#### 关闭`vscode`的自动更新

配置是否接收自动更新，更改后需要重新启动。

> update mode:none

#### 默认行尾字符

`windows`系统环境下的换行方式默认为`CRLF: \r\n`，`linux`系统环境下的换行方式默认为`LF: \n`。配置行位字符为`\n`。

> Eol: \n

#### 自动保存

避免一些不可逆的异常关闭导致数据丢失。`onFocusChange`-文件焦点变化时自动保存。

> autoSave: onFocusChange

#### `tab`相关

配置一个制表符等于的空格数以及是否启用`tab`补全。

> Tab Size: 2

> Tab Completion: on

#### '括号对'着色和匹配

`Bracket Pair Colorization`插件弃用，插件功能已经被`vscode`内置。直接设置就可以控制每个括号类型是否具有自己的独立颜色池。

> Bracket Pair Colorization: Enabled

> Bracket Pair Colorzation: Independent Color Pool Per Bracket Type

> Bracket Pairs: active

#### 其他配置项

设置字体大小

> Font Size: 14

控制在删除括号或者引号时编辑器是否应删除相邻的右引号或右方括号。

> Auto Closing Delete: always

在保存文件时自动运行`ESLint`的自动修复命令`eslint --fix`，修改`setting.json`

```json
{
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

超出显示范围，自动换行（只是显示效果上，并没有真正换行）

> word Wrap: on

在`Word Wrap`为`wordWrapColumn`或`bounded`时，控制编辑器的折行列。（与`word Wrap: on`相悖，根据自己喜好设置）

> Word Wrap Column: 100

差异编辑器是否忽略前导空格或尾随空格中的更改。

> Ignore Trim Whitespace: false

---

### 插件(`plugins`)推荐

#### [`Guides`](https://marketplace.visualstudio.com/items?itemName=spywhere.guides)

带色竖线提示所属区域块，效果如下：

[![vscode-setting-and-plugins-p1](/images/posts/vscode-setting-and-plugins/p1.png)](/images/posts/vscode-setting-and-plugins/p1.png)

#### [`ESLint`](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)-代码语法规范校验

该扩展使用安装在打开的工作区文件夹中的`ESLint`库。如果该文件夹没有提供一个，则扩展程序会查找全局安装版本。(需要先安装-一般安装在项目的`devDependenice`)

配置文件：添加`.eslintrc`，或者终端`Create ESLint configuration`，如果你在全局安装了`ESLint`，执行`eslint --init`。

#### [`Prettier - Code formatter`](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)-代码风格格式化

为不同文件配置默认的格式化程序。

- 可通过界面操作（右键 -> 使用...格式化文档 -> 配置默认格式化程序 -> 选择`prettier`）

- 在`setting.json`文件中添加字段配置。

```json
{
  // ... 其他配置
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

`prettier`相关配置项，可通过下面两种方式设置。

- 方式一，添加全局用户配置（`vscode`设置）

```bash
# 使用单引号。
singleQuote: true

# 尽可能控制尾随逗号的输出。
trailingComma: all

# 指定每行代码的最佳长度，如果超出长度则换行。
printWidth: 100

# 当箭头函数仅有一个参数时加上括号。
arrowParens: always
```

- 方式二：项目的`package.json`种添加`prettier`字段配置。

#### [`Document This`](https://marketplace.visualstudio.com/items?itemName=oouo-diogo-perdigao.docthis)-`js/ts`-注释生成

可以自动为`TypeScript`和`JavaScript`文件生成详细的`JSDoc`注释。使用：

- 方式一：，光标指中需要注释的函数体，按两次`Ctrl + Alt + d`即可。

- 方式二：打开命令行方式`Ctrl + Shift + p`输入`document this`。

#### [`TODO Highlight`](https://marketplace.visualstudio.com/items?itemName=wayou.vscode-todo-highlight)-高亮提示

代码行添加`TODO`字样，高亮效果提示。突出尚未完成的功能或者事情。

#### [`GitLens — Git supercharged`](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)和[`Git Graph`](https://marketplace.visualstudio.com/items?itemName=mhutchie.git-graph)

前者：

- 查看单个文件的提交记录
- 代码行尾浅色的`commit-message`
- 鼠标覆盖到代码块上，可以查看更详细的提交信息和`commit-id`，更甚至能跳转到当次记录查看详细提交
- `git`命令界面化操作
- 其他的侧边栏视图和功能

后者：

- 图形化展示所有分支及结构，可选择性过滤分支再查看
- 可直接选择某一次的提交节点右键执行某些操作（`revert/cherryPick/createBranch/drop?/checkOut/addTag`等）
- 点击节点，查看当次提交的详细文件修改，进行代码审查
- 单击一个提交，`CTRL/CMD`单击另一个提交比较任何两个提交
- 其他功能

#### [`Search node_modules`](https://marketplace.visualstudio.com/items?itemName=jasonnutter.search-node-modules)-`node_modules`文件搜索

`Ctrl + Shift + p`，或者`Ctrl + p 输入 >`。在打开的面板中输入或者选择`Search node_modules`。点击查询出来的`Search node_modules`后，会根据当前的工作区文件夹查找`node_modules`并打开，支持**类似**`Ctrl + f`对路径的文件夹或者文件进行搜索。

#### [`DotENV`](https://marketplace.visualstudio.com/items?itemName=mikestead.dotenv)-`.env`文件代码高亮

#### [`Import Cost`](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost)-内联显示导入的包文件大小

#### [`Template String Converter`](https://marketplace.visualstudio.com/items?itemName=meganrogge.template-string-converter)-字符串与模板字符串的自动转换

#### [`TypeScript Barrel Generator`](https://marketplace.visualstudio.com/items?itemName=eliostruyf.vscode-typescript-exportallmodules)-`ts`文件导出到`index`

允许通过或者自动导出某个文件夹（模块）的所有`export`至`index.ts`，使用时直接引入`index`即可。

使用：

- 方式一：右键文件夹，选择`Typescript: export all modules`

- 方式二：手动创建`index.ts`，如下：

```typescript
// 例如 folder/a、folder/b 都 export 了一些变量或者函数
// 创建 folder/index.ts
export * from './a';
export * from './b';
```

#### [`Code Runner`](https://marketplace.visualstudio.com/items?itemName=formulahendry.code-runner)-代码运行器

点击编辑器右上角类似 |> 的按钮，即可执行当前文件代码片段，并在编辑器内输出结果。

#### [`Path Intellisense`](https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense)-路径感知

输入路径时，智能提示，帮助自动完成文件名填充。

#### `Better Comments`-注释美化

根据特殊字符前缀标识不同类型的注释。

#### [`Auto Rename Tag`](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-rename-tag)和[`Auto Close Tag`](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-close-tag)

前者：标签重命名时，成对的另外一半标签会自动重命名。

后者：输入前半个标签`<div>`，会自动生成另外一半用于关闭的标签`</div>`

#### [`Markdown All in One`](https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one)-`Markdown 所有功能支持`

在`typora`开始收费后，发现的一款`markdown`插件。`.md`文件的右上角类似窗口扩展的查看图标，点击后会在右侧窗口打开`Markdown`的预览效果页面。

#### [`open in browser`](https://marketplace.visualstudio.com/items?itemName=techer.open-in-browser)-浏览器打开文件

在写`html`文件的时候，方便快速在浏览器打开该文件。`Alt + b`快捷键打开，或者右键 `open in browser`。对应还有一个插件`View In Browser`，不过作者已经不再维护了。

#### [`Flutter Color`](https://marketplace.visualstudio.com/items?itemName=circlecodesolution.ccs-flutter-color)-颜色代码效果实时预览

<div class="success">

> 框架相关

</div>

#### [`ES7+ React/Redux/React-Native snippets`](https://marketplace.visualstudio.com/items?itemName=dsznajder.es7-react-js-snippets)-`React`代码片段

#### [`Vue 3 Snippets`](https://marketplace.visualstudio.com/items?itemName=hollowtree.vue-snippets)-`Vue`代码片段

基于`Vue 2 Snippets`开发的兼容适配`Vue 3`的插件。另外一个片段插件`VueHelper`。

#### [`Vue Language Features (Volar)`](https://marketplace.visualstudio.com/items?itemName=Vue.volar)-`Vue 语言扩展`

为`Vue template`提供原生的`TypeScript`语言服务。推荐在`Vue3`项目中开启。

#### [`Vetur`](https://marketplace.visualstudio.com/items?itemName=octref.vetur)-`Vue相关支持`

- 语法、语义高亮
- `Vue`代码片段
- 格式化相关代码
- 智能感知及`debug`
- 错误检测
- 其他功能

<div class="success">

> 主题相关

</div>

#### [`One Dark Pro`](https://marketplace.visualstudio.com/items?itemName=zhuangtongfa.Material-theme)和[`Material Theme`](https://marketplace.visualstudio.com/items?itemName=Equinusocio.vsc-material-theme)-`vscode主题`

#### [`Material Icon Theme`](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme)-`vscode文件图标主题`

---

### 常用快捷键

#### 跳转到代码行

`Ctrl + g`或者`Ctrl + p`输入`:`，再输入行号回车，即可完成行跳转

#### 文件搜索

`Ctrl + p`输入文件名，回车。文件內搜索`Ctrl + f`

#### 打开最近工作区或者文件（夹）

`Ctrl + r` 选择你的工作区，按住`Ctrl`点击打开，会新开一个`vscode`打开。不加`Ctrl`会在当前窗口替换并打开。

#### 新建文件和新开`vscode`

`Ctrl + n`新建文件，`Ctrl + Shift + n`新开`vscode`。

#### 新键行

`Ctrl + 回车`，效果类似在行尾按下回车。带来的便捷就是不需要先将光标移动到行尾。

#### 打开文件`Ctrl + o`，打开、关闭终端`Ctrl + ~`

#### 代码行快速复制、剪贴

当需要对整行代码操作时，无需选中直接`Ctrl + c`即可复制，`Ctrl + x`即可剪贴。

同时按住`Shift + Alt + 方向键（上/下）`，可实现基于当前行的向上向下复制粘贴。

#### 缩进调整

`Tab`向后缩进，`Shift + Tab`向前缩进。

#### 批量选中

按住`Shift`，鼠标左键点击开头结尾（或者方向键），即可选中。`Ctrl + a`选中所有。选中变量`or`函数后`Ctrl + d`持续使用即可**批量查找**该函数或变量。

#### 修改引用

鼠标点击变量或者函数，按下`F2`，输入你要更改的名称，回车即可**同步修改**所有用到该变量的名称。

#### 添加注释

单行注释`Ctrl + /`，多行注释`Shift + Alt + a`。

#### 多光标

按住`Alt`键，鼠标左键每次点击都会生成一个新的光标。或者`Ctrl + Alt + 方向键`创建多个光标。

#### 窗口拆分

`Ctrl + \`，在右侧开一个编辑器。

#### 窗口视图移动（滚轮效果）

按住`Ctrl`，点击方向键上下，即可实现滚动。

#### 代码行移动

按住`Alt`，点击方向键上下，即可完成光标代码行的移动，如行 11 移动到行 19。

#### 代码`debug`

- 方式一：可以选择侧边栏的调试按钮，选择配置进行调试。

- 方式二：`F5`运行调试。

#### 代码格式化

> (Code 快捷键 Shift + Alt + F) 格式化工具使用 prettier

> (Code 快捷键 Shift + Alt + O) 整理 import 引用
