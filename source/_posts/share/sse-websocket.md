---
title: SSE 和 WebSocket 通信以及它们之间的区别
date: 2023-04-11 10:36:07
updated: 2023-04-11 10:36:07
tags:
  - 网络通信
  - SSE
  - WebSocket
categories:
  - 技术分享
  - 前后端交互
---

### `What`？

`SSE`又称`Server-Sent-Events`，是一种基于 `HTTP` 的单向通信协议，允许服务器向客户端发送异步事件流。它是 `HTML5` 规范的一部分，通过常规的 `HTTP` 连接（使用"长轮询"技术）从服务器向客户端推送数据。`SSE` 不需要额外的协议切换，因为它构建在现有的 `HTTP` 协议之上。

`WebSocket` 是一种全双工通信协议，允许在单个 `TCP` 连接上进行双向通信。它提供了一个持久化的连接，可以在客户端和服务器之间实时地双向传输数据，而不需要像 `HTTP` 请求那样频繁地创建和关闭连接。

<!-- more -->

### `SSE`

#### 工作原理

- 客户端通过简单的 `EventSource API` 连接到服务器。

- 一旦连接建立，服务器可以在任何时候异步地向客户端发送事件数据。

- 客户端通过监听 `onmessage` 事件来接收来自服务器的消息。

#### 优点

- 简单易用，不需要特殊的协议或库。

- 服务器向客户端推送数据时比传统轮询（`polling`）更高效。

#### 缺点

- 只能由服务器单向向客户端发送数据，客户端无法直接向服务器发送消息。

- 支持度较低：虽然主流浏览器都支持 `SSE`，但在一些旧版本或特定情况下可能不兼容。

#### `API`

##### `EventSource` 实例对象

`client` 生成实例对象，建立 `SSE` 连接

```ts
const eventSource = new EventSource('https://example.com/events');

// 表示当前实例是否开启 CORS 的 withCredentials
const source = new EventSource(url, { withCredentials: true });
```

##### `EventSource` 实例对象属性

###### `readyState`

表明连接的当前状态。该属性只读，可以取以下值：

- `0`：相当于常量 `EventSource.CONNECTING`，表示连接还未建立，或者断线正在重连。

- `1`：相当于常量 `EventSource.OPEN`，表示连接已经建立，可以接受数据。

- `2`：相当于常量 `EventSource.CLOSED`，表示连接已断，且不会重连。

###### `onopen`

连接一旦建立，就会触发 `open` 事件，可以在 `onopen` 属性定义回调函数。

```ts
source.onopen = (e) => {};
// 另一种写法
source.addEventListener('open', (e) => {});
```

###### `onerror`

如果发生通信错误（比如连接中断），就会触发 `error` 事件，可以在 `onerror` 属性定义回调函数。用法同上。

###### `onmessage`

客户端收到服务器发来的数据，就会触发 `message` 事件，可以在 `onmessage` 属性定义回调函数。

```ts
source.onmessage = (event) => {
  const data = event.data;
  const origin = event.origin;
  const lastEventId = event.lastEventId;
  // handle message
};
// 另一种写法
source.addEventListener(
  'message',
  (event) => {
    const data = event.data;
    const origin = event.origin;
    const lastEventId = event.lastEventId;
    // handle message
  },
  false,
);
```

##### `EventSource` 实例对象方法 `close`

`close` 方法用于关闭 `SSE` 连接。

```ts
source.close();
```

除此之外，还支持发送自定义事件...

### `WebSocket`

#### 工作原理

- 客户端通过 `WebSocket API` 与服务器建立 `WebSocket` 连接。

- 一旦连接建立，客户端和服务器可以在任何时候通过发送消息进行双向通信。

- 连接保持打开状态，直到其中一方显式关闭连接。

#### 优点

- 实时性和效率高：由于 `WebSocket` 建立一次连接后就保持打开状态，可以实时地在客户端和服务器之间传递数据，减少了额外的连接建立和 `HTTP` 头信息的开销。

- 双向通信：服务器和客户端之间可以实现双向实时通信。

#### 缺点

- 实现复杂一些：相对于 `SSE，WebSocket` 的实现可能需要一些额外的编程工作。

- 兼容性问题：虽然现代浏览器普遍支持 `WebSocket`，但在一些特殊情况下，可能需要处理一些兼容性问题。

#### `API`

##### `WebSocket` 实例对象

`WebSocket` 对象作为一个构造函数，用于新建 `WebSocket` 实例。

```ts
// 执行语句之后，客户端就会与服务器进行连接。
const ws = new WebSocket('ws://localhost:8080');
```

#####

属性

###### `readyState`

属性返回实例对象的当前状态，共有四种。

- 0：`CONNECTING`，表示正在连接。

- 1：`OPEN`，表示连接成功，可以通信了。

- 2：`CLOSING`，表示连接正在关闭。

- 3：`CLOSED`，表示连接已经关闭，或者打开连接失败。

###### `onopen` 和 `onclose`、 `onerror`

指定连接成功后和连接关闭后、报错时的回调函数。用法同上

###### `onmessage`

用于指定收到服务器数据后的回调函数

```ts
ws.onmessage = (event) => {
  const data = event.data;
  // 处理数据
};
ws.addEventListener('message', (event) => {
  const data = event.data;
  // 处理数据
});
```

