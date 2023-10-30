---
title: 常用的功能辅助函数列举
date: 2023-10-07 10:41:01
updated: 2023-10-07 10:41:01
tags:
  - 辅助函数
categories:
  - 技术分享
  - 辅助函数
---

> 列举一些在日常开发中常用的辅助函数，以便于快速开发

<!-- more -->

## 资源加载相关

### 加载一个远程脚本

从给定的地址链接加载 `js` 脚本，并在加载完成后执行后续的一些操作

```ts
const needRunCallbacks: Record<string, any[]> = {};

function onScriptLoaded($script: HTMLScriptElement) {
  const onEnd = 'onload' in $script ? stdOnEnd.bind($script) : ieOnEnd.bind($script);
  onEnd($script);

  function stdOnEnd(this: any, script: HTMLScriptElement) {
    const _that = this;
    const handler = (isError: boolean) => {
      const error = isError ? new Error(`Failed to load ${src}`) : null;

      // 移除事件监听，避免重复执行
      _that.onerror = _that.onload = null;
      needRunCallbacks[script.src]?.forEach((callback) => callback(error, script));

      delete needRunCallbacks[script.src];
    };

    script.onload = () => handler(false);
    script.onerror = () => handler(true);
  }

  function ieOnEnd(this: any, script: HTMLScriptElement) {
    const _that = this;

    script.onreadystatechange = () => {
      if (_that.readyState !== 'complete' && _that.readyState !== 'loaded') return;

      _that.onreadystatechange = null;
      needRunCallbacks[script.src]?.forEach((callback) => callback(null, script));

      delete needRunCallbacks[script.src];
    };
  }
}

/**
 * 加载一个远程脚本
 * @param {String} src 一个远程脚本
 * @param {Function} callback 回调
 */
export function loadScript(src: string, callback?: (err?: Error, script?: any) => void) {
  const existScript = document.getElementById(src);
  const cb = callback || (() => {});

  if (existScript) {
    // 如果当前脚本资源还未结束加载，则只需要往回调数组里追加，否则(表示已加载完成)直接执行即可
    return needRunCallbacks[src] ? needRunCallbacks[src].push(cb) : cb();
  } else {
    const $script = document.createElement('script');

    needRunCallbacks[src] = [cb];
    $script.src = src;
    $script.id = src;
    $script.async = true;
    document.body.appendChild($script);

    onScriptLoaded($script);
  }
}
```

同上实现，如果要卸载脚本资源：

```ts
export function unloadScript(src: string) {
  document.getElementById(src)?.remove();
}
```

### 加载一组远程脚本

在上面的基础上，添加实现：

```ts
import { cloneDeep } from 'lodash';

/**
 * 顺序加载一组远程脚本
 * @param {Array} list 一组远程脚本
 * @param {Function} cb 回调
 */
export function loadScripts(srcs: string[], callback?: () => void) {
  const cloneSrcs = cloneDeep(srcs);
  const firstSrc = cloneSrcs.shift() as string;

  firstSrc ? loadScript(firstSrc, () => loadScripts(cloneSrcs, callback)) : callback?.();
}
```

## `moment/dayjs`相关

### 时间格式化

通常在开发中，存在类似这样的需求：`将时间字符串转为 今天..昨天..具体时间`，下面通过 `moment/dayjs(包体积更小，两者 api 大致相同)` 进行实现：

```ts
/**
 * 将时间字符串转为 今天..昨天..具体时间
 * @param {String} timeStr 目标时间
 * @param {String} defaultFormat 默认输出格式
 * @return {String} 转换后的时间
 */
export function timeFormat(timeStr: string, defaultFormat = 'YYYY-MM-DD HH:mm') {
  const dataTime = moment(timeStr);
  const timeSuffix = dataTime.format('HH:mm');

  const isToday = dataTime.isSame(moment().startOf('day'), 'd');
  const yd = moment().subtract(1, 'days').startOf('day');
  const isYesterday = dataTime.isSame(yd, 'd');

  // 或者你可以再分的细一点，当年的不显示年份...
  if (isToday) return '今天 ' + timeSuffix;
  else if (isYesterday) return '昨天 ' + timeSuffix;
  return dataTime.format(defaultFormat);
}
```

