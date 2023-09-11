---
title: 网络请求之响应异常处理拦截器
date: 2023-09-05 15:00:17
updated: 2023-09-05 15:00:17
tags:
  - Axios
  - UmiRequest
  - BlobError
  - RequestRetry
  - Interceptors
categories:
  - 技术分享
  - 网络请求
---

### 背景前提

不管是使用 `axios` 还是类似 `umi-request` 进行网络请求，在实际开发中，我们都需要对请求异常进行处理，`eg.` **请求超时**、**网络错误**等。

针对不同类型的异常，可以采取不同的措施，`eg.` **弹出消息提示**、**跳转到错误页面**、**重新请求**等。

<!-- more -->

### 处理异常

针对 `axios` 的 `Interceptors`，我们可以通过定义并导出自定义的 `axios` 实例，并在实例中配置 `Interceptors.response` 来实现。

然后在项目中，使用导出的 `axios` 进行网络请求，即可实现全局对请求异常的统一处理。

#### `axios.interceptors.response`

```ts
import axios from 'axios';
import { Message } from 'element-ui';

axios.defaults.baseURL = '/';
// axios other configs...

axios.interceptors.response.use(
  (res) => res,
  (error) => {
    // 被取消的请求不弹出message. 一般发生在页面跳转时
    if (axios.isCancel(error)) {
      return Promise.reject(error);
    }

    // 处理未登录 401 问题，跳转到登录页
    if (
      (error?.response?.status === 400 &&
        error?.response?.data?.error?.includes('invalid_token')) ||
      error?.response?.status === 401
    ) {
      location.href = '/login';
    } else {
      const message = error?.response?.data?.message || '网络请求异常';

      Message.error({ message, duration: 2000, offset: 70 });
    }

    return Promise.reject(error);
  },
);

export default axios;
```

以上就是比较通用一点的拦截处理。

### 拦截器优化

在实际使用中，我们可能需要对部分定制的请求有单独的处理，以下是列出的可能需要优化的点：

- 基于 `blob` 类型请求的错误处理

- 有多条错误消息时，提示期间只显示一条

- 不显示异常消息，或者不做任何拦截处理

- 请求超时重试

- ...

#### 扩展请求配置

可以在 `axios` 请求函数里添加单独的 `config` 配置，在 `error.config` 中拿到请求时传入的参数，实现特殊处理：

`axios-extend.d.ts`：

```ts
import { AxiosRequestConfig } from 'axios';

declare module 'axios' {
  export interface AxiosRequestConfig {
    metadata?: {
      start: number;
    };

    /**
     *  用于控制该请求出现错误时， 是否由默认message显示错误信息
     *
     * @type {boolean}
     * @memberof AxiosRequestConfig
     */
    suppressError?: boolean;

    /**
     * 用于请求在401状态码时， 是否自动重定向到登陆页。
     *
     * @type {boolean}
     * @memberof AxiosRequestConfig
     */
    suppressUnauthRedirect?: boolean;
  }
}
```

`request.ts`：

```ts
import axios from '@/plugin/axios';

function checkAppTemplate(code: string) {
  return axios.get(`/xxx/${code}`, { suppressError: true });
}
```

从上面可以看到，请求的时候 `axios` 的 `config` 配置里，添加了 `suppressError`参数，在 `Interceptors` 中只需要判断是否含有该参数, 如果为 `true`，则不提示 `message`，否则提示，`suppressUnauthRedirect` 同理。

上面的 `axios.ts` 做以下调整：

```ts
if (!error?.config.suppressError) {
  const message = error?.response?.data?.message || '网络请求异常';

  Message.error({ message, duration: 2000, offset: 70 });
}
```

#### 基于 `blob` 的请求错误消息处理

前端在下载文件时，通常会基于 `blob` 的方式，当我们设置了 `responseType` 为 `blob`的时候，如果接口抛出异常并返回了例如下面的错误消息，由于返回的是 `Blob` 对象，所以无法同其他请求一样去解析 `message`：

```json
{
  "code": 500,
  "message": "下载异常",
  "data": null
}
```

为了能够解析返回的 `message`，需要扩展上面提到的通用的 `interceptors`:

```ts
let message = error?.response?.data?.message || '网络请求异常';

if (error.response?.data instanceof Blob && error.response?.data.type === 'application/json') {
  const resText = await error.response.data.text();
  message = JSON.parse(resText).message || message; // 下载异常
}

Message.error({ message, duration: 2000, offset: 70 });
```

#### `message` 防抖处理

通过外部变量来标识是否正处于 `message` 提示中：

```ts
let isErrorShowing = false;

// axios.interceptors.response.use(....

if (!isErrorShowing) {
  const message = error?.response?.data?.message || '网络请求异常';

  Message.error({ message, duration: 2000, offset: 70, onClose: () => (isErrorShowing = false) });
}
```

#### 请求超时重试

##### 基于 `axios` 请求重试

可以参考 [axios-retry](https://github.com/softonic/axios-retry) 实现。

##### 基于 `umi-request` 的请求重试

借鉴了 `axios-retry`，具体实现如下：

`umiRequestConfig.ts`：

```ts
import type { RequestConfig } from 'umi';
import type { RequestOptionsInit } from 'umi-request';

export interface RequestOptions extends RequestOptionsInit {
  retryConfig?: { retryCount?: number; retryCondition?: (res: Response) => boolean };
}

export interface CRequestConfig extends RequestConfig {
  requestInterceptors?: ((
    url: string,
    options: RequestOptions,
  ) => {
    url?: string;
    options?: RequestOptions;
  })[];
  responseInterceptors?: ((
    response: Response,
    options: RequestOptions,
  ) => Response | Promise<Response>)[];
}
```

`app.tsx`:

```ts
// 超时重试的最大次数
const TIMEOUT_MAX = 3;

// 超时重试延迟
const TIMEOUT_RETRY_DELAY = 50;

function shouldRetry(res: Response, retryCount: number, retryCondition?: (r: Response) => boolean) {
  return (res.status === 504 && retryCount < TIMEOUT_MAX) || retryCondition?.(res);
}

function requestRetry(
  res: Response,
  { prefix, retryConfig, ...retryOption }: RequestOptions, // 移除 prefix，避免 retry 的时候重复添加
): Response | Promise<Response> {
  const retryCount = retryConfig?.retryCount ?? 0;

  if (requestHelper.shouldRetry(res, retryCount, retryConfig?.retryCondition)) {
    return new Promise((resolve) =>
      setTimeout(() => {
        resolve(
          umiRequest(new URL(res.url).pathname, {
            ...retryOption,
            retryConfig: { ...(retryConfig ?? {}), retryCount: retryCount + 1 },
          } as RequestOptions),
        );
      }, TIMEOUT_RETRY_DELAY),
    );
  }

  return res;
}

export const request: CRequestConfig = {
  // errorHandler:
  // requestInterceptors: []
  responseInterceptors: [requestRetry],
};
```