服务器数据可能是文本，也可能是二进制数据（`blob` 对象或 `arraybuffer` 对象）

```ts
// 收到的是 blob 数据
ws.binaryType = 'blob';
ws.onmessage = (e) => {
  console.log(e.data.size);
};
// 收到的是 ArrayBuffer 数据
ws.binaryType = 'arraybuffer';
ws.onmessage = (e) => {
  console.log(e.data.byteLength);
};
```

###### `bufferedAmount`

表示还有多少字节的二进制数据没有发送出去。它可以用来判断发送是否结束。

```ts
const data = new ArrayBuffer(10000000);
socket.send(data);
if (socket.bufferedAmount === 0) {
  // 发送完毕
} else {
  // 发送还没结束
}
```

###### `WebSocket` 实例对象方法 `send`

用于向服务器发送数据。

```ts
// 发送文本
ws.send('your message');

// 发送 Blob
const file = document.querySelector('input[type="file"]').files[0];
ws.send(file);

// 发送 ArrayBuffer
const img = canvas_context.getImageData(0, 0, 400, 320);
const binary = new Uint8Array(img.data.length);
for (let i = 0; i < img.data.length; i++) {
  binary[i] = img.data[i];
}
ws.send(binary.buffer);
```

### `SSE` vs `WebSocket` 区别?

- `SSE` 使用 `HTTP` 协议，现有的服务器软件都支持。`WebSocket` 是一个独立协议。

- `SSE` 是单工通信，只能服务器向浏览器推送，`WebSocket` 是双向通信。

- `SSE` 属于轻量级，使用简单；`WebSocket` 协议相对复杂。

- `SSE` 默认支持断线重连，`WebSocket` 需要自己实现。

- `SSE` 一般只用来传送文本，二进制数据需要编码后传送，`WebSocket` 默认支持传送二进制数据。

- `SSE` 支持自定义发送的消息类型。

### 使用示例

#### `SSE` 优化登录状态心跳检测

`client` 实现：

```ts
function checkingLogin() {
  sseChecker.value?.close();

  sseChecker.value = new EventSource(`/authorize/session-auth-status`);
  sseChecker.value.addEventListener(`message`, async (ev: MessageEvent<string>) => {
    const data = JSON.parse(ev.data) as SessionStatusResponse;

    if (data.status === 401) {
      console.error('Detected session token expired');
      sseChecker.value?.close();
      unauthorized.value = true;

      await MessageBox.confirm('当前账号已在别处登录，是否重新登录？', '提示', {
        showClose: false,
        showCancelButton: false,
        cancelButtonText: '退出',
        type: 'warning',
      });

      location.href = '/authorize/logout';
    }
  });
}
```

基于 `nestJs` 的 `server` 实现：

```ts
import Axios from 'axios';
import { Observable } from 'rxjs';
import { Sse } from '@nestjs/common';

@Sse('session-auth-status')
  public checkSessionTokenStatus(
    @Req() req: Request,
    @Res() res: Response,
  ): Observable<Partial<MessageEvent>> {
    // 出于http连接问题， sse连接在5分钟后自动断开
    const maxCheckDuration = 1000 * 60 * 5;
    // 每两秒检查一次
    const checkInterval = 2000;

    let checkedCount = 0;
    let needDispose = false;
    return new Observable<Partial<MessageEvent>>((sub) => {
      const doCheck = async () => {
        try {
          while (!needDispose) {
            checkedCount++;
            await new Promise<void>((resolve, reject) => {
              req.session.reload((err) => {
                if (err) return reject(err);
                resolve();
              });
            });

            const token = await getAccessToken(req.session);
            if (!token) {
              throw new UnauthorizedException();
            }

            const res = await new AuthorizeService(req).getMeInfo(token, false);
            sub.next({
              data: {
                message: 'OK',
                status: 200,
                payload: res.data,
              },
              // 控制前端断开后重连间隔时机
              retry: 500,
            });
            await delayMs(checkInterval);
          }
        } catch (err) {
          let message = 'failed to do check';
          if (Axios.isAxiosError(err)) {
            message = 'Token is expired';
          } else if (err instanceof UnauthorizedException) {
            message = `Got empty token`;
          }

          sub.next({
            data: {
              message,
              status: 401,
            },
            retry: Number.MAX_SAFE_INTEGER,~
          });

          logger.error('failed to do check %o', err);
          needDispose = true;
        } finally {
          if (needDispose || checkedCount * checkInterval >= maxCheckDuration) {
            sub.complete();
            res.end();
          }
        }
      };

      doCheck();
      return () => {
        needDispose = true;
      };
    });
  }
```

### 问题记录

#### 连接数限制

`SSE` 基于常规的 `HTTP` 连接，受制于浏览器对同一域名下的并发连接数限制，通常为 6-8。**错误的使用 sse，不及时关闭造成连接数超出限制时，浏览器页面会卡顿异常**

`WebSocket` 建立持久化的全双工连接，通常不受 HTTP 连接数限制，但服务器和操作系统可能有自己的并发连接数限制。

> 更详细的 `API` 使用教程参考：[EventSource](https://developer.mozilla.org/zh-CN/docs/Web/API/EventSource) 、 [WebSocket](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket)
