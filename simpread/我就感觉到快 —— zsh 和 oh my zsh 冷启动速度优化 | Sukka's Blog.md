> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.skk.moe](https://blog.skk.moe/post/make-oh-my-zsh-fly/)

> 不论是在 WSL、Linux 还是 macOS 上，强大的 zsh 一直是我的不二法宝，而 oh my zsh 自然成了最趁手的瑞士军刀，我自己还编写了数个 oh my zsh 插件和主题。

不论是在 WSL、Linux 还是 macOS 上，强大的 zsh 一直是我的不二法宝，而 oh my zsh 自然成了最趁手的瑞士军刀，我自己还编写了数个 oh my zsh 插件和主题。直到有一天我突然发现：见鬼，为什么开个 iTerm2 的 Tab 要等上好几秒钟？

zsh 启动耗时测量
----------

首先，我们需要一个客观衡量 zsh 启动速度的标准，而使用 macOS 和众多 Linux 发行版中自带的 `time` 可以轻松计算任何命令的执行用时，包括 shell：

```
$ /usr/bin/time /bin/zsh -i -c exit

        1.77 real         1.04 user         0.95 sys

```

`time` 输出了 zsh 启动时 user-land 和 system 用时，而我的 zsh 启动用时将近 2 秒钟。为了获得更精确的结果，使用 for 循环连续启动 zsh 5 次：

```
$ for i in $(seq 1 5); do /usr/bin/time /bin/zsh -i -c exit; done

        1.74 real         1.02 user         0.92 sys
        1.69 real         1.00 user         0.90 sys
        1.71 real         1.01 user         0.91 sys
        1.68 real         0.99 user         0.89 sys
        1.74 real         1.02 user         0.93 sys

```

为了排除 zsh 本身的性能问题，使用 zsh 的 `--no-rcs` 参数进行测试：

```
$ for i in $(seq 1 20); do /usr/bin/time /bin/zsh --no-rcs -i -c exit; done

        0.00 real         0.00 user         0.00 sys
        0.00 real         0.00 user         0.00 sys
        0.00 real         0.00 user         0.00 sys
        0.00 real         0.00 user         0.00 sys
        0.00 real         0.00 user         0.00 sys

```

不加载 `.zshrc` 时，zsh 的启动速度是如此的快，以至于 `time` 给出了 `0.00` 的结果。

Profiling
---------

zsh 提供了专门的 profiling 模块 `zprof` 用于衡量 zsh 各个函数的执行用时。在 `.zshrc` 文件第一行添加下述命令用于加载 `zprof` 模块：

```
zmodload zsh/zprof

```

接着启动 zsh、并使用 `zprof` 命令获取各函数用时数据：

```
$ /bin/zsh
$ zprof

num  calls                time                       self            name
-----------------------------------------------------------------------------------
 1)    1         395.66   395.66   33.10%    395.59   395.59   33.09%  _zsh_nvm_auto_use
 2)    1         216.22   216.22   18.09%    216.13   216.13   18.08%  nvm_die_on_prefix
 3)    1         648.00   648.00   54.20%    168.85   168.85   14.12%  nvm_auto
 4)    2         479.15   239.57   40.08%    160.50    80.25   13.43%  nvm
 5)    1         102.30   102.30    8.56%     84.99    84.99    7.11%  nvm_ensure_version_installed
 6)    2          51.21    25.60    4.28%     29.55    14.78    2.47%  compinit
 7)    1         680.18   680.18   56.89%     22.17    22.17    1.85%  _zsh_nvm_load
 8)    2          21.66    10.83    1.81%     21.66    10.83    1.81%  compaudit
 9)    1          17.31    17.31    1.45%     17.31    17.31    1.45%  nvm_is_version_installed
10)  193          17.43     0.09    1.46%     14.50     0.08    1.21%  _zsh_autosuggest_bind_widget
[Redacted]

```

`zprof` 模块只能获取每个 zsh 函数的用时，因此适合找出拖累 zsh 冷启动的 oh my zsh 的插件。如果要获取完整的 `.zshrc` 性能分析，应该使用 `xtrace`。在 `.zshrc` 开头添加如下命令：

```
zmodload zsh/datetime
setopt PROMPT_SUBST
PS4='+$EPOCHREALTIME %N:%i> '

logfile=$(mktemp zsh_profile.7Pw1Ny0G)
echo "Logging to $logfile"
exec 3>&2 2>$logfile

setopt XTRACE

```

并在 `.zshrc` 结尾添加如下命令：

```
unsetopt XTRACE
exec 2>&3 3>&-

```

