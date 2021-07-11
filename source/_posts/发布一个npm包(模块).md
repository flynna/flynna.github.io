---
title: 发布一个npm包(模块)
date: 2021-07-05 19:53:34
tags: [npm, npm-adduser, npm-publish]
# type: "categories"
categories: [大前端, Node]
---

[toc]

`react-field-viewer (根据传入的data 及 field-define(含组件的 tag，如 input、radio、selector...) 生成用于 table显示的 viewer组件)，发布至 npm or 私服。 实际项目中有用到，所以把过程及‘坑’就记录下来了`

## 1. 初始化

`若仓库已存在, 则忽略初始化的过程, 直接在原包基础上更新迭代，git clone 项目至本地完成开发`。

### 1.1 初始化 git 仓库

- `gitHub` 创建远程仓库
- 本地创建项目文件夹

```bash
mkdir componentName
cd ./componentName
```

- 本地与线上仓库建立关联

```bash
git remote add origin [线上仓库的SSH地址]
```

### 1.2 初始化 `npm` 及 `tsc`

```bash
npm init -y
# 安装 typescript
npm i typescript -D # or yarn add typescript --dev
# 初始化 typescript 生成 tsconfig.json 配置文件
tsc --init
```

### 1.3 安装项目依赖

`说明：我这里案例是开发的 react-typescript 功能组件，故部分依赖包自己斟酌是否需要安装`

```bash
# 1. 安装 react 相关依赖包
npm i react react-dom -D

# 2. 安装 babel 编译依赖包
npm i @babel/cli @babel/core @babel/preset-env @babel/preset-react -D
# or yarn add @babel/cli @babel/core @babel/preset-env @babel/preset-react --dev

# 3. 采用 webpack 构建，webpack-dev-server 作为本地开发服务
# 需要注意 webpack 版本，webpack5 build 会生成 LICENSE.txt 文件
npm i webpack webpack-cli webpack-dev-server -D

# 4. 安装相关 loader 编译代码 babel-loader 用来编译 jsx，ts-loader 编译 typescript
# 一个最基本的组件只需要编译 jsx，所以我这里没有安装 css 以及处理其他的 loader
npm i babel-loader ts-loader -D

# 5. ts 类型检查转译 awesome-typescript-loader
# 方案不唯一 参考: [https://juejin.cn/post/6844904052094926855]
npm i awesome-typescript-loader -D

# 6. 安装一个 webpack 插件 html-webpack-plugin ，用来生成 html
npm i html-webpack-plugin -D

# 7. 引入 NodeJs 环境变量types: @types/webpack-env
npm i @types/webpack-env -D

# 8. 引入 react 相关types: @types/react @types/react-dom
npm i @types/react @types/react-dom -D

# 9. 安装其他开发过程中需要使用的 组件
npm i antd moment -D
```

## 2. 修改配置文件

### 2.1 修改 `package.json`

```bash
# 添加 start 和 build 脚本
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "start": "webpack-dev-server --open development",
  "build": "webpack --mode production"
},

# 修改 license， 一般就是 MIT 和 BSD-3-Clause 之类
# 模块制定一个协议，让用户知道他们有何权限来使用你的模块，以及使用该模块有哪些限制
"license": "MIT",

# 描述你的 npm发布地址
"publishConfig": {
  "registry": "http://xx:8081/repository/npm-private/"
},

# 指定一个代码存放地址，对想要为你的项目贡献代码的人有帮助
"repository": {
  "type": "git",
  "url": "http://xx/x.git"
},

# 模块下文件名或者文件夹名 files：数组。or 添加 .npmignore 配置文件，作用类似 .gitignore，其权重更高
# .npmignore 内的文件配置会排除在外
"files": ["lib/**/*", "typings/**/*", "src/**/*"],

# 修改入口文件地址, 案例是通过 typescript 写的，需要指定 types 路径。
"main": "lib/index.js",
"types": "lib/index.d.ts",
```

### 2.2 修改 `tslint.json`

