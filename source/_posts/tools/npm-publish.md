---
title: 如何发布一个 npm-package?
date: 2022-07-22 10:50:45
updated: 2022-07-22 10:50:45
tags:
  - npm
  - npm-publish
categories:
  - 工具学习
  - Npm
---

### 什么是`npm-package`？

先说模块，`node`模块大致可以分为三类：内置模块（由`nodeJs`原生提供，可直接使用）、自定义模块（`module.export/require、export/import`）、第三方模块（需要通过`npm install`下载后才能使用）。

我这里提到`npm-package`的发布，自然是指的三方模块。后续如果有使用需求，可以直接`install`安装使用，大大提高咱的开发效率。

> 本文主要针对`publish`做说明。至于`package`模块如何定义，以及具体如何配置，后续会写另外一篇文章补充。

<!-- more -->

### 创建一个`npm`账号

如果你有相关的账号，请跳过该步骤；如果你没有`npm`的账号，那就需要先注册。注册有两种方式：官网注册 / 命令行创建

#### 官网注册

进入[https://www.npmjs.com/signup](https://www.npmjs.com/signup)输入用户、密码、邮箱注册。

#### 命令行创建

> **如果你通过这种方式创建账号，创建成功过后会默认你已经登录，无需使用登录指令再次登录。**

如果你配置过`npm-registry` ~~（例如你配置了淘宝镜像、或者你所在公司的`private-npm`）~~，请记得在发布之前切换到`npm`的官方源`https://registry.npmjs.org`。

##### 重置`registry`地址

**重置的前提**你这个包是要发布到`npmjs`，如果你要发布至你所配置的`registry`，那就不需要操作。

> npm config set registry https://registry.npmjs.org

总有一些懒人不想这么干，毕竟包发布完了过后还要切回原来的`registry`，觉得麻烦 ~~（不会承认是我）~~，有没有别的办法咧？

> 在使用 addUser 指令时，指定当前注册并登录的 registry，覆盖全局配置中的 registry.（不会修改全局配置） 具体用法如下：

##### 开始创建

> npm addUser \-\-registry=https://registry.npmjs.org

如果你是通过`npm config set registry xx`重置的`registry`：

> npm addUser

回车，你会得到一个`Username、Password、Email`的输入提示，录入你的用户信息（注意`password`不是明文显示的，会被隐藏）。

录完回车，得到一个`Enter one-time password from your authenticator app`的输入提示，录入验证码。~~这个验证码会发送到你上面填的邮箱里~~

继续回车，提示`Logged in as xxx on https://registry.npmjs.org/.`登录成功。

---

### 准备好模块

> **友情提示：最好为你的模块包添加私有前缀，避免与已经发布的包冲突，导致发布不成功...**

初始化一个项目，`-y`使用默认配置项。

```bash
mkdir test-publish-project
cd ./test-publish-project
npm init -y
```

创建一个`index.js`的入口文件（同`package.json`中指定的`main`字段），随便写点东西... ~~这里只是一个简单演示~~

```javascript
(function () {
  console.log('这是一个测试用的模块入口');
})();
```

---

### **登录`npm`**

前面有提到：通过`addUser`方式创建的账户在创建成功后会默认登录，所以如果你是通过命令行的方式注册的，可以跳过当前步骤...

> npm login

同时你也可以登录到指定`registry`，避免`publish`过程中抛出`401`。

> npm login \-\-registry=xxx

### **发布（`publish`）**

如果你不是通过`npm config set registry`修改的源地址，那么发布的时候同样要指定`registry`参数 😢😢😢。

> npm publish \-\-registry=xxx

有没有觉得这个很鸡肋？别急...咱这就解决

修改`package.json`，添加`publishConfig`配置项：

```json
{
  // other config...
  "publishConfig": {
    "registry": "https://registry.npmjs.org/"
  }
}
```

好了（意味着，以后只有`login or addUser`的时候需要指定`registry`🎉🎉🎉），现在可以愉快的直接使用`npm publish`。

发布成功！~~虽然遇到了一点点小问题，写在下面了~~

打开 [https://www.npmjs.com](https://www.npmjs.com) 找了一下我的包，纳尼 🤔🤔🤔，居然搜不到。。~~可能是因为缓存?~~

通过路径直接搜索：[https://www.npmjs.com/package/your-project-name](https://www.npmjs.com/package/your-project-name) 能找到，通过`npm install packagename`也成功了，说明包实实在在是发布上去了...

### **版本更新迭代**

添加/修改你的功能，修改完成以后...

#### 手动修改版本号

打开`package.json`文件，修改`version`字段。

#### 指令修改版本号

小版本升级：`1.0.0 -> 1.0.1`

```bash
# 如果没有预发布号：直接升级小号，去掉预发布号；如果有预发布号：去掉预发布号，其他不动
npm version patch
# 直接升级小号，增加预发布号为 0
npm version prepatch
```

中版本升级：`1.0.0 -> 1.1.0`

```bash
# 如果没有预发布号，则升级一位中号，大号不动，小号置为空；如果有预发布号小号为0，则不升级中号，将预发布号去掉
npm version minor
# 直接升级中号，小号置为 0，增加预发布号为 0
npm version preminor
```

大版本升级：`1.0.0 -> 2.0.0`

```bash
# 如果没有预发布号，则直接升级一位大号，其他位都置为0；如果有预发布号：中号和小号都为0，则不升级大号，而将预发布号删掉，中号小号存在不为0，则升级大号，清空预发布号。
npm version major
# 直接升级中号，小号置为 0，增加预发布号为 0
npm version premajor
```

##### 修改版本号的同时添加`commit`

> npm version \[patch\] -m '你的 commit 内容'

#### 再次发布

可以在发布之前确认一下当前登录信息`npm whoami --registry=xx`，如果没登录，需要重新登录。

> npm publish

### **包卸载**

参考：[https://www.npmjs.cn/cli/unpublish/](https://www.npmjs.cn/cli/unpublish/)

注意：`npm -f unpublish`不允许您取消发布超过`24`小时的任何内容。

> npm -f unpublish

**tips：如果这个包不是仅测试使用，建议不要删除...否则，对别的用户而言是极不道德的行为。**

---

### 过程中的错误记录

<div class="error">

> 400 Bad Request

</div>

`The password you have entered was detected on a public list of known compromised passwords. Please enter a different password.`

重新设个密码... ~~密码设置简单了，抛了个异常...~~

<div class="error">

> 403 Forbidden

</div>

`You do not have permission to publish "test-publish-project". Are you logged in as the correct user?`

~~what?告诉我没有权限发布...我这不是已经登录了吗，怎么肥四？~~

赶紧查一下是否登录成功！！！

> npm whoami \-\-registry=xxx

确实已经登录了 🤣🤣🤣。~~一万个尼玛心中飘过~~

查了相关资料，总结可能性：`该包已被别的作者发布`；`邮箱未验证`~~（扯淡，邮箱我已经验证了）~~

那就验证一下是包是否已经存在，去搜了一下（或者直接`install`），还真有... ~~那好吧，咱改个名~~，重新改了个包名（`package.json`中的`name`值）再次`publish`，成功!!!
