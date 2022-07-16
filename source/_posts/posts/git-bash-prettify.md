---
title: windows环境的git-bash终端美化
date: 2022-07-16 21:01:38
updated: 2022-07-16 21:01:38
tags:
  - git-bash
  - terminal
  - 终端美化
categories:
  - 土豆の收藏安利
---

### Before this

最开始是想着直接安装`zsh`，但是或多或少还是存在问题。

比如：安装完了过后，除去`vscode`的其他运行方式和环境都很`ok`，在`vscode`的`terminal`中，如果使用`git-bash`并且安装了`zsh`，那么只要`terminal`的窗口视图发生变化（新建、拆分终端，又或者改变`terminal`的高度），都会导致命令行无法正常输入，就像卡死了一般。其他的运行方式，比如直接右键在文件中使用`git-bash`，又或者是在`windows-terminal`中使用，都是没得问题的。如果有想要了解的伙伴，可以参考这篇文章。[一文搞定`Windows Terminal`设置与`zsh`安装【非`WSL`】](https://zhuanlan.zhihu.com/p/455925403)

当然，如果**只是想美化**，做到类似`zsh`的显示效果，有没有办法咧？~~💀💀💀~~

<!-- more -->

### 修改配置文件

找到文件`git-prompt.sh`，在安装目录下的`Git/etc/profile.d/git-prompt.sh`。右键在`vscode`中打开，或者终端输入`code git-prompt.sh(绝对路径，或者相对路径都可以)`。

替换为下面代码。实际只修改了`$PS1`的值：

> 注意：\w 表示的详细路径，如果只想展示当前工作区，设置为 \W

> 类似于 \[\033[32m\] 这样的值，是命令行的颜色色值，具体可参考：[.bash_aliases](https://gist.github.com/vratiu/9780109)

```sh
if test -f /etc/profile.d/git-sdk.sh
then
    TITLEPREFIX=SDK-${MSYSTEM#MINGW}
else
    TITLEPREFIX=$MSYSTEM
fi

if test -f ~/.config/git/git-prompt.sh
then
    . ~/.config/git/git-prompt.sh
else
    PS1='\[\033]0;Bash\007\]'      # 窗口标题
    # PS1="$PS1"'\n'               # 换行
    PS1="$PS1"'\[\033[32m\]'     # 高亮绿色
    PS1="$PS1"'➜ '               # 右箭头
    PS1="$PS1"'\[\033[33m\]'     # 高亮黄色
    PS1="$PS1"'[\w]'                 # 当前目录
    if test -z "$WINELOADERNOEXEC"
    then
        GIT_EXEC_PATH="$(git --exec-path 2>/dev/null)"
        COMPLETION_PATH="${GIT_EXEC_PATH%/libexec/git-core}"
        COMPLETION_PATH="${COMPLETION_PATH%/lib/git-core}"
        COMPLETION_PATH="$COMPLETION_PATH/share/git/completion"
        if test -f "$COMPLETION_PATH/git-prompt.sh"
        then
            . "$COMPLETION_PATH/git-completion.bash"
            . "$COMPLETION_PATH/git-prompt.sh"
            PS1="$PS1"'\[\033[31m\]'   # 红色
            PS1="$PS1"'`__git_ps1`'    # git 插件
        fi
    fi
    PS1="$PS1"'\[\033[0;36m\] '      # 青色
fi

MSYS2_PS1="$PS1"
```

### 效果

!['git-bash-preview'](/images/git-bash-prettify/p1.png)

<div class="primary">

> 把你的脸迎向阳光，那就不会有阴影

</div>