`第一步 tsc --init 后，会自动生成默认配置文件，不知道怎么配置可以查阅一下。`

`重点修改 outDir、jsx、typeRoots、types、include `

```json
{
  "compilerOptions": {
    /* Visit https://aka.ms/tsconfig.json to read more about this file */

    /* Basic Options */
    // "incremental": true,                   /* Enable incremental compilation */
    "target": "es5" /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019', 'ES2020', or 'ESNEXT'. */,
    "module": "commonjs" /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', 'es2020', or 'ESNext'. */,
    // "lib": [],                             /* Specify library files to be included in the compilation. */
    // "allowJs": true,                       /* Allow javascript files to be compiled. */
    // "checkJs": true,                       /* Report errors in .js files. */
    "jsx": "react" /* Specify JSX code generation: 'preserve', 'react-native', or 'react'. */,
    "declaration": true /* Generates corresponding '.d.ts' file. */,
    // "declarationMap": true,                /* Generates a sourcemap for each corresponding '.d.ts' file. */
    "sourceMap": true /* Generates corresponding '.map' file. */,
    // "outFile": "./",                       /* Concatenate and emit output to single file. */
    "outDir": "./lib/" /* Redirect output structure to the directory. */,
    // "rootDir": "./",                       /* Specify the root directory of input files. Use to control the output directory structure with --outDir. */
    // "composite": true,                     /* Enable project compilation */
    // "tsBuildInfoFile": "./",               /* Specify file to store incremental compilation information */
    "removeComments": true /* Do not emit comments to output. */,
    // "noEmit": true,                        /* Do not emit outputs. */
    // "importHelpers": true,                 /* Import emit helpers from 'tslib'. */
    // "downlevelIteration": true,            /* Provide full support for iterables in 'for-of', spread, and destructuring when targeting 'ES5' or 'ES3'. */
    // "isolatedModules": true,               /* Transpile each file as a separate module (similar to 'ts.transpileModule'). */

    /* Strict Type-Checking Options */
    "strict": true /* Enable all strict type-checking options. */,
    "noImplicitAny": true /* Raise error on expressions and declarations with an implied 'any' type. */,
    // "strictNullChecks": true,              /* Enable strict null checks. */
    // "strictFunctionTypes": true,           /* Enable strict checking of function types. */
    // "strictBindCallApply": true,           /* Enable strict 'bind', 'call', and 'apply' methods on functions. */
    // "strictPropertyInitialization": true,  /* Enable strict checking of property initialization in classes. */
    // "noImplicitThis": true,                /* Raise error on 'this' expressions with an implied 'any' type. */
    // "alwaysStrict": true,                  /* Parse in strict mode and emit "use strict" for each source file. */

    /* Additional Checks */
    // "noUnusedLocals": true,                /* Report errors on unused locals. */
    // "noUnusedParameters": true,            /* Report errors on unused parameters. */
    // "noImplicitReturns": true,             /* Report error when not all code paths in function return a value. */
    // "noFallthroughCasesInSwitch": true,    /* Report errors for fallthrough cases in switch statement. */

    /* Module Resolution Options */
    // "moduleResolution": "node",            /* Specify module resolution strategy: 'node' (Node.js) or 'classic' (TypeScript pre-1.6). */
    // "baseUrl": "./" /* Base directory to resolve non-absolute module names. */,
    // "paths": {},                           /* A series of entries which re-map imports to lookup locations relative to the 'baseUrl'. */
    // "rootDirs": [],                        /* List of root folders whose combined content represents the structure of the project at runtime. */
    "typeRoots": [
      "node_modules/@types",
      "./typings"
    ] /* List of folders to include type definitions from. */,
    "types": [
      "node",
      "webpack-env",
      "./typings"
    ] /* Type declaration files to be included in compilation. */,
    "allowSyntheticDefaultImports": true /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */,
    "esModuleInterop": true /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */,
    // "preserveSymlinks": true,              /* Do not resolve the real path of symlinks. */
    // "allowUmdGlobalAccess": true,          /* Allow accessing UMD globals from modules. */

    /* Source Map Options */
    // "sourceRoot": "",                      /* Specify the location where debugger should locate TypeScript files instead of source locations. */
    // "mapRoot": "",                         /* Specify the location where debugger should locate map files instead of generated locations. */
    // "inlineSourceMap": true,               /* Emit a single file with source maps instead of having a separate file. */
    // "inlineSources": true,                 /* Emit the source alongside the sourcemaps within a single file; requires '--inlineSourceMap' or '--sourceMap' to be set. */

    /* Experimental Options */
    // "experimentalDecorators": true,        /* Enables experimental support for ES7 decorators. */
    // "emitDecoratorMetadata": true,         /* Enables experimental support for emitting type metadata for decorators. */

    /* Advanced Options */
    "skipLibCheck": true /* Skip type checking of declaration files. */,
    "forceConsistentCasingInFileNames": true /* Disallow inconsistently-cased references to the same file. */
  },
  "exclude": ["node_modules"],
  "include": ["./src/**/*", "typings"]
}
```

