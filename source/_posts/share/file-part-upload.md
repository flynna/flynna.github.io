---
title: 大文件的分段并发上传实现
date: 2023-06-08 11:53:51
updated: 2023-06-08 14:53:51
tags:
  - 文件上传
  - 并发请求
categories:
  - 技术分享
  - 功能实现
---

_在开发过程中，难免会遇到文件上传这个功能。有些服务器会限制单个文件的存储大小，那么用户上传超大文件怎么处理？_

<!-- more -->

### 文件为什么需要分段？

- 上传速度：将一个大文件分割成多个小文件块后，每个文件块的大小相对较小，可以大大提高文件的上传速度。

- 上传可靠性：大大降低了由于网络不稳定或者上传中断情况出现的概率。分段后即使上传失败了，也只需要重新上传失败的文件块即可。

- 服务器限制：有些服务器可能会限制单个文件的上传大小，如果上传文件过大，可能会造成上传异常。

- 便于管理：将文件分割成多个小文件块，可以更方便地管理文件，对文件块单独操作。

### 文件分段处理

- 获取文件对象 `File`

- 利用 `Blob.slice` 方法分割文件：使用 `Blob` 对象的 `slice` 方法将文件分割成多个文件块。

  实现如下：

  ```typescript
  // 获取文件对象
  function getFileById(id: string) {
    return document.getElementById(id)?.files?.[0];
  }

  /**
   * 获取文件分片 slice 信息
   * @param fileSize 文件大小
   * @param chunkSize chunk 大小，默认每片 5 MB
   */
  function getFileSliceRange(fileSize: number, chunkSize = 1024 * 1024 * 5) {
    const chunksRange: Array<{ start: number; end: number }> = [];
    const chunksLen = Math.ceil(fileSize / chunkSize);

    for (let i = 0; i < chunksLen; i++) {
      const start = i * chunkSize;
      const end = Math.min(start + chunkSize, total);
      chunksRange.push({ start, end });
    }

    return chunksRange;
  }

  // 文件碎块： file.slice(chunksRange[index].start, chunksRange[index].end);
  ```

有的同学应该注意到了，我这里并没有直接对文件进行分割，然后将文件碎块用变量存入内存当中，而仅仅只是保存了每个文件碎块对应的位置信息。（**在实际取用的时候，通过`file.slice(start, end)`取用。避免直接一次性对文件切割再放入内存，后者是对整个文件的一份拷贝，如果文件过大，会加大浏览器内存损耗**）

_ok，现在文件确实是已经做到分片了，解决了大文件没办法上传的问题，那如果文件碎片很多，又该怎么提升上传效率，实现优化呢？_

### 分段上传并发控制

目前的一些主流的浏览器，都有默认的并发请求限制，换句话说，即时我们在代码中对请求并发数不做限制，实际上传时，仍然也不会同时发起过多的请求。

> `Chrome/Firefox/Safari` 同域名下默认的并发数为 `6`，`IE` 同域名下默认的并发数为 `2`。那么如何修改这些限制呢？

> `Chrome` 可以通过修改网络参数实现修改，`IE` 需要修改注册表。`Safari` 则是在 `HTTP/2` 不做限制。

那么，浏览器限制了我们真的不用在代码中限制并发数了吗？`no...no...no`，为什么仍然需要限制呢？

**避免将网络请求的并发限制全交给浏览器，造成在文件上传过程中，界面的其他操作请求被阻塞，直到文件碎块上传结束才被执行。同时，代码中的限制数，也应该尽量小于浏览器的默认并发数量，一般取 `2` 或者 `3`。**

如何简洁优雅的实现文件分段的并发控制呢？

