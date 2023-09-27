---
title: 使用 Nginx 搭建静态资源服务器
date: 2023-09-26 18:35:55
updated: 2023-09-26 18:35:55
tags:
  - 静态网站
  - 服务器
  - nginx
categories:
  - 工具学习
  - Nginx
---

> `Nginx` 通常用于托管 Web 应用程序、静态网站、代理请求到后端应用程序服务器（如 `Node.js`、`Ruby`、`Python` 等），以及为高流量和高可用性场景提供解决方案。

有些场景下，需要我们通过 `nginx` 搭建一个可访问的静态资源服务器，而无需繁琐的环境配置和程序运行。例如：托管不涉及前后端交互的静态资源网站...

<!-- more -->

### `nginx` 下载安装

[前往官网](https://nginx.org/en/download.html)，选择 `windows` 稳定版（`stable version：nginx/Windows-xxx `）下载压缩包，然后解压。

<div class="danger">

> `Tips:`
>
> 由于不同的 `terminal` 限制，有的指令需要可能会抛异常，有的需要管理员权限，为了一步到位，直接在`nginx.exe` 所在根目录用管理员打开 `powershell` 操作即可
>
> 此外，下面的一些常用指令均是针对 `windows` 环境提出的，`linux` 环境下还可通过其他指令完成

</div>

### 启动 `nginx`

- 方式一： 双击 `nginx.exe`，屏幕会弹一个黑框闪一下。

- 方式二： 打开 `cmd` 窗口，切换到 `nginx.exe` 目录，输入指令：

```bash
# 方式一
nginx.exe

# 方式二
start nginx
```

### 访问 `nginx`

默认情况下，启动成功后会在本地开启 `80` 端口服务。在浏览器地址栏输入 `http://localhost:80`，出现如下界面表示启动成功：

[![nginx-static-server-p1](/images/tools/nginx-static-server/p1.png)](/images/tools/nginx-static-server/p1.png)

当然你也可以通过指令观察是否启动成功：

```bash
tasklist /fi "imagename eq nginx.exe"
```

结果如下：

[![nginx-static-server-p2](/images/tools/nginx-static-server/p2.png)](/images/tools/nginx-static-server/p2.png)

### 配置 `nginx`

比如我们可以修改默认的 `80` 端口、服务名称、入口文件等等。

在刚刚解压的目录文件里，找到 `conf` 文件夹下的 `nginx.conf` 文件，用文本编辑器打开，修改如下配置：

[![nginx-static-server-p3](/images/tools/nginx-static-server/p3.png)](/images/tools/nginx-static-server/p3.png)

可以看到，我修改了一些比较常见的，比如端口、入口文件...一个简单的文件服务就配置好了 😍😍😍

<div class="info">

> `Tips`:
>
> 检测 `80` 端口是否被占用：
>
> `netstat -ano | findstr 0.0.0.0:80`
>
> 或者
>
> `netstat -ano | findstr "80"`

</div>

### 重启 `nginx`

当修改完配置文件后，需要重启 `nginx` 服务，才能生效。~~不需要执行关闭操作~~

```bash
nginx -s reload
# 或者根据提示
./nginx -s reload
```

### 关闭 `nginx`

值得一提的是，当我们使用完以后，记得要关闭 `nginx` 服务，不然会占用我们的端口，影响其他服务。

又特别是我们在测试一些代码的时候，由于更新了静态资源文件，此时不关闭上一次启动的服务，再一次启动 `nginx`，是不会抛异常的，而代价就是，我们看到的页面是上一次的，而不是最新的。

关闭窗口并不能关掉服务，需要执行指令：

```bash
# nginx.exe 所在根目录
.\nginx -s stop
# 或者
./nginx -s quit

# 推荐 --- 一般在脚本中使用该指令，避免由于异常导致无法正常关闭
taskkill /f /t /im nginx.exe
```

### 实战

创建服务文件夹，将 `nginx` 相关资源放入子文件夹，然后将项目打包后的代码粘贴到文件夹内，最后启动服务即可。

项目目录形如：

```
- assets
- images
- nginx
- index.html
...
```

`nginx` 目录即为下载的 `nginx` 解压目录，相关配置如上面截图所示。

然后我们需要给这个静态资源服务器添加启动和关闭的脚本命令~~总不能每次都自己打开 `cmd` 写指令吧...太麻烦了~~

#### 脚本编写

- `start.bat`

```bat
@ECHO OFF
ECHO Starting NGINX
cd nginx
start nginx.exe

start chrome http://localhost:5000

popd
EXIT /b
```

- `stop.bat`

```bat
@ECHO OFF
taskkill /f /IM nginx.exe
EXIT
```

<div class="success">

> 说明：
>
> `@ECHO OFF` 表示不在命令提示符窗口显示这句话以下的所有命令（即只显示输出结果）
>
> `start` 用于启动程序 `taskkill` 杀死程序
>
> `popd` 将当前工作目录还原为之前保存的目录（即返回到脚本执行之前的目录）
>
> `EXIT` 退出批处理文件（/b 表示返回退出代码）

</div>

效果如下：

[![nginx-static-server-p4](/images/tools/nginx-static-server/p4.png)](/images/tools/nginx-static-server/p4.png)
