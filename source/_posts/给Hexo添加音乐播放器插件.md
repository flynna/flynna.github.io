---
title: 给Hexo添加音乐播放器插件
date: 2021-07-10 17:43:42
tags: [hexo插件, Aplayer, MetingJS]
# type: "categories"
categories: [工具, Hexo]
---

[toc]

## 转：[基于外链生成播放器插件](https://yelog.org/2019/10/08/3-hexo-add-music/)

### 网易云音乐

前往[网易云音乐官网](https://music.163.com/)，搜索一个作为背景音乐的歌曲，并进入播放页面，点击 **生成外链播放器**

![image-20210710165708010](/images/给Hexo添加音乐播放器插件/image-20210710165708010.png)

设置好想要显示的样式后，复制 `html` 代码

![image-20210710165810418](/images/给Hexo添加音乐播放器插件/image-20210710165810418.png)

最好外层在加一个 `div`，如下，可直接将上一步复制的 `iframe` 替换下方里面的 `iframe`

```html
<div
  id="musicMouseDrag"
  style="position:fixed; z-index: 9999; bottom: 0; right: 0;"
>
  <div
    id="musicDragArea"
    style="position: absolute; top: 0; left: 0; width: 100%;height: 10px;cursor: move; z-index: 10;"
  ></div>
  <iframe
    frameborder="no"
    border="0"
    marginwidth="0"
    marginheight="0"
    width="330"
    height="86"
    src="//music.163.com/outchain/player?type=2&id=38592976&auto=1&height=66"
  >
  </iframe>
</div>
```

### 外链引入博客: `3-hexo` 主题

将上一步加过 `div` 的代码粘贴到主题下 `layout/_partial/footer.ejs` 的最后面

### 调整位置

默认给的样式是显示在右下角，可以通过调整上一步粘贴的 `div` 的 `style` 中 `bottom` 和 `right` 来调整位置。

### 自由拖动

如果需要自由拖动，在刚才添加的代码后面，再添加下面代码即可，鼠标就可以在音乐控件的 **上边沿** 点击拖动。

```html
<!--以下代码是为了支持随时拖动音乐控件的位置，如没有需求，可去掉下面代码-->
<script>
  var $DOC = $(document);
  $("#musicMouseDrag").on("mousedown", function (e) {
    // 阻止文本选中
    $DOC.bind("selectstart", function () {
      return false;
    });
    $("#musicDragArea").css("height", "100%");
    var $moveTarget = $("#musicMouseDrag");
    $moveTarget.css("border", "1px dashed grey");
    var div_x = e.pageX - $moveTarget.offset().left;
    var div_y = e.pageY - $moveTarget.offset().top;
    $DOC
      .on("mousemove", function (e) {
        var targetX = e.pageX - div_x;
        var targetY = e.pageY - div_y;
        targetX =
          targetX < 0
            ? 0
            : targetX + $moveTarget.outerWidth() >= window.innerWidth
            ? window.innerWidth - $moveTarget.outerWidth()
            : targetX;
        targetY =
          targetY < 0
            ? 0
            : targetY + $moveTarget.outerHeight() >= window.innerHeight
            ? window.innerHeight - $moveTarget.outerHeight()
            : targetY;
        $moveTarget.css({
          left: targetX + "px",
          top: targetY + "px",
          bottom: "inherit",
          right: "inherit",
        });
      })
      .on("mouseup", function () {
        $DOC.unbind("selectstart");
        $DOC.off("mousemove");
        $DOC.off("mouseup");
        $moveTarget.css("border", "none");
        $("#musicDragArea").css("height", "10px");
      });
  });
</script>
```

---

`网易云单曲外链没什么问题，但是否能满足list需求呢？于是接入了歌单等外链，经测试发现，歌单外链并不可用，循环播放始终是同一首歌，并且经常会资源加载失败！，于是又开始了对 hexo 中添加列表音乐插件查询...`

## 基于`Aplayer ` 插件使用

[hexo-tag-aplayer](https://easyhexo.com/3-Plugins-use-and-config/3-1-hexo-tag-aplayer/#%E4%BB%8B%E7%BB%8D)

安装 `hexo-tag-aplayer` 插件 ,在根目录执行：

```bash
npm install hexo-tag-aplayer -s
npm install aplayer --save
```

修改 `Hexo` 的配置文件 `_config.yml`:

```yaml
aplayer:
  script_dir: some/place # Public 目录下脚本目录路径，默认: 'assets/js'
  style_dir: some/place # Public 目录下样式目录路径，默认: 'assets/css'
  cdn: http://xxx/aplayer.min.js # 引用 APlayer.js 外部 CDN 地址 (默认不开启)
  style_cdn: http://xxx/aplayer.min.css # 引用 APlayer.css 外部 CDN 地址 (默认不开启)
  meting: true # MetingJS 支持
  meting_api: http://xxx/api.php # 自定义 Meting API 地址
  meting_cdn: http://xxx/Meing.min.js # 引用 Meting.js 外部 CDN 地址 (默认不开启)
  asset_inject: true # 自动插入 Aplayer.js 与 Meting.js 资源脚本, 默认开启
  externalLink: http://xxx/aplayer.min.js # 老版本参数，功能与参数 cdn 相同
```

使用 `hexo-tag-aplayer` 非常简单，只需要在 `MarkDown `文件中插入正确的标记就可以了。

```markdown
{% aplayer title author url [picture_url, narrow, autoplay, width:xxx, lrc:xxx] %}
```

开启了[文章资源文件夹 (opens new window)](https://hexo.io/zh-cn/docs/asset-folders.html#文章资源文件夹)功能, 可以这样使用：

```markdown
{% aplayer "Caffeine" "Jeff Williams" "caffeine.mp3" "picture.jpg" "lrc:caffeine.txt" %}
```

---

`以为这样就结束了吗，并没有。。。我发现这样使用确实可以实现在文章内部完成对播放器的生成及使用，同时满足了对单曲，歌单列表等不同类型的插件加载。但是核心需求，在全局生成次模块并没有实现，怎样在不大范围的修改代码同时，最简单的实习对播放器插件的引入呢？`

## 基于`MetingJS` 生成

### 在`md`文章内生成播放器

通过引入`MetingJS`，播放器将支持`QQ`音乐、网易云音乐、虾米、酷狗、百度等平台的音乐播放。
这种方式比较方便添加在线音乐播放列表。引入需要在`Hexo`的配置文件`_config.yml`中设置

```yaml
aplayer:
  meting: true # MetingJS 支持
  asset_inject: true # 自动插入Aplayer.js与Meting.js资源脚本（默认开启）
```

设置后页面在初始化播放器时会引入 1.2.0 版本的`Meting.min.js`文件
注意不是在主题配置文件`_config.yml`设置，设置后要重启`hexo`服务才能生效 。

meting 格式：

```markdown
{% meting "id" "server" "type"%}
```

具体的选项列表见：[MetingJS](https://easyhexo.com/3-Plugins-use-and-config/3-1-hexo-tag-aplayer/#metingjs)

### 全局播放实现

- 先完成根目录 `_config.yml` 的配置。如上
- 在 global 的页面文件内插入代码，如在 `3-hexo` 主题中，我选择了在 `layout/_partial/footer.ejs` 最后面添加：

```html
<!-- Meting 配置需要添加 data- 前缀, 具体含义见文档 -->
<div
  class="aplayer"
  data-id="2176799008"
  data-server="tencent"
  data-type="playlist"
  data-fixed="true"
  data-listfolded="true"
  data-autoplay="false"
  data-order="list"
  data-volume="0.5"
  data-theme="#1da496"
  data-preload="auto"
></div>
<link
  rel="stylesheet"
  href="https://cdn.jsdelivr.net/npm/aplayer@1.10.1/dist/APlayer.min.css"
/>
<script src="https://cdn.jsdelivr.net/npm/aplayer@1.10.1/dist/APlayer.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/meting@1.2.0/dist/Meting.min.js"></script>
```

`注意 script 的引入顺序关系。 引入的metingjs版本得是1.2.0的，最新版本2.0.0与APlayer不兼容。看hexo-tag-aplayer的github仓库两年前更新后就没动了，只支持到metingjs的v1.2.0`

- 修改根目录下 刚刚配置的 `config.yml`:

```yaml
aplayer:
  meting: true
  asset_inject: false # 关闭自动脚本插入
```

这个时候 重启项目 `hexo server` 刷新页面即可看到吸底的播放器插件。

- 样式调整。发现歌词出现在了播放器外，可能设计是这样的，但是我想把它放到播放器内部进度条位置：

```html
<style>
  .aplayer.aplayer-fixed .aplayer-lrc,
  .aplayer.aplayer-fixed .aplayer-body {
    position: absolute !important;
    z-index: 999 !important;
  }
</style>
```

- 如果想又吸底。直接暴力修改样式：

```html
<style>
  .aplayer.aplayer-fixed {
    left: unset !important;
  }
  .aplayer .aplayer-miniswitcher {
    right: unset !important;
    left: -18px;
  }
</style>
```

## 最终效果

- 最小化:

![image-20210710174012507](/images/给Hexo添加音乐播放器插件/image-20210710174012507.png)

- 展开：

![image-20210710174055194](/images/给Hexo添加音乐播放器插件/image-20210710174055194.png)

- 最大化：

![image-20210710174125346](/images/给Hexo添加音乐播放器插件/image-20210710174125346.png)