## 复制/粘贴相关

### 文本选中

```ts
/**
 * 选中元素中的文本
 * @param {Element} textEl 目标元素
 * @param {Number} startIndex 起始索引
 * @param {Number} stopIndex 结束索引
 */
export function selectText(textEl: any, startIndex: number, stopIndex: number) {
  if (textEl.createTextRange) {
    // ie
    const range = textEl.createTextRange();
    range.collapse(true);
    range.moveStart('character', startIndex); // 起始光标
    range.moveEnd('character', stopIndex - startIndex); // 结束光标
    range.select(); // 不兼容苹果
  } else {
    // firefox/chrome
    textEl.setSelectionRange(startIndex, stopIndex);
    textEl.focus();
  }
}
```

### 文本复制到剪贴板（丢弃）

考虑到复制的文本中存在换行符，所以使用的 `textarea`元素：

```ts
/**
 * 将一段文本复制到剪贴板
 * @param {String} value 文本内容
 */
export function setClipboard(value: string) {
  const tempInput = document.createElement('textarea');
  (tempInput as any).style = 'position: absolute; left: -1000px; top: -1000px';
  tempInput.value = value;
  document.body.appendChild(tempInput);
  tempInput.select();
  document.execCommand('copy');
  document.body.removeChild(tempInput);
}
```

### 文本复制到系统剪贴板（新）

```ts
async function copyToClipboard(text: string) {
  if (navigator.clipboard) {
    try {
      await navigator.clipboard.writeText(text);
      return true;
    } catch (error) {
      console.error('无法将文本复制到剪贴板：', error);
      return false;
    }
  } else {
    // 备用方案：在不支持 navigator.clipboard 的浏览器中创建临时输入框并复制文本
    const tempInput = document.createElement('textarea');
    tempInput.value = text;
    document.body.appendChild(tempInput);
    tempInput.select();
    document.execCommand('copy');
    document.body.removeChild(tempInput);
    return true;
  }
}
```

### 从系统剪贴板读取内容

```ts
async function readFromClipboard() {
  let text = '';
  if (navigator.clipboard) {
    try {
      text = await navigator.clipboard.readText();
    } catch (error) {
      console.error('无法从剪贴板读取文本：', error);
    }
  } else {
    // 备用方案：在不支持 navigator.clipboard 的浏览器中使用 document.execCommand('paste')
    const tempInput = document.createElement('textarea');
    document.body.appendChild(tempInput);
    tempInput.focus();
    document.execCommand('paste');
    text = tempInput.value;
    document.body.removeChild(tempInput);
  }
  return text;
}
```

<div class='warning'>

> 注意：基于安全性和隐私保护限制，`document.execCommand` 不会直接从系统剪贴板获取内容。所以更多时候请优先使用 `navigator.clipboard`，因为它提供了更安全的方法来执行这些任务，并且更具兼容性。

</div>

### `HTML` 复制到剪贴板

在需要将元素带样式复制到 `word/excel` 中的场景使用（例如表格复制？）：

```ts
/**
 * 将 html 带格式复制到剪贴板
 * @param {Element} elToBeCopied 需要粘贴的元素节点 eg. document.querySelector('.table')
 */
export function copyEl(elToBeCopied: Element) {
  let sel;
  // Ensure that range and selection are supported by the browsers
  if (document.createRange && window.getSelection) {
    const range = document.createRange();
    sel = window.getSelection();
    // unselect any element in the page
    sel?.removeAllRanges();

    try {
      range.selectNodeContents(elToBeCopied);
    } catch (e) {
      range.selectNode(elToBeCopied);
    } finally {
      sel?.addRange(range);
    }

    document.execCommand('copy');
  }
  sel?.removeAllRanges();
}
```

## 文字头像相关

### 根据名称随机设置背景

