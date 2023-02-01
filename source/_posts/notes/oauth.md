---
title: OAuth 认证的几种实现方式
date: 2023-02-01 15:37:37
updated: 2023-02-01 15:37:37
tags:
  - oauth
categories:
  - 土豆の学习笔记
  - 大前端
---

### Before...

> `OAuth 2.0` 通过建立授权机制，让第三方应用可以获取用户或者系统数据。

前后端在进行通信(或者称之为数据请求)时，都需要前端在请求时携带用户 `token`，用来验证用户的登录情况以减轻服务器压力。而 `OAuth` 的核心作用就是向客户端颁发令牌(`token`)。

通常所说的令牌`token`，分为两种：客户端令牌(任何时候都能访问资源的凭证，通常由 `client_id(身份id)/client_secret(密钥：只能保存到后端)` 生成)、用户令牌(完成登录后生成)。

<!-- more -->

### 授权方式

#### 授权码

> 适用于有中间层的 WEB 项目,并将生成的 token 存到 node 后端，避免令牌泄露。这种方式安全性最高，也是我最近项目用到的鉴权方式。

- 步骤一：网站`A(client)`点击登录(或者其他链接)，跳转至网站`B(JAVA或者其他后端)：https://b.com/oauth/authorize?query`,`query`参数如下：

  ```json
  {
    "response_type": "code", // 表示要求返回授权码
    "client_id": "", // 应用标识 告诉 B 是哪个站点应用在请求
    "redirect_uri": "" // 参数表示 B 完成完成授权过程后重定向的链接地址
    // "scope": ""  // 参数表示要求的授权范围，可不设
  }
  ```

  - 登录逻辑放到了前端`A`，那么由前端`A`完成账号密码录入，发起登录请求`https://b.com/oauth/login(提供登录成功后的重定向地址 successRedirect... 注意区分 redirect_uri)`，由网站`B`重定向到`A(node)：/a.com/oauth/successRedirect(中间层获取授权码)`，由`A(node: /successRedirect)`拿到授权码进行重定向`(redirect_uri)`...

  - 登录逻辑放到了后端`B`，那么在点击了`A`提供的链接后，会在`B`网站登录页完成登录并提示接受`A`授权，完成登录过程,并生成授权码，然后重定向...

  基于两种不同场景下的登录，最终都将生成授权码，并作为`query`携带，重定向回`A(node的一个接口，eg. https://a.com/SSOCheckUser?code=xxx)`

- 步骤二：网站`A(node)`拿到授权码(`code`)后，就可以向`B`请求令牌 `https://b.com/oauth/token?query`，`query`参数如下：

  ```json
  {
    "client_id": "CLIENT_ID", // 应用标识
    "client_secret": "CLIENT_SECRET", // 密钥，只能存储在 A(node) 层
    // B 通过 A 提供 client_id client_secret 来判定 A 的身份 (身份识别码，这个通常是会备案到网站 B)
    "grant_type": "authorization_code", // 标识授权方式是授权码
    "code": "xxx" // 第一步拿到的授权码
    // "redirect_uri": "WEB_REDIRECT_URL" // 可以是 A(node) 的一个配置，比如跳转到 A(client) 的首页
  }
  ```

  该请求会返回令牌信息：包含`access_token(令牌)/refresh_token(用于更新令牌)/expires_in(过期时间)...`

  同时，你可以通过拿到的令牌信息获取当前登录用户的详细个人信息，并将用户信息及`refresh_token`存储到`node:session`。~~还可以把用户 access_token 存到 redis，key 即为 sessionId~~

  最终重定向回`A(client)`配置的一个地址。

  前端访问后端服务时，通过中间层代理，追加`token`信息到`header`中，如果`token`过期，则通过`refresh_token`更新令牌，如果登录过期，则重定向到登录地址。

#### 凭证式

> 适用于有后端(`node`)的`web`应用，在中间层通过客户端令牌完成一些需要登录（但并没有登录）才能获取资源的请求。

> 同时适用于命令行/`postman`接口测试使用。

这种方式获取的令牌并不是针对用户使用的，因为生成的令牌可以多个用户共享。~~当然后端可以通过判断 session 的用户信息，标记当前使用客户端令牌的资源请求范围~~

```javascript
const info = await axios.get(`https://b.com/oauth/token`, {
  params: {
    grant_type: 'client_credentials', // 采用凭证式
    client_id: CLIENT_ID, // 应用标识
    client_secret: CLIENT_SECRET, // 密钥
  },
});
```

#### 密码式

> 适用范围小。为了确保安全性，一般是在中间层使用，或者在接口调试的时候使用

这种方式需要用户给出自己的账号密码，风险大，需要高度信任网站`B`，不建议使用。获取方式：

```javascript
const info = await axios.get(`https://b.com/oauth/token`, {
  params: {
    grant_type: 'password', // 表示密码式
    username: USERNAME, // 账号
    password: PASSWORD, // 密码
    client_id: CLIENT_ID, // 应用标识
  },
});
```

该方式是将令牌放到 `response` 中作为 `json` 数据进行返回的，不进行重定向。

#### 隐藏式

> 适用于前后端分离，且没有`node`中间层服务的`web`应用。

```javascript
const info = await axios.get(`https://b.com/oauth/authorize`, {
  params: {
    response_type: 'token', // 表示要求直接返回令牌
    client_id: CLIENT_ID, // 应用标识
    redirect_uri: REDIRECT_URL, // 完成认证后的重定向地址
  },
});
```

这种方式虽然使用面最广，但是也很不安全，因为`token`是直接传递给前端的，只适合安全要求不高的场景，同时有效期通常为会话期间(`session`).
