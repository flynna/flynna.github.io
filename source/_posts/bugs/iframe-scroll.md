---
title: 容器嵌入iframe后的滚动问题记录
date: 2022-07-14 16:32:59
updated: 2022-07-14 16:32:59
tags:
  - bug
  - iframe滚动
categories:
  - 土豆の踩坑之路
  - 大前端
---

### 问题描述

最初，我做了一个`h5`的页面，该页面使用`iframe`内链了一个网页。当我直接在手机浏览器打开时，发现无法正常进行滚动。

拟解决办法：我给这个`iframe`添加了一个包裹容器，通过设置该容器的`overflow: auto`，以及设置`iframe height:100%`达到预期可滚动的效果。

新问题：首轮解决办法确实能够让 iframe 滚动，不过页面却出现了两个滚动条，`what f?`那肯定就是`wrapper 和 iframe`出现了高度差了。

<!-- more -->

### 解决办法

> 分析高度差出现的原因，然后...解决。

从节点本身出发，`iframe`是内联元素，而内联元素是跟基线`baseline`对齐的，iframe 的后面有个`行内空白节点`(该节点产生的原因其实和`dom`结构有关系，下面贴上源代码：)

```html
<div class="iframe-container">
  <iframe src="https://www.baidu.com" frameborder="0" width="100%" class="iframe"></iframe>
</div>
```

可以看到`iframe`与`div`之间有个换行符，也可以理解为空白节点，空白节点占据着高度，`iframe`与空白节点的基线对齐，导致了`div`被撑开，从而出现滚动条。

解决方案：

```
方案一：设置iframe的vertical-align:top
方案二：设置父div的font-size:0
方案三：改变iframe的内联元素性质，display: block
```

通常采用第三种方案，改变`iframe`的元素性质，贴上部分源码：

```css
.iframe-container {
  height: 300px;
  overflow: auto;
  -webkit-overflow-scrolling: touch; // 兼容 ios
}
.iframe-container .iframe {
  height: 100%;
  display: block;
}
```

至此，`iframe`的滚动问题得到完美解决。~~算是踩了个小坑，以后注意就好了~~

> 试一下，你会比你自己想象中的还要强大