```ts
/**
 * 根据名称随机设置背景
 *
 * @param {String} name 名称
 * @returns {String} 背景色
 */
export function randomColorByName(name: string) {
  let str = '';
  // 避免名称过短导致不能生成正常长度的十六进制数
  let tempName = name.padEnd(2, 'a');
  0;
  for (let i = 0; i < tempName.length; i++) {
    str += parseInt(name[i].charCodeAt(0), 10).toString(16);
  }
  return '#' + str.slice(1, 4);
}
```

### 通过 `canvas` 绘制文字头像

```ts
/**
 * 根据传入的尺寸和名字生成文字头像
 *
 * @param {[Number, Number]} size 头像尺寸
 * @param {String} name 名字
 * @param {String} bgc 背景色
 * @returns 返回生成的图片
 * @example
 * import { genTextImg } from '@/utils/genTextImg';
 * genTextImg([100, 100], '张三');
 */
export function genTextImg(size: [number, number], name: string, bgc?: string) {
  const showName = name?.charAt(0);
  const cvs = document.createElement('canvas');
  cvs.setAttribute('width', size[0]);
  cvs.setAttribute('height', size[1]);

  const ctx = cvs.getContext('2d')!;

  ctx.fillStyle = bgc ?? randomColorByName(name);
  ctx.fillRect(0, 0, size[0], size[1]);
  ctx.fillStyle = '#fff';

  ctx.font = size[0] * 0.6 + 'px Arial';
  ctx.textBaseline = 'middle';
  ctx.textAlign = 'center';
  ctx.fillText(showName, size[0] / 2, size[1] / 2);

  return cvs.toDataURL('image/jpeg', 1);
}
```

## `window.location` 相关

### 链接追加 `query` 参数

```ts
import querystring from 'querystring';

/**
 * 在链接末尾追加上指定的query参数
 *
 * @param {String} link 链接
 * @param {String} queryId 参数名
 * @param {String} queryVal 参数值
 *
 * @returns 返回新的链接
 */
export const appendLinkQuery = (link: string, queryId: string, queryVal: any) => {
  const queryObj = querystring.parse(link.split('?')[1]);
  const query = querystring.stringify({ ...queryObj, [queryId]: queryVal });
  return `${link.split('?')[0]}?${query}`;
};
```

### 获取 `hash` 路径中需要的 `key` 值

`形如：http://www.xxx.com/#/xxx?a=1&b=true&c=name`

```ts
/**
 * 获取 hash 路径中需要的 queryKey 的值
 * @param {String} [key] 可选，需要的 queryKey
 * @returns {String | Object} 返回 queryKey 的值，未传 queryKey 时返回整个 query 对象
 */
export function getHashParams(queryKey?: string) {
  const params: Record<string, string> = {};
  const query = window.location.hash.split('?')[1];

  if (query) {
    const keyValuePairs = queryString.split('&');

    keyValuePairs.forEach((keyValue) => {
      const [key, value] = keyValue.split('=');
      if (key && value) {
        const decodedKey = decodeURIComponent(key);
        const decodedValue = decodeURIComponent(value);
        params[decodedKey] = decodedValue;
      }
    });
  }

  return queryKey ? params[queryKey] : params;
}
```

### 获取 `hash` 内部的 `query` 参数值

`形如：http://www.xxx.com/?d=1&e=2#/xxx?a=1&b=true&c=name` 从中获取 `d` 的值

```ts
import qs from 'querystring';

/**
 * 获取 hash 内部的 query 参数值
 * @param {String} [queryKey] 可选，需要的 queryKey
 * @returns {String | Object} 返回 queryKey 的值，未传 queryKey 时返回整个 query 对象
 */
function getHashInnerParams(queryKey?: string) {
  const search = window.location.search;
  const filterSearch = search.split(encodeURIComponent('#/'))[0];

  const hashQuery = qs.parse(filterSearch?.slice(1));

  return queryKey ? hashQuery[queryKey] : hashQuery;
}
```

## 持续更新中...