```typescript
/**
 * 文件块上传
 * @param file 文件
 * @param preSignQuery 文件块预签名上传的请求 query
 * @param [onUploadProgress] 自定义上传进度
 */
function partUploadHandler(
  file: File | Blob,
  preSignQuery: { uploadId: string; objectName: string; partNumber: number },
  onUploadProgress?: (e: ProgressEvent) => void,
) {
  // 获取文件上传的预签名地址 preSignProvider --- 提供接口
  const url = await preSignProvider(preSignQuery);
  const bucket = new URLSearchParams(url).get('bucket');

  // 文件上传到对象存储服务器
  await axios.put(`${window.location.origin}/proxyServicePrefix/s3${url}`, file, {
    headers: { 'Content-Type': file.type },
    onUploadProgress: onUploadProgress,
  });

  return `${url.split('?')[0]}?bucket=${bucket}`;
}

/**
 * 文件分片并发上传处理
 * @param file 需要分片上传的源文件
 * @param preSignQuery 文件块预签名上传的请求 query
 * @param [chunkSize] 切片大小，用于分片、计算已上传进度，Minio有分页时每一片大小至少为 5m 的限制
 * @param [limit] 并发数，默认是 2
 */
function uploadParallel(
  file: File,
  preSignQuery: { uploadId: string; objectName: string },
  chunkSize = 1024 * 1024 * 5,
  limit = 2,
) {
  const { uid, type, size } = file; // uid 非 File 对象提供，可以理解为文件的 key

  const reqPromise: Promise<string>[] = []; // 记录分段后的全部请求结果
  const pendingReq: Promise<void>[] = []; // 目前的请求队列，用于并发限制

  const chunksLoaded: number[] = []; // 已经上传完成的数据流，用于计算上传进度并展示

  const chunksRange = this.getFileSliceRange(size, chunkSize); // 获取分段信息

  for (const [idx, { start, end }] of chunksRange.entries()) {
    // 创建文件块的请求 promise
    const p = partUploadHandler(
      file.slice(start, end),
      {
        ...preSignQuery,
        partNumber: idx + 1, // 记录当前是上传的第几片
      },
      (e) => {
        chunksLoaded[idx] = e.loaded;

        const loaded = chunksLoaded.reduce((sum, c) => sum + c, 0);
        const percentage = (loaded / size) * 100;

        // 找到当前正在上传的文件，给它绑定自定义的进度条信息
        const currentFile = this.uploadingFiles.find((f) => f.uid === uid);

        // percentage 100，并非真的完成了整个上传过程了，仍需后续的完整性校验通过
        if (currentFile && percentage !== 100) {
          currentFile.percentage = Number(percentage.toFixed(2)) || 0;
        }
      },
    );

    reqPromise.push(p);
    // 当前 p 上传结束后，从 pendingReq 队列中移除
    pendingReq.push(p.then(() => pendingReq.splice(pendingReq.indexOf(next), 1)));

    if (pendingReq.length >= limit) {
      // 当达到并发数限制时，等待前面一个文件碎块上传完成，才继续后续的分段
      await Promise.race(pendingReq);
    }
  }

  // 记录第一个地址（都是一样的，详见 partUploadHandler）
  return Promise.all(reqPromise)[0];
}

// main 函数. eg. element/antDesign 拿到的自定义上传对象
function httpRequestMethod(request: HttpRequestOptions) {
  const file = res.file;

  // 记录当前正在上传的文件信息
  this.uploadingFiles.push({ uid: file.uid, status: 'uploading', percentage: 0 });

  try {
    // 获取分段上传 id、和 objectName (提供 multipartUploadId 接口)
    const { uploadId, objectName } = await multipartUploadId(file.name);

    // 开始分片上传文件
    const path = uploadParallel(file, { uploadId, objectName });

    // 上传完成 --- 提供 finishMultipartUpload 接口
    await finishMultipartUpload(uploadId, { objectName });

    // 这个数据是存入到 form 表单的
    return { name: file.name, path };
  } catch {
    message.error('文件上传失败！');
  } finally {
    // 从记录的正在上传的文件列表中移除
    const idx = this.uploadingFiles.findIndex((f) => f.uid === file.uid);
    this.uploadingFiles.splice(idx, 1);
  }
}
```

以上就是文件分段并发上传的前端核心代码。那服务端应该怎么实现呢？

### `NodeJS` 接口实现

由上面的核心代码可以看到，需要提供三个接口:获取分段上传前置信息(`uploadId, objectName`)、获取文件片的预签名上传地址、完成上传(完整性校验)。

如果接口要设配不同的文件服务器，比如`minio、华为obs、阿里云`，需要自己封装一套`OssClient`的抽象父类，由子类集成并重写`client`提供的`API`.

```typescript
private client: MinioClient | ObsClient | AliOssClient;
private DEFAULT_BUCKET = 'xxx';

// 获取分段上传前置信息
@Post('/multipart-upload')
public async startMultipartUpload(@Query('fileName') fileName: string) {
  const bucketName = DEFAULT_BUCKET; // bucket 名称，可以来自 yml 配置
  const objectName = this.getUploadObject(fileName); // 自己生成唯一键 objectName

  // 初始化分段上传，获取 uploadId
  const res = await client.initMultipartUpload(bucketName, objectName);

  return { uploadId: res.uploadId, objectName };
}

// 获取文件片的预签名上传地址
@Get('/multipart-upload/part/presign')
public async getPartUploadUrl(
  @Query('objectName') objectName: string,
  @Query('partNumber') partNumber?: string,
  @Query('uploadId') uploadId?: string,
) {
  if ((isNil(partNumber) && !isNil(uploadId)) || (!isNil(partNumber) && isNil(uploadId))) {
    throw new BadRequestException('分段上传需要同时提供 partNumber以及 uploadId 参数');
  }

  const bucketName = DEFAULT_BUCKET;

  // 默认24小时过期，算上传预签名地址
  const url = await client.presignedPutObject(
    bucketName,
    objectName,
    24 * 60 * 60,
    { partNumber, uploadId },
  );

  const urlObj = new URL(url);
  // query 上记录 bucket 参数，返回给前端
  return appendQueryParams(`${urlObj.pathname}${urlObj.search}`, { bucket: bucketName });
}

// 完成上传
@Post('/multipart-upload/:uploadId')
public async finishMultipartUpload(
  @Param('uploadId') uploadId: string,
  @Query('objectName') objectName: string,
) {
  if (isNil(objectName)) {
    throw new BadRequestException('对象名称不能为空');
  }

  const bucketName = DEFAULT_BUCKET;

  // 读取当前 uploadId 的所有已上传的文件块
  const parts = await client.listParts(bucketName, objectName, uploadId);
  // 文件分段上传完整性校验
  const res = await client.completeMultipartUpload(bucketName, objectName, uploadId, parts);

  return { etag: res.etag, name: res.name };
}
```
