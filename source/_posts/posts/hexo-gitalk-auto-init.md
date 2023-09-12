---
title: Hexo-Gitalk 评论模块的自动化创建
date: 2022-07-11 16:48:14
updated: 2022-07-11 16:48:14
tags:
  - hexo
  - gitalk
  - issue
categories:
  - 收藏安利
  - Hexo
---

目前几乎所有的 `hexo` 框架都为大家集成了 `comment` 评论模块，本文主要面向的是这部分已集成 `Gitalk` 评论的博客。~~当然，你也可以通过插件集成到没有提供 comment 的 hexo 也是可以的 😊😊😊~~

在该框架基础上，完成 `写文章 -> 文章发布 -> issue 初始化 -> 可评论` 这个自动化流程。

避免打开新发布的博客文章后，**底部提示 `未找到相关的 Issues 进行评论 请联系\***\***\* 初始化创建`，需要登陆 `github`，完成初始化后才能使用**。

​ 咱总不能新写一篇文章就去仓库手动初始化 `issue` 吧....有点阔怕 🤐🤐🤐

<!-- more -->

### 自动化创建 `issues` 评论仓库

#### `sitemap` 站点地图

站点地图是一种文件，您可以通过该文件列出您网站上的网页，从而将您网站内容的组织架构告知 `Google` 和其他搜索引擎。搜索引擎网页抓取工具会读取此文件，以便更加智能地抓取您的网站 。

​ 简言之，就是通过 `sitemap`，记录下当前博客的所有地址链接(包括所有的文章)，用于后期自动化创建。

- 通过插件生成 `sitemap` ( 在你 `hexo` 的根目录，执行下面两个命令来安装针对 `google` 和百度的插件 )：

> `npm i hexo-generator-sitemap hexo-generator-baidu-sitemap --save`

- 根目录下的 `_config.yml` 配置 `sitemap` 映射

```yaml
# hexo sitemap网站地图
sitemap:
  path: sitemap.xml
baidusitemap:
  path: baidusitemap.xml
```

- 执行 `hexo generate` , `public` 文件夹下面，就会生成 `sitemap.xml`和`baidusitemap.xml `

#### `access token` 获取

​ 有了 `sitemap` 后， 我们需要调用 `github` 的相关接口，`token` 必不可少，`token` 怎么获取呢？ 创建一个就是了