### 2.3 添加 `tslint` 代码校验规则及 `editorconfig,prettier` 代码风格(非必须)

`tslint.json 非必须项，案例中未使用，自行了解配置，不做介绍。`

### 2.4 添加 `.editorconfig` 文件配置 编辑器习惯

`非必须，不做介绍，配置仅供参考`

```bash
# EditorConfig is awesome: https://EditorConfig.org

# top-most EditorConfig file
root = true

# Unix-style newlines with a newline ending every file
[*]
end_of_line = lf
insert_final_newline = true
charset = utf-8
indent_style = space
indent_size = 4

[{*.json,*.md,*.yml,*.*rc}]
indent_style = space
indent_size = 2
```

### 2.5 添加 `.prettierrc` 文件并配置

`在使用eslint规则时，本地编辑器的格式化代码会与eslint的规则冲突，这时可以自己配置一个.prettierrc格式化代码的文件`

```bash
# 案例采用的是 umi 的格式化规范 @umijs/fabric
npm i @umijs/fabric -D
```

```javascript
// .prettierrc or .prettierrc.js
const fabric = require("@umijs/fabric");
module.exports = {
  ...fabric.prettier,
};
```

- or 仅供参考

```bash
{
  "trailingComma": "all",
  "tabWidth": 4,
  "semi": false,
  "singleQuote": true,
  "endOfLine": "lf",
  "printWidth": 120,
  "overrides": [
    {
      "files": ["*.md", "*.json", "*.yml", "*.yaml"],
      "options": {
        "tabWidth": 2
      }
    }
  ]
}
```

### 2.6 添加 `.gitignore` 文件并配置

` Git 应当使用 .gitignore 文件忽略那些编译结果，以及 NPM 依赖的包文件`

```bash
node_modules
lib
.editorconfig
.npmrc
*.log
```

### 2.7 添加 `.npmignore`文件并配置

`作用及配置方法类似 .gitignore, 发布时排除文件中列出的文件夹或路径文件，作用与 package.json 中的 files相反，权重更高`

```bash
/.git/
/.vscode/
/node_modules/
.gitignore
.npmignore
.prettierrc
.editorconfig
tslint.json
tsconfig.json
note.md
*.log
```

### 2.8 配置 git 提交校验

`可省略，非必须。省略后直接 git commit -m '描述'`

```bash
npm i pre-commit -S
# pre-commit 可以自动在.git/下生成 pre-commit 脚本

# 在package.json中进行配置：
，再次提交 git commit 会先执行 lint 进行 tslint 代码检查
"scripts": {
  #...
  "lint": "tslint -p tsconfig.json"
}
# 此处的lint命令就是scripts的lint
"pre-commit": [
  "lint"
],
# 手动忽略当前提交检查
git commit -m '描述' --no-verify（或者-n）
```

### 2.9 配置 `webpack.config.js`

```bash
# 新建webpack配置文件 or 手动创建
touch webpack.confifg.js
```

