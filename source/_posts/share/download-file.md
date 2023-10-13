---
title: 前端实现文件下载的几种方案
date: 2023-10-11 16:15:19
updated: 2023-10-11 16:15:19
tags:
  - 文件下载
categories:
  - 技术分享
  - 功能实现
---

`前端下载文件已经是一个非常普遍的需求了，同时也已有了非常成熟的解决方案。简单做个罗列，并把可能会遇到的问题描述出来`

<!-- more -->

### `window.open` 或 `window.location.href`

最简单的方式，同 `a` 标签访问下载

```ts
window.open(url); // 默认是 '_self' 页面会闪一下，体验不是很完美
window.location.href = url;
```

> `url` 可以是文件地址，也可以是一个 `GET` 下载请求地址
>
> 浏览器可直接浏览的文件类型是不能通过这种方式下载的（当然我指的是 `url` 不是一个下载的 `get` 请求时）。例如：`txt/png/pdf...`
>
> 同时，这种方式还有个弊端，无法获取下载进度

### `a` 标签下载

指定下载的文件名并添加下载属性，不设置默认即为文件原名（考虑到兼容性问题，`filename` 后面尽量加上文件的类型后缀）。

```html
<a href="url" download="filename">下载</a>
```

> 相比较前面通过 `location.href` 的方式，这种方式可以下载任意类型的文件。

```ts
export function downloadFile(url: string, filename: string) {
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  document.body.appendChild(a); // appendChild 是为了更好的兼容
  a.click();
  document.body.removeChild(a);
}
```

### `iframe` 下载

```ts
export function downloadFile() {
  const iframe = document.createElement('iframe');
  iframe.style.display = 'none';
  iframe.src = url; // url 是可下载的文件链接
  document.body.appendChild(iframe);
  iframe.onload = () => {
    setTimeout(() => {
      document.body.removeChild(iframe);
    }, 0);
  };
}
```

### 基于 `Blob` 对象下载

这种方式可以说是最通用的下载方式了，兼容性也很好。通过网络请求从后端获取，利用 `Blob` 对象将文件流转化为 `Blob` 二进制对象:

> 现在主流的网络请求工具，例如 `axios...` 是支持设置 `responseType` 的，所以可以直接使用 `responseType = 'blob'` 的方式下载。

```ts
export function downloadFile(url: string, filename: string) {
  const res = await axios.post(url, undefined, {
    responseType: 'blob',
    params,
  });
  const resData = res.data;

  const link = document.createElement('a');
  link.href = URL.createObjectURL(resData);
  link.download = filename;

  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
  URL.revokeObjectURL(link.href);
}
```

> `URL.createObjectURL` 作用同 `fileReader.readAsDataURL`，只不过后者是转为 `Data(base64) URL`，前者转为 `Blob URL`。

通过网络请求下载，可以配置函数来自定义下载进度显示（例： `axios`，则在请求体的 `config.onUploadProgress` 中）：

```ts
const fileConfig = {};

const defaultUploadProgress: OnUploadProgress = (e) => {
  const percentage = (e.loaded / e.total!) * 100;
  if (percentage !== 100) {
    fileConfig.percentage = Number(percentage.toFixed(2)) || 0;
  }
};
```

通常，在下载文件时，会在服务端生成对请求文件类型的描述信息，例如 `Content-Type`，`Content-Disposition` 等，这些信息需要与下载的文件名一起发送，例如：

```ts
res.setHeader('Content-Type', 'application/zip');
res.setHeader(
  'content-disposition',
  `attachment;filename=${encodeURIComponent('资源文件列表')}.zip`,
);
```

当服务端配置了以后，前端下载文件即可无需通过 `new Blob([res], { type: 'application/octet-stream' })` 进行指定。

### 文件预览

默认情况下，访问文件资源，部分类型的文件仅支持预览而无法下载，同理，部分能够下载的文件又不支持预览，此时，我们需要通过网络请求去获取资源，通过设置设置 `content-disposition` 告诉浏览器是需要预览还是下载。

例如文件资源链接 `/s3/xxx/filename.pdf`，那么预览：`/s3/xxx/filename.pdf?preview=true`，下载即为：`/s3/xxx/filename.pdf?download=true`

`server` 实现：

```ts
if (req.query.preview) {
  res.setHeader('content-disposition', `inline;filename=filename.pdf`);
} else if (req.query.download) {
  res.setHeader('content-disposition', `attachment;filename=filename.pdf`);
}
```

`client` 实现：

```ts
// preview
export function previewInNewTab(imgUrl: string) {
  const img = `<iframe src="${imgUrl}&preview=true" style="width: 100vw;height: 100vh;border: 0;margin: -8px;" />`;
  const popup = window.open();
  popup?.document.write(img);
}

// download
export function downloadFile(fileUrl: string) {
  window.location.href = `${fileUrl}&download=true`;
}
```