这会在 `$HOME` 目录下生成一个文件名包含随机字符串的文件（`zsh_profile.123456` ）。一些介绍 zsh profiling 的文章会推荐使用 [kcachegrind](http://kcachegrind.sourceforge.net/html/Home.html) 这个工具可视化这个文件，但是我们只需要知道是什么拖累了 zsh 冷启动，将这个文件格式化一下即可。这里提供一个简单的脚本：

```
#!/usr/bin/env zsh

typeset -a lines
typeset -i prev_time=0
typeset prev_command

while read line; do
    if [[ $line =~ '^.*\+([0-9]{10})\.([0-9]{6})[0-9]* (.+)' ]]; then
        integer this_time=$match[1]$match[2]

        if [[ $prev_time -gt 0 ]]; then
            time_difference=$(( $this_time - $prev_time ))
            lines+="$time_difference $prev_command"
        fi

        prev_time=$this_time

        local this_command=$match[3]
        if [[ ${#this_command} -le 80 ]]; then
            prev_command=$this_command
        else
            prev_command="${this_command:0:77}..."
        fi
    fi
done < ${1:-/dev/stdin}

print -l ${(@On)lines}

```

将上述内容保存在 `$HOME` 目录下 `format_profile.zsh` 文件中，然后在终端中执行：

```
$ cd $HOME
$ chmod +x format_profile.zsh
$ ./format_profile.zsh zsh_profile.123456 | head -n 30

356910 _zsh_nvm_auto_use:14> [[ none != N/A ]]
307791 /Users/sukka/.zshrc:312> hexo '--completion=zsh'
178444 /Users/sukka/.zshrc:310> thefuck --alias
161193 nvm_version:21> VERSION=N/A
148555 nvm_version:21> VERSION=N/A
96497 (eval):4> pyenv rehash
58759 /Users/sukka/.zshrc:311> pyenv init -
48629 nvm_auto:15> VERSION=''
42779 /Users/sukka/.zshrc:114> FPATH=/usr/local/share/zsh/site-functions:/usr/local...
42527 nvm_auto:15> nvm_resolve_local_alias default
41620 nvm_resolve_local_alias:7> VERSION=''
35577 nvm_resolve_local_alias:7> VERSION=''
29444 _zsh_nvm_load:6> source /Users/sukka/.nvm/nvm.sh
24967 compaudit:154> _i_wfiles=( )
24889 nvm_resolve_alias:15> ALIAS_TEMP=''
22000 nvm_auto:18> nvm_rc_version
20890 nvm_ls:29> PATTERN=default
[Redacted]

```

这样就一目了然了。可以看到，除了 `nvm` 以外、`hexo` 的自动补全、`thefuck` 的初始化、`pyenv` 都大幅拖慢了 zsh 的启动速度。

Lazyload
--------

你可能听过 [网页的图片可以 lazyload](https://blog.skk.moe/post/img-lazyload-hexo/)、[Disqus 评论系统可以 lazyload](https://blog.skk.moe/post/prevent-disqus-from-slowing-your-site/)，但是 `.zshrc` 一样也有 lazyload。lazyload 的特点是启动时快，首次使用时慢，因此很适合用于优化不常用而且初始化非常耗时的功能。

lazyload 的方法是声明一个占位函数，当执行这个函数时完成对真实命令的初始化、并移除命令占位。以 pyenv 为例：

```
export PATH="/Users/sukka/.pyenv/shims:${PATH}"

pyenv() {
  
  unfuntion pyenv

  
  eval "$(command pyenv init -)"

  
  pyenv "$@"
}

```

pyenv 在初始化时会自动加载补全（completion），但是由于 lazyload、第一次执行 `pyenv` 时就没有补全了，因此还需要为补全添加 lazyload：

```
__lazyload_completion_pyenv() {
  
  comdef -d pyenv
  
  unfunction pyenv
  
  source "$(brew --prefix pyenv)/completions/pyenv.zsh"
}

compdef __lazyload_completion_pyenv pyenv

```

这样，当首次输入 `pyenv` 并按下 Tab 时会加载 pyenv 的命令补全，第二次按下 Tab 时就可以正常显示命令补全了。

将上述 lazyload 封装成函数便于调用：

```
sukka_lazyload_add_command() {
    eval "$1() { \
        unfunction $1 \
        _sukka_lazyload__command_$1 \
        $1 \$@ \
    }"
}

sukka_lazyload_add_completion() {
    local comp_
    eval "${comp_name}() { \
        compdef -d $1; \
        _sukka_lazyload_completion_$1; \
    }"
    compdef $comp_name $1
}

```

使用封装好的 lazyload 函数添加 `pyenv` 和 `thefuck` 的 lazyload、Hexo completion 的 lazyload：

```
_sukka_lazyload__command_pyenv() {
  
  eval "$(command pyenv init -)"
}
_sukka_lazyload__compfunc_pyenv() {
  
  source "$(brew --prefix pyenv)/completions/pyenv.zsh"
}

sukka_lazyload_add_command pyenv
sukka_lazyload_add_completion pyenv

_sukka_lazyload__command_fuck() {
  
  eval $(thefuck --alias)
}

sukka_lazyload_add_command fuck

_sukka_lazyload__completion_hexo() {
  
  eval $(hexo --completion=zsh)
}

sukka_lazyload_add_completion hexo

```

替换 NVM
------

我使用 nvm 的方式是 `zsh-nvm` 插件。由于我的开发环境也高度依赖 `.nvmrc` 文件，所以不得不启用 nvm auto use。由于我的许多工具高度依赖 Node.js（如我的 [Nali CLI](https://nali.skk.moe/)），lazyload nvm 也是不现实的。我不得不寻找另一个代替 nvm 的 Node.js 版本管理器，最后我选中了 [`tj/n`](https://github.com/tj/n)。

首先是卸载 nvm、nvm 安装的所有 Node.js 版本、以及 zsh-nvm 插件：

```
$ rm -rf $HOME/.nvm


$ rm -rf $ZSH/custom/plugins/zsh-nvm



```

接着安装 `tj/n` 作为 Node.js 版本管理器，macOS 上可以通过 Homebrew 直接安装：

```
$ brew install n

```

在 `.zshrc` 中配置 `tj/n`：

```
export N_PREFIX="$HOME/.n"

export N_PRESERVE_NPM=1

export PATH="$N_PREFIX/bin:$PATH"

```

使用 zsh 内置语法
-----------

zsh 强大之处不仅在于内建的插件、优雅的使用方式，更重要的是极其强大的语法。在 `.zshrc` 广泛使用 zsh 内置的语法可以大幅提高执行性能。

### zsh 判断命令是否存在

我们经常需要在 `.zshrc` 之中编写命令是否存在的条件语句，比如「仅当命令存在时加载该命令的自动补全」，或者「当 Node.js 存在时输出 Node.js 版本」。通常情况下我们会写出以下三种条件判断方式：

```
[[ command -v node &>/dev/null ]] && node -v
[[ which -a node &>/dev/null ]] && node -v
[[ type node &>/dev/null ]] && node -v

```

但是在 zsh 中，还有一种速度更快的判断命令存在的方法：

```
(( $+commands[node] )) && node -v

```

zsh 提供了一个数组元素查找语法 `$+array[item]` （元素存在则返回 1 否则返回 0），同时 zsh 也维护了一个命令数组 `$commands`，在数组中检索元素比调用 `which`、`type`、`command -v` 命令都要快许多。

### 变量字符串查找

在 `.zshrc` 中鲜少需要用到这样的语法，不过依然存在一些 case，比如为了避免向 `$FPATH` 中重复添加 Homebrew 的自动补全，提前检查 `$FPATH` 中是否已经包含了 Homebrew 的路径。一般常见的写法都涉及到 `echo` 和 `grep` ：

```
[[ $(echo $FPATH | grep "/usr/local/share/zsh/site-functions") ]] && echo "homebrew exists in fpath"

```

但是在 zsh 中我们不需要 `grep` 也可以实现同样的功能：

```
(( $FPATH[(I)/usr/local/share/zsh/site-functions] )) && echo "homebrew exists in fpath"

```

zsh 内置了在变量中匹配字符串的语法：`$variable[(i)keyword]` 和 `$variable[(I)keyword]`，前者是从左往右寻找、后者是从右往左寻找，返回值为第一个匹配的首字符位置，当没有匹配时返回值则是变量的最终位置，也就是说当找不到匹配时 `(i)` 会返回字符串的长度、而 `(I)` 会返回 0。因此只需要从右往左寻找、判断返回值是否为 0 即可，搭配将数字转化为布尔值的 `(( ))` 就可以写出又快又漂亮的条件语句。

### 变量字符串替换

当需要截断或者替换字符串时，大部分人第一时间会想到 `sed` ，因当此需要替换变量中的字符时自然而然的会使用 `echo | sed`。比如，在 macOS 中主机名 `$HOST` 变量通常以 `.local` 结尾：

```
$ echo $HOST

Sukka-MBP.local

```

如果要显示 `Sukka-MBP` （在 prompt 中常常会用到）就需要写成：

```
$ echo $HOST | sed -e "s/.local//"

Sukka-MBP

```

但是，强大的 zsh 内置了简单的变量字符串替换语法，使用下述命令可以达到相同的效果：

```
$ echo ${HOST/.local/}

Sukka-MBP

$ echo ${HOST/.local/.foxtail}

Sukka-MBP.foxtail

```

其它优化手段
------

### 禁用多余的插件

oh my zsh 在 Wiki 里说「Add wisely, as too many plugins slow down shell startup」。通过 profiling 可以发现一些插件（如 `git` 插件）执行耗时也不短。考虑到 oh my zsh 内置的 `git` 插件只是一些 alias、大部分我都用不到，因此将其从 `plugins` 数组中移除。

### 避免产生子进程

在 shell 中有不少语法会产生子进程。由于这些不受控制的子进程可能会产生其它子进程、从而导致潜在的巨大开销。常见的会产生子进程的语法有是 `eval` 和 Command substitution，在编写 `.zshrc` 时应该尽量避免使用它们。

例如，Homebrew 是通过 Ruby —— 一种没有性能优势的语言编写的，而且 Homebrew 的开发者甚至因为不会翻转二叉树而错失了 Google 的 offer（想必大家大体可以猜得出 Homebrew 中的负优化），因此在 zsh 启动时产生一个子进程运行 Homebrew 将是不能忍受的，绝大部分使用 Homebrew 的人都不会改变 Homebrew 的路径，因此与其在 `.zshrc` 中使用 `$(brew --prefix)`，不如直接将命令执行的结果（`/usr/local`）直接写在 `.zshrc` 中。

### 启用 ZSH_DISABLE_COMPFIX

oh my zsh 内置了安全功能、避免 oh my zsh 插件使用不安全的目录和文件，但是这意味着插件在加载时需要通过一系列 security checker。通过禁用安全功能 （`export ZSH_DISABLE_COMPFIX="true"`）可以使 zsh 启动速度加快 0.06s。微不足道，但值得一试。

针对 macOS 的优化
------------

### path_helper

和 Linux 不同，在 macOS 上 zsh 启动序列的第一项为 `/etc/zprofile` 而不是 `~/.zprofile`。macOS 通过 `/etc/zprofile` 来调用 `path_helper`：

```
$ cat /etc/zprofile



if [ -x /usr/libexec/path_helper ]; then
  eval `/usr/libexec/path_helper -s`
fi

if [ "${BASH-no}" != "no" ]; then
  [ -r /etc/bashrc ] && . /etc/bashrc
fi

```

而 `path_helper` 又会读取 `/etc/paths` 、`/etc/paths/d`、`etc/manpaths` 和 `etc/manpaths.d`、并将其添加到 `$PATH` 和 `$MANPATH` 变量中。通过 `path_helper` macOS 提供了一种快速在不同 shell 中共享 `$PATH` 和 `$MANPATH` 的方法。过去，`path_helper` 是一个 [运行速度很慢的 shell 脚本](https://mjtsai.com/blog/2009/04/01/slow-opening-terminal-windows) 以至于有人制作了 [专门的 patch](https://gist.github.com/mkhl/123525)、甚至 [使用 Perl](https://github.com/mgprot/path_helper) 重写了一个替代品。不过 macOS 意识到了这个问题，现在 `path_helper` 不再是一个脚本而是一个预编译好的二进制文件。如果你通过 profiling 发现 `path_helper` 有在拖累 zsh 启动，那么可以考虑放弃使用 `/etc/paths/d`、而是在 `.zshrc` 中直接维护 `$PATH`。

### login process

默认在启动、终端登陆 shell 时会触发 macOS 的 `login -fp username`。这一操作会调用 `syslog()` 函数向 `/var/log/asl` 写入日志、并读取上一次登录记录、以 `Last login` 的形式显示出来。你可以使用下述命令证实这一行为：

```
ps -ef | grep login

```

如果想要通过减少日志写入来加快 zsh 启动速度，可以修改 `etc/asl.conf` 配置文件中定义的日志等级。

不少文章也提到，修改 iTerm2 设置中的 `Login Command` 为 `/bin/zsh` 可以加快 zsh 启动速度，本质上也是绕过了上述读取和写入日志的环节。

> ASL 即 **A**pple **S**ystem **L**og，macOS 10.12 起被弃用，但是仍有系统组件在使用这一接口。

尾声
--

经过一系列优化，我终于让 zsh 启动速度提升了十倍，速度甚至不亚于 fish 等以性能著称的 shell：

```
$ for i in $(seq 1 5); do /usr/bin/time /bin/zsh -i -c exit; done

        0.14 real         0.08 user         0.05 sys
        0.12 real         0.07 user         0.04 sys
        0.12 real         0.07 user         0.04 sys
        0.13 real         0.07 user         0.04 sys
        0.13 real         0.07 user         0.04 sys

```

如果对我的 `.zshrc` 文件感兴趣，可以 [前往 GitHub 查看我开源的 dotfiles](https://github.com/SukkaW/dotfiles/blob/master/_zshrc/macos.zshrc)。