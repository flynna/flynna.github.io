---
title: vi、vim 编辑器学习
date: 2022-07-30 14:49:50
updated: 2022-07-30 14:49:50
tags:
  - vi
  - vim
categories:
  - 工具学习
  - Vi/Vim
---

### Why study?

在`linux`系统中，如果需要对文件进行修改编辑，一般可以使用`vi/vim`编辑器。`vim`是`vi`的升级版本，所以同样兼容`vi`的所有指令，它们都是多模式编辑器。

`git-bash`工具在`windows`中实现了该功能，因此可以直接在`git-bash`中使用`vi/vim`。

~~一次偶然的需求，在`rancher`管理平台里对正在运行的容器日志在线修改，领导随口提了一句用`vi`，当时的我很懵逼，也很尴尬。🙄🤥😌 然后就查阅了相关资料，自己动手敲了试了一下，感觉还是有必要学习一下。~~

<!-- more -->

### `vi/vim`模式介绍

`vi`编辑器有两种操作模式：

> **命令模式**：输入的每个字符都是对正在编辑的文本文件执行某些操作的命令。**部分按键按下会切换命令模式至插入模式**，后面详细介绍。

> **插入模式**：输入的每个字符都会添加到文件中的文本中。**按\<Esc\> ( Escape ) 键关闭插入模式，同时光标会向前移一位**。

**`注意：UNIX 和vi都区分大小写。确保不要使用大写字母代替小写字母；否则结果可能不会是你所期望的。`**

---

### 编辑器的启动和关闭

#### 启动`vi/vim`编辑器（进入`vi`）

> vi 文件名. （说明：如果文件存在，则打开该文件，如果参文件不存在，则会新建该文件。不是立即创建，而是在保存的时候创建）

**注意**：如果在`vi`编辑过程中遇到不可逆的操作导致强制退出的情况，直接输入上面命令会抛异常。当然可以在抛异常后根据提示输入`r`恢复；也可以直接在`vi`启动的时候添加`-r`参数避免这个异常提示。

> vi -r 文件名

进入过后，默认是插入模式，可以对文件内容做任意修改。

#### 关闭`vi/vim`编辑器

通常在你想离开`vi`界面，并且保存你所修改的内容的时候使用。或者不保存直接退出。

> **只要键入冒号(:)，光标就会移动到底部命令行**。

> 输入 **:wq 或者 :x** 回车即可完成保存并退出；

> 输入 **:q! 或者 :q** 回车即可不保存直接退出；

---

### 编辑器中的光标移动

与其他编辑器不同的是，鼠标无法在`vi`操作界面移动或者指定光标的位置，只能通过键盘完成这个操作。

在某些平台上，可以直接使用方向（箭头）键，但是部分不包括箭头键的键盘又该如何使用呢？`vi`也是考虑过这个问题，可以通过字母键来移动。

值得一提的是：方向键在两种模式下均可使用，而**代替键仅可在命令模式下使用**。（避免与插入模式下输入的字符冲突）

> 左：h 或者 ← 键，将光标左移一个字符。

> 右：l 或者 → 键，将光标又以一个字符。

> 上：k 或者 ↑ 键，将光标上移一行。

> 下：j 或者 ↓ 键，将光标下移一行。

> 跳转到首行：gg 或者 :0 或者 :1 键

> 跳转到最后一行：G 键

> 跳转至 n 行：输入 :n 回车，这里 n 代表数字。

> 跳转至当前行行头：0 或者 ^ 键。

> 跳转到当前行行尾：$ 键。

> 跳转到上个单词的开头：b 键。

> 跳转到下个单词的开头：w 键。

**`注意：方向键（左右）只能在当前行的可编辑区域进行移动，不会跨行边界`**

---

### 屏幕操作

以下命令允许`vi`编辑器屏幕（或窗口）向上或向下移动几行并进行刷新。

> 上翻一屏：Ctrl + b（PageUp）。

> 上翻半屏：Ctrl + u。

> 下翻一屏：Ctrl + f（PageDown）。

> 下翻半屏：Ctrl + d。

---

### 撤销、删除和复制粘贴

#### 回退、撤销

> Ctrl + u 撤消您刚才所做的一切（而非一步操作）。

#### 删除

> 删除整个当前行：dd

> 删除多行：dnd 或者 ndd，n 指的行数

> 删除单个字符：x，类似于 backspace 键效果

> 删除多个字符：nx，n 指的数字，从光标位置开始

#### 复制粘贴

> 粘贴：p，将粘贴内容放到当前行文本之后。

> 复制当前行：yy

> 复制多行：nyy 或者 yny，n 值行数

#### 修改

~~反正我是没怎么用过这个修改指令，习惯使然，有这个需求的时候我都是先删除再添加，避免出错 🙈🙉🙊~~详见末尾**其他指令**。

> 光标位置的字符替换：r 键 + 输入你要替换的字符

---

### 模式切换 and 插入操作

在命令模式下，想要输入文本内容，就需要切换到编辑模式。涉及的键有好几个，分别是`a i o s`，它们的区别：

> a：在光标后追加文本（光标会向后移动一位）

> i：在光标前插入文本（光标位置不会变化）

> o：新建并跳转至光标行的下一行

> s：删除光标后一个字符，并切换至插入模式

### 文本搜索

~~光速打脸，刚说修改字符的命令没啥用，这就来了~~

场景：输入你要搜索的文本，替换为你想要的新内容。

> 从上到下搜索：/ + 你要搜索的内容 + 回车

> 从下向上搜索：? + 你要搜索的内容 + 回车。

> 跳转至搜索到的下一个匹配项：n 键，一直按，一直向后匹配

### 获取行号

> 获取当前光标所在行的行号：:.= 键回车

> 获取文件总行数（末尾行号）：:= 键回车

### [其他指令](https://www.cs.colostate.edu/helpdocs/vi.html)