- 案例中配置 （有遇到 bug： 导入包后无法识别 `types 参数类型`，`ps:`案例中未使用 export-import 方式访问类型）

`这里使用了 CopyWebpackPlugin 需要 npm i copy-webpack-plugin -D, 主要是为了解决： 不使用 import 在组件内部导入 interface，而是直接通过 d.ts 中声明访问时，打包后的组件 缺失 typings/index.d.ts，需要通过 这个 plugin 拷贝 typings 文件至打包后的 lib 文件中。`

```javascript
const path = require("path");
const CopyWebpackPlugin = require("copy-webpack-plugin");
const distPath = path.resolve(__dirname, "lib");

module.exports = {
  entry: "./src/index.tsx",
  output: {
    path: distPath,
    filename: "index.js",
    libraryTarget: "umd",
    libraryExport: "default",
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        loader: "ts-loader",
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: [".tsx", ".ts", ".js"],
  },
  // 案例中的组件在使用场景中需要从外部依赖获取 modules， 避免打包至 bundle
  externals: {
    antd: {
      commonjs: "antd",
      commonjs2: "antd",
      amd: "antd",
      root: "antd",
    },
    react: {
      root: "React",
      commonjs2: "react",
      commonjs: "react",
      amd: "react",
    },
    "react-dom": {
      root: "ReactDOM",
      commonjs2: "react-dom",
      commonjs: "react-dom",
      amd: "react-dom",
    },
  },
  plugins: [
    new CopyWebpackPlugin({
      patterns: [{ from: "typings/**/*", to: distPath }],
    }),
  ],
};
```

- `webpack.config.js` 简单的处理 `tsx` 文件/图片

```javascript
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const htmlWebpackPlugin = new HtmlWebpackPlugin({
  template: path.join(__dirname, "./example/src/index.html"),
  filename: "./index.html",
});

module.exports = {
  entry: path.join(__dirname, "./example/src/index.tsx"),
  output: {
    path: path.join(__dirname, "example/dist"),
    filename: "bundle.js",
  },
  module: {
    rules: [
      {
        test: /\.tsx?/,
        loader: "ts-loader",
      },
      {
        // pre/nomal/post - loader的执行顺序 - 前/中/后
        enforce: "pre",
        test: /\.tsx?/,
        loader: "source-map-loader",
      },
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.(png|jpg|gif|mp4)$/,
        use: {
          loader: "url-loader",
          options: {
            limit: 20,
          },
        },
      },
    ],
  },
  //映射工具
  // devtool: 'source-map',
  //处理路径解析
  resolve: {
    //extensions 拓展名
    extensions: [".tsx", ".ts", ".js", ".jsx", ".json"],
  },
  plugins: [htmlWebpackPlugin],
  devServer: {
    port: 3005,
  },
};
```

## 3. 组件功能开发

### 3.1 入口文件 `./src/index.tsx`

`最终暴露出去的组件，同样也是 外部 import 当前包后直接使用的组件，自行发挥。`

```tsx
import React, { FC } from "react";
import { componentMap, globalConfig } from "./constansts";

const FieldViewer: FC<ViewerProps> = (props) => {
  const Viewer = componentMap[props.define?.__config__?.tag || ""];

  if (!globalConfig.proxyPrefix) {
    globalConfig.proxyPrefix =
      props.proxyPrefix || `/${props.clientId}/app-factory-web`;
  }
  if (!globalConfig.clientId) {
    globalConfig.clientId = props.clientId;
  }

  return Viewer ? <Viewer {...props} /> : <span>{props.data}</span>;
};

export default FieldViewer;
```

### 3.2 其他 import 依赖自定义组件

- `./src/constansts.tsx` 读取 `packages/` 下的所有 `tsx` 组件文件, 存一份 map 索引（只是我案例开发的组件需要）

