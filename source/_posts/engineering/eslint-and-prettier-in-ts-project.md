---
title: 在 Typescript 项目中使用 Eslint 和 Prettier
date: 2022-09-15 10:27:23
updated: 2022-09-15 10:27:23
tags:
  - eslint
  - prettier
  - typescript
categories:
  - 前端工程化
  - 代码规范约束
---

### 为什么是`eslint`和`prettier`？

> 在`ESlint`推出`--fix`参数前`ESLint`并没有自动化格式代码的功能，要对一些格式问题做批量格式化只能用`Prettier`这样的工具。并且`Prettier`在代码风格的检测上比`ESlint`更全面，所以两者通常是结合在一起使用的。

> 对`typescript`代码进行`linting`时，有两个主要的`linting`工具可供选择：`tslint、eslint`，`tslint`是一个只能用于`typescript`项目的`linter`，而`eslint`同时支持`typescript、javascript`.

> 据`typescript`核心团队解释：`ESLint 具有比 TSLint 更高性能的架构`，并且后续在为`typescript`中`linting`集成时，`只会关注eslint`，另外官方自`2019`起已经弃用`tslint`，**[TSLint has been deprecated as of 2019](https://palantir.github.io/tslint/)**

<!-- more -->

### 在`typescript`项目中搭建`eslint`

> `tips`：如果是`create-react-app`创建的项目，`eslint`已经被作为依赖项集成，无需另外安装.

```bash
# @typescript-eslint/parser 允许 ESLint 对 TypeScript 代码进行 lint 解析
# @typescript-eslint/eslint-plugin 包含一堆特定于 TypeScript 的 ESLint 规则
yarn add -D eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

`then..`在项目根目录添加`.eslintrc.js`的配置文件，`eg`：

```javascript
module.exports = {
  root: true,
  env: {
    node: true,
  },
  parser: '@typescript-eslint/parser', // Specifies the ESLint parser
  extends: ['plugin:@typescript-eslint/recommended'],
  parserOptions: {
    ecmaVersion: 2020, // Allows for the parsing of modern ECMAScript features
    sourceType: 'module', // Allows for the use of imports
  },
  extends: [
    'plugin:@typescript-eslint/recommended', // Uses the recommended rules from the @typescript-eslint/eslint-plugin
  ],
  rules: {
    // Place to specify ESLint rules. Can be used to overwrite rules specified from the extended configs
    'no-console':
      process.env.NODE_ENV === 'production'
        ? ['error', { allow: ['error'] }]
        : ['warn', { allow: ['error'] }],
    'no-debugger': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    // -----
    // 配置 unused-imports 规则需要单独安装插件 yarn add -D eslint-plugin-unused-imports 然后在最外层添加配置项 plugins: ['unused-imports']
    // 'unused-imports/no-unused-imports': 'error',
    // -----
    // typescript configs e.g.
    '@typescript-eslint/no-unused-vars': 'off',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-explicit-any': 'off',
    '@typescript-eslint/consistent-type-imports': 'warn',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-empty-function': 'off',
    '@typescript-eslint/no-non-null-assertion': 'off',
    '@typescript-eslint/no-this-alias': 'off',
  },
};
```

> 如果在`react的ts`項目中使用，另外安装 `eslint-plugin-react dev`依赖项，并在上面配置的`.eslintrc`中添加配置：

```javascript
module.exports = {
  // ... others
  settings: {
    react: {
      version: 'detect', // Tells eslint-plugin-react to automatically detect the version of React to use
    },
  },
  parserOptions: {
    // ... others
    ecmaFeatures: {
      jsx: true, // Allows for the parsing of JSX
    },
  },
  extends: [
    // ... others
    'plugin:react/recommended',
  ],
};
```

当然你也可以配置`eslint`的`ignore`文件`.eslintignore`，符合其配置规则（语法同其他的`ignore`配置，如`.gitignore`）的文件将忽略`lint`检测。

😍😍😍[查看更多的配置项及规则](https://eslint.org/docs/latest/) -> [中文文档](http://eslint.cn/docs/rules/)

### 混入`prettier`配置

> 混用后，就可以通过 eslint --fix 来自动修复不符合 prettier 规则的代码

```bash
# eslint-config-prettier 禁用可能与 prettier 冲突的 ESLint 规则
# eslint-plugin-prettier 将 prettier 作为 eslint 规则运行
yarn add -D prettier eslint-config-prettier eslint-plugin-prettier
```

接下来在项目的根目录下添加一个`.prettierrc`文件，用以配置`prettier`，`eg:`

```json
{
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "arrowParens": "always"
}
```

继续...在原有的配置基础上，更新`.eslintrc.js`配置，用以支持混用：

```javascript
module.exports = {
  // ... others
  extends: [
    // ... others
    // 确保添加的 'plugin:prettier/recommended' 在 extends 配置的最后一项
    'plugin:prettier/recommended',
  ],
};
```
