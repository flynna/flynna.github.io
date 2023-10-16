---
title: 基于 zip-stream 实现批量文件的打包压缩下载
date: 2023-10-13 16:24:30
updated: 2023-10-13 16:24:30
tags:
  - archiver
  - zipStream
categories:
  - 技术分享
  - 功能实现
---

之前的一个项目需求：“用户从列表中勾选一些数据，这些数据包含了文件信息。需要下载所勾选数据中的所有文件”

直接通过传统方式 `location.href 或者 a标签` 下载，会发现并不能并行下载，最多只能同时下载一个文件...

然后尝试通过基于 `iframe` 并行下载，但会在浏览器上产生很多的下载任务，生成一串的文件在浏览器底部显示，看着都很不科学...

<div class="info">

> 为什么是基于 `iframe` 而不是 `a` 链接下载？
>
> 受浏览器安全策略影响，浏览器会阻止超过一个下载任务的请求，如果存在多个，会取消掉前面的下载任务。
>
> 除了使用 `iframe` 以外，还可以通过 `setTimeout` 来规避，（但需要知道上一个任务的下载是否完成，且理论上也并不是并行下载）

</div>

最后想了下还是基于 `node` 在中间层封装一个 `zip` 的下载接口，将这一批文件统一打包处理再供给前端下载。

<!-- more -->

### 客户端实现

#### 基于 `POST` 下载

通常我们上传文件数据后，都是将文件可访问路径作为值存在表单里，当我们需要批量下载时（前提是后端并没有实现这个接口，需要我们在中间层自己封装实现），需要将这些路径拼接到 `query` 中，很容易就会导致路径过长超出浏览器限制。此时就需要我们通过 `POST` 方式进行下载。

前端只需要将返回的文件流通过 `blob` 的方式下载到客户端即可。

```ts
function download(file: Blob, name: string) {
  const link = document.createElement('a');
  link.href = URL.createObjectURL(file);
  link.download = name;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
  URL.revokeObjectURL(link.href);
}

async function downloadFiles(files: Array<{ name: string; path: string }>) {
  const zipRes = await request(`/downloadFilesZip`, {
    method: 'POST',
    data: { list: files },
    responseType: 'blob',
  });

  const blob = new Blob([zipStreamRes], { type: 'application/octet-stream' });

  download(blob, '附件资源.zip');
}
```

#### 基于 `GET` 下载（推荐）

为了规避上面提到的 `GET` 链接超长的问题，可以先调用 `POST` 将需要下载的文件任务存在 `session` 中，然后再调用 `GET` 执行下载。

<div class="info">

> 为什么前面提到了 `GET` 的劣势，我们仍然需要使用 `GET` 进行下载？
>
> `POST` 请求下载通常是需要前端生成 `Blob` 文件对象，而这又是直接存储在内存当中的，意味着当文件过大的时候，可能会出现下载异常。

</div>

改造一下前面的 `POST` 下载：

```ts
async function downloadFiles(files: Array<{ name: string; path: string }>) {
  // 调用接口，只是将 files 数据存到 session，而非真正意义上的下载
  await request(`/setupDownloadFilesZip`, { method: 'POST', data: { list: files } });

  // 中间层提供的 GET /downloadFilesZip 接口下载
  location.href = `/downloadFilesZip`;
}
```

### 服务端实现

服务端实现相对 `client` 来说稍微复杂一点。需要将客户端提供的文件链接在服务端请求下载并做压缩转发。

```ts
type File = { name: string; path: string };

export interface ISession extends Express.Session {
  pendingDownloadTasks: Array<{
    name: string;
    path: string;
  }>;
}

@Post('/downloadFilesZip')
public async setFilesZip(
  @Session() session: ISession,
  @Body('list') pendingList: File[],
) {
  session.pendingDownloadTasks = pendingList;
}

@Get('/downloadFilesZip')
public async downloadFilesZip(
  @Session() session: ISession,
  @Res() res: Response,
) {
  // download...
}
```