```tsx
export const componentMap: {
  [index: string]: React.FC<ViewerProps>;
} = {};

const viewersContext = require.context("./packages", true, /index\.tsx$/);

viewersContext.keys().forEach((viewer) => {
  const viewerModule = viewersContext(viewer);

  // 这里规定了模块中必须导出组件viewer 的tag, 未导出会保存在 unknowTag 属性上
  componentMap[viewerModule.tagName || "unknowTag"] = viewerModule.default;
});

if (process.env.NODE_ENV !== "production") {
  console.log(componentMap);
}

export const globalConfig = {
  proxyPrefix: "",
  clientId: "",
};
```

### 3.3 异步请求`./src/utils/axios.ts`

```typescript
import axios from "axios";
import { globalConfig } from "../constansts";

axios.interceptors.request.use((config) => {
  config.headers["client-id"] = globalConfig.clientId;
  return config;
});

export default axios;
```

### 3.4 field-define 解析生成单个 viewer

- 例如 `./src/packages/UploaderViewer/index.tsx` , 针对上传类型组件生成用于显示的 viewer

```tsx
import React, { FC } from "react";

const UploaderViewer: FC<ViewerProps> = ({ data }) => {
  let showValue = "";
  if (data) {
    showValue = data.map((d: any) => d.name).join("、");
  }
  return <span>{showValue}</span>;
};

export default UploaderViewer;
export const tagName = "sobey-uploader";
```

## 4. 单元测试及编写测试用例

`可省略，这里不做介绍。具体可见后续文章`

## 5. `npm`包发布

### 5.1 本地执行 build，测试是否能打包成功

### 5.2 测试组件功能是否实现

`将打包后的 lib 文件 拷贝至需要使用到这个组件的 项目的 node_modules 文件夹内(注意文件夹命名规范 同 发布的包名： package.json 中 name 属性值) 最终效果 node_modules/XX/lib`

### 5.3 编写 `readme` 使用文档

### 5.4 登录 `npm`并发布

`注意这里有坑(npm registry 地址导致)`

- 有账号 直接登录

```bash
npm login
# 输入账号、密码、邮箱(作者邮箱 随便写) ，提示登录成功

# 查看当前登录信息
npm whoami # or npm whoam i
npm publish

# 悲剧来了.......
# npm ERR! code E401
# npm ERR! Unable to authenticate, need: BASIC realm="Sonatype Nexus Repository Manager"

# 查阅了各种文档，总结如下：
# 1. 登录账号密码不匹配  ---- 排除
# 2. 私服(内部 npm) 设置未打开 设置–Security–Realms–把npm Bearer Token Realm添加到Active ---- 排除
# 3. 清除缓存:npm cache clear --force    ---- 没啥用
# 4. 注册账号已被使用  ---- 我这是登录  排除
# 5. package.json中限定的发布地址和你当前登录使用的地址冲突  ---- 验证

# 打开 C:\Users\xx\.npmrc 登录后会有 authToken 信息
# //xxxx/repository/npm-private/:_authToken=NpmToken.xxxx.xxxxx
# 对比 // 后面的 npm 地址 与当前 package.json 中 发布的地址 repository 是否一致。。。好家伙---不一样

# 问题找到了就好办了 先退出
npm logout
# 重新登录 并 指定 当前 login 的地址
npm login --registry=https://xxx
# 重新 输入账号密码邮箱
# 发布
npm publish
# ok 成功
# 登录私服或 npm， 查看刚发布的包
```

## 6. `npm`包使用及版本更新迭代

- 安装及使用

```bash
# 安装
yarn add react-field-viewer

# 使用
import Viewer from 'react-field-viewer';
<Viewer data={} define={} />
```

- 版本更新迭代

```bash
# 1. 拉取代码，正常开发维护 完成后 push

# 2. 版本号调整 更新功能后必须修改 version，不然会遇到问题。
# 可以手动修改 package.json 中的 version，也可以通过指令修改
# 指令参考 [https://www.jianshu.com/p/5565536a1f82]

# 3. 重新登录 npm 并执行 publish

# 4. 使用者对组件更新 --- 以下命令均可
yarn upgrade
yarn upgrade react-field-viewer
yarn upgrade [package@version]
yarn upgrade [package@tag]
```