- [创建一个 `access token`](https://github.com/settings/tokens/new)
- `access token` 配置项： `Note` 描述(随便写), `Select scopes / repo` 勾选 `repo:status repo_deployment public_repo`
- 将生成的 `token` 记录下来，方便自动化时使用

#### 添加脚本文件

- 安装脚本文件依赖包

> `npm i request xml-parser yamljs cheerio --save`

- 添加 `comment.js` 脚本代码（部分代码需要结合自身情况修改）

```javascript
const request = require('request');
const fs = require('fs');
const path = require('path');
const url = require('url');
const xmlParser = require('xml-parser');
const YAML = require('yamljs');
const cheerio = require('cheerio');
// 根据自己的情况进行配置
const config = {
  username: 'GitHub 用户名', // GitHub 用户名
  token: 'GitHub Token', // GitHub Token
  repo: 'xxx.github.io', // 存放 issues的git仓库
  // sitemap.xml的路径，commit.js放置在根目录下，无需修改，其他情况自行处理
  sitemapUrl: path.resolve(__dirname, './public/sitemap.xml'),
  kind: 'Gitalk', // "Gitalk" or "Gitment"
};
let issuesUrl = `https://api.github.com/repos/${config.username}/${config.repo}/issues?access_token=${config.token}`;

let requestGetOpt = {
  url: `${issuesUrl}&page=1&per_page=1000`,
  json: true,
  headers: {
    'User-Agent': 'github-user',
  },
};
let requestPostOpt = {
  ...requestGetOpt,
  url: issuesUrl,
  method: 'POST',
  form: '',
};

console.log('开始初始化评论...');

(async function () {
  console.log('开始检索链接，请稍等...');

  try {
    let websiteConfig = YAML.parse(
      fs.readFileSync(path.resolve(__dirname, './_config.yml'), 'utf8'),
    );

    let urls = sitemapXmlReader(config.sitemapUrl);
    console.log(`共检索到${urls.length}个链接`);

    console.log('开始获取已经初始化的issues:');
    let issues = await send(requestGetOpt);
    console.log(`已经存在${issues.length}个issues`);

    let notInitIssueLinks = urls.filter((link) => {
      return !issues.find((item) => {
        link = removeProtocol(link);
        return item.body.includes(link);
      });
    });
    if (notInitIssueLinks.length > 0) {
      console.log(`本次有${notInitIssueLinks.length}个链接需要初始化issue：`);
      console.log(notInitIssueLinks);
      console.log('开始提交初始化请求, 大约需要40秒...');
      /**
       * 部署好网站后，直接执行start，新增文章并不会生成评论
       * 经测试，最少需要等待40秒，才可以正确生成， 怀疑跟github的api有关系，没有找到实锤
       */
      setTimeout(async () => {
        let initRet = await notInitIssueLinks.map(async (item) => {
          let html = await send({ ...requestGetOpt, url: item });
          let title = cheerio.load(html)('title').text();
          let pathLabel = url.parse(item).path;
          let body = `${item}<br><br>${websiteConfig.description}`;
          let form = JSON.stringify({ body, labels: [config.kind, pathLabel], title });
          return send({ ...requestPostOpt, form });
        });
        console.log(`已完成${initRet.length}个！`);
        console.log('可以愉快的发表评论了！');
      }, 40000);
    } else {
      console.log('本次发布无新增页面，无需初始化issue!!');
    }
  } catch (e) {
    console.log(`初始化issue出错，错误如下：`);
    console.log(e);
  } finally {
  }
})();

function sitemapXmlReader(file) {
  let data = fs.readFileSync(file, 'utf8');
  let sitemap = xmlParser(data);
  return sitemap.root.children.map(function (url) {
    let loc = url.children.filter(function (item) {
      return item.name === 'loc';
    })[0];
    return loc.content;
  });
}

function removeProtocol(url) {
  return url.substr(url.indexOf(':'));
}

function send(options) {
  return new Promise(function (resolve, reject) {
    request(options, function (error, response, body) {
      if (!error) {
        resolve(body);
      } else {
        reject(error);
      }
    });
  });
}
```

- 修改代码中 `config` 部分 `username`、 `token`、` repo` (存放 `issues` 的仓库名称)

#### 脚本执行

- 执行下面的命令，就可以部署站点，并初始化所有的评论了。

```bash
hexo clean
hexo generate
hexo deploy
node ./comment.js
```

- 也可以通过在站点根目录的 `package.json` 文件中，新建 `npm` 脚本

```json
"scripts": {
    "talk": "hexo clean && hexo generate && hexo deploy && node ./comment.js"
}
```

- 执行 `yarn talk` 即可一键完成所有操作

#### 注意事项

​ 第一步中的 `sitemap` 插件会生成的 `sitemap.xml` 会包含**全部的界面**，包括标签页、关于页等，执行上面的代码也会对这些页面生成评论框(也就是 `issue`) ，我在原作者的基础上添加了文章的过滤逻辑：

- 过滤 `sitemap` 中非文章的链接

```javascript
// comment.js 42 行
const oUrls = sitemapXmlReader(config.sitemapUrl);
// oUrls 即为分析出的网站链接 字符串数组 --- 添加自身的过滤逻辑
// 我这里，访问每个文章时，链接上都有时间信息，即 2021/xx/xx 按照时间格式过滤掉非md文档的链接
const urls = oUrls.filter((l) => /\/(\d{4})\/(\d{2})\/(\d{2})\//.test(l));
```

- `id` 规则不一样导致的 `issues-lable` 与文章不匹配

```javascript
// 在生成 gitalk时，每一篇文章有一个独立的 id， 规则是不同主题自己定的 即： new Gitalk 传入的 id 规则
// 调整 comment.js 66 行(3-hexo 主题 文章id 通过 decodeURI解过码)
```

```diff
- let pathLabel = url.parse(item).path;
+ let pathLabel = decodeURI(url.parse(item).pathname);
```

### 参考

[`nodejs` 版本的 `Gitalk/Gitment` 评论自动初始化](https://daihaoxin.github.io/post/322747ae.html)