同 `client` 实现思路，先将需要打包压缩的文件信息存到 `session`，然后在 `Get` 请求中执行下载。下面是 `downloadFilesZip` 函数的两种实现

#### 基于 `archiver` 实现

> `archiver` 是一个流式的、高级别的打包工具，支持创建各种压缩格式，如 `ZIP、TAR、TAR.GZ`。适用于大型和复杂的打包任务。
>
> `p-limit` 在异步任务中控制并发操作数量，避免资源的过渡消耗。

```ts
import Archiver from 'archiver';
import pLimit from 'p-limit';

async function zipFile(zipArchive: Archiver.Archiver, file: File) {
  const fileStream = await axios.get(file.path, { responseType: 'stream' });

  return new Promise<void>((resolve, reject) => {
    zipArchive.append(fileStream, { name: file.name });
    fileStream.on('end', () => {
      console.info(`file ${file.name} download success`);
      resolve();
    });
    fileStream.on('error', (err) => {
      reject(err);
    });
  });
}

async function downloadFilesZip(@Session() session: ISession, @Res() res: Response) {
  const fileList = session.pendingDownloadTasks;

  session.pendingDownloadTasks = [];

  // 下载后立即清除session中保存的下载任务
  await new Promise<void>((resolve) => {
    session.save(() => resolve());
  });

  if (fileList && fileList.length === 0) {
    throw new BadRequestException('当前会话没有待打包下载文件数据');
  }

  // 设置压缩等级
  const zipArchive = Archiver('zip', { zlib: { level: 9 } });

  // 设置响应头
  res.contentType('zip');
  res.attachment(encodeURIComponent('附件资源.zip'));

  // 通过管道响应流式数据，中间层只做流转发不做存储
  zipArchive.pipe(res);

  try {
    // 控制并发数2
    const limit = pLimit(2);
    const tasks = fileList.map((file) => limit(() => this.zipFile(zipArchive, file)));

    await Promise.all(tasks);

    zipArchive.finalize();
  } catch {
    console.error(`download error: ${(error as Error)?.message}`);
    throw new HttpException((error as Error)?.message, 500);
  }
}
```

#### 基于 `zip-stream` 实现（推荐）

> `zip-stream` 是一个轻量级的、流式的 `zip` 压缩工具，专注于 `zip` 格式。相较于前者更轻量，在本例场景中更加适用。
>
> `promisify` 是一个 `node` 内置的 `util` 模块函数，用于将回调函数转换为 `promise` 对象。

```ts
import { promisify } from 'util';
import ZipStream from 'zip-stream';

async function downloadFilesZip(@Session() session: ISession, @Res() res: Response) {
  const fileList = session.pendingDownloadTasks;

  session.pendingDownloadTasks = [];

  // 下载后立即清除session中保存的下载任务
  await new Promise<void>((resolve) => {
    session.save(() => resolve());
  });

  if (fileList && fileList.length === 0) {
    throw new BadRequestException('当前会话没有待打包下载文件数据');
  }

  const zipArchive = new ZipStream({ zlib: { level: 9 } });

  res.contentType('zip');
  res.attachment(encodeURIComponent('附件资源列表.zip'));

  zipArchive.pipe(res);

  const addEntry = promisify(zipArchive.entry).bind(zipArchive);
  const abortController = new AbortController();

  // 客户端断开时，停止下载
  res.on('close', () => {
    const aborted = !res.writableFinished;

    if (aborted) {
      console.warn(`client disconnected`);
      abortController.abort();
      zipArchive.destroy();

      if (!res.headersSent) {
        res.status(400).send({ message: 'Client disconnect' });
      }
    }
  });

  for (const file of fileList) {
    try {
      console.info(`start download file ${file.name} download`);

      const fileStream = await axios.get(file.path, {
        responseType: 'stream',
        signal: abortController.signal,
      });

      await addEntry(fileStream, {
        name: file.name,
      });

      console.info(`complete download file ${file.name} download`);
    } catch (err) {
      if (axios.isCancel(err)) {
        break;
      } else {
        return console.info(`failed to add entry`);
      }
    }
  }

  zipArchive.finalize();
}
```
