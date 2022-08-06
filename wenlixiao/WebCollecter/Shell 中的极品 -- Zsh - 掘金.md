> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6844903636967882760)

> 用了很久的 zsh，一直感叹它的强大与便捷，很早就打算记录一篇安装和使用 zsh 的心路历程，工作忙一直在往后拖 (借口 -。

[![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/7/12/1648c29f3e02d8b4~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)](https://link.juejin.cn/?target=https%3A%2F%2Fblog.richardweitech.cn%2F2018%2F07%2F01%2Fuse-zsh%2Ftitle.png "https://blog.richardweitech.cn/2018/07/01/use-zsh/title.png")

用了很久的 `zsh`，一直感叹它的强大与便捷，很早就打算记录一篇安装和使用 `zsh` 的心路历程，工作忙一直在往后拖 (借口 -。-)

[](#zsh-是什么？ "#zsh-是什么？")zsh 是什么？
---------------------------------

> Z Shell(Zsh) 是一种 Unix shell，它可以用作为交互式的登录 shell，也是一种强大的 shell 脚本命令解释器。Zsh 可以认为是一种 Bourne shell 的扩展，带有数量庞大的改进，包括一些 bash、ksh、tcsh 的功能。

简而言之，就是 `shell` 脚本语言的一种扩展与加强。

[](#zsh-有哪些功能？ "#zsh-有哪些功能？")zsh 有哪些功能？
---------------------------------------

> [From Wiki](https://link.juejin.cn/?target=https%3A%2F%2Fzh.wikipedia.org%2Fwiki%2FZ_shell "https://zh.wikipedia.org/wiki/Z_shell")

*   开箱即用、可编程的命令行补全功能可以帮助用户输入各种参数以及选项。
*   在用户启动的所有 shell 中共享命令历史。
*   通过扩展的文件通配符，可以不利用外部命令达到 find 命令一般展开文件名。
*   改进的变量与数组处理。
*   在缓冲区中编辑多行命令。
*   多种兼容模式，例如使用 / bin/sh 运行时可以伪装成 Bourne shell。
*   可以定制呈现形式的提示符；包括在屏幕右端显示信息，并在键入长命令时自动隐藏。
*   可加载的模块，提供其他各种支持：完整的 TCP 与 Unix 域套接字控制，FTP 客户端与扩充过的数学函数。
*   完全可定制化。

[](#zsh-的安装与使用 "#zsh-的安装与使用")zsh 的安装与使用
---------------------------------------

> 以下例子全部基于 centOS 7.4 实现, macOS 类似

通用 `yum` 安装:

```
sudo yum update && sudo yum -y install zsh
复制代码

```

安装成功后，键入 `zsh --version` 查看是否安装成功，成功会有如下输出（不同的版本、平台会有不同的输出）:

```
zsh 5.0.2 (x86_64-redhat-linux-gnu)
复制代码

```

`zsh` 安装成功之后，我们就可以将默认的 `shell` 切换至 `zsh`，在做这个操作之前，我们可以确认一下已经安装好的 `shell` 脚本有哪些：

```
~ cat /etc/shells
/bin/sh
/bin/bash
/sbin/nologin
/usr/bin/sh
/usr/bin/bash
/usr/sbin/nologin
/bin/zsh
复制代码

```

能够清楚的查看已经安装好的 `shell` 脚本列表，当然，验证的方法还有很多，比如直接打印 `shell` 的环境变量:

```
~ echo $SHELL
/bin/zsh
复制代码

```

切换默认的 `shell` 至 `zsh`:

```
~ chsh -s $(which zsh)
Changing shell for root.
chsh: Warning: "/usr/bin/zsh" is not listed in /etc/shells.
Shell changed.
复制代码

```

安装好 `zsh` 之后，他还并不像我们前面所描述的那样这么强大，我们还需要一个很酷的工具 `Oh My Zsh` 来管理和扩展我们的 `zsh`

[](#Oh-My-Zsh "#Oh-My-Zsh")Oh My Zsh
------------------------------------

[官网地址](https://link.juejin.cn/?target=https%3A%2F%2Fohmyz.sh%2F "https://ohmyz.sh/")

官网的描述: `It comes bundled with a ton of helpful functions, helpers, plugins, themes, and a few things that make you shout...`

支持超多的扩展函数、插件以及主题等; 具体有什么用，根据我的使用习惯列出了一个清单:

**Plugins**

*   git: 强大的 git 缩写命令 (默认已开启)
*   zsh-autosuggestions: shell 命令提示，自动补全可能的路径
*   zsh-syntax-highlighting: 特殊命令高亮提示

**Themes**

*   robbyrussell: 默认主题，很好看
*   af-magic
*   agnoster: 暗系主题
*   avit: 界面干净
    
*   **alias**: 超链接别名，特有用，后文会提到如何使用
    

### [](#安装 "#安装")安装

```
1、
git clone https://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
2、
cp ~/.zshrc ~/.zshrc.orig
3、
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
4、
chsh -s /bin/zsh
复制代码

```

安装成功后，重新打开一个新的终端，如果发现界面主题已经改变，代表安装成功配置已经生效了。

**为了保证配置的有效性，我们可以输入简单的命令进行验证:**

```
➜  ~ la
总用量 100K
-rw
-rw-r
-rw-r
-rw-r
drwx
drwx
-rw-r
-rw
drwxr-xr-x  11 root root 4.0K 7月   1 18:04 .oh-my-zsh
drwxr-xr-x   2 root root 4.0K 10月 15 2017 .pip
drwxr
-rw-r
drwx
-rw-r
-rw
-rw-r
-rw
-rw-r
-rw-r
复制代码

```

如果`la` 代替了命令 `ls -a`，恭喜你安装成功。

### [](#主题选择 "#主题选择")主题选择

关于主题的选择，默认的主题 `robbyrussell` 对我来说已经很满足了，如果对主题有更多要求和追求改变，可以从[这里选择相应主题进行切换](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Frobbyrussell%2Foh-my-zsh%2Fwiki%2FThemes "https://github.com/robbyrussell/oh-my-zsh/wiki/Themes")。

这里描述一下如何切换主题:

```
1. 打开配置文件
➜  ~ vim ~/.zshrc
2. 找到主题配置所在行`ZSH_THEME`，切换对应的主题即可
ZSH_THEME="robbyrussell"
3. :wq 保存退出，source ~/.zshrc 使配置文件生效，打开新终端
复制代码

```

### [](#alias-别名的使用 "#alias-别名的使用")alias 别名的使用

在配置文件 `.zshrc` 中，默认有一部分设置别名的例子:

```
# Example aliases
# alias zshconfig="mate ~/.zshrc"
# alias ohmyzsh="mate ~/.oh-my-zsh"
复制代码

```

什么意思呢？很容易理解，为我们的命令设置别名 (简化缩写)

例：当我们要跳转到一个位于 `/home/richardwei/blog` 路径下的博客目录时，我们通常会这么写: `cd /home/richardwei/blog`，如果这个命令我们常用，那么每次去敲击一遍或者寻找一遍会显得比较繁琐，现在我们这样做:

```
➜  ~ vim ~/.zshrc

alias myblog="cd /home/richardwei/blog"



myblog 键入此命令会发现已经进入对应的博客目录了
复制代码

```

对于一些我们常用的命令，但是又比较繁琐且固定的，完全可以使用 `alias` 别名来代替，会有效的提高我们的工作效率。

### [](#插件选择和配置 "#插件选择和配置")插件选择和配置

插件的配置:

```
1. 打开配置文件
➜  ~ vim ~/.zshrc
2. 找到插件配置所在行`plugins`
plugins=(
  git
)
3. 如果需要增加一个插件，只需要:
plugins=(
  git
  zsh-autosuggestions
)
复制代码

```

#### [](#git "#git")git

> 使用缩写代替各种复杂 git 命令

`git` 插件默认已经配置开启，我们可以通过  
`cat ~/.oh-my-zsh/plugins/git/git.plugin.zsh` 命令查看已经配置好的命令 (别名)， 也是最常用的功能之一

例:  
从当前分支检出新分支 `git checkout -b feature/test` => `gco -b feature/test`  
推送到远端 `git push` => `gp`  
推送到远端并建立远程分支 `git push --set-upstream origin test` => `gpsup`  
所有文件改动加入到工作区 `git add .` => `ga .`  
表状、多种颜色、格式化展示提交日志 `git log --graph --pretty='%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --all` => `glola`  
….  
简直就是冗长并且难记命令的克星

#### [](#zsh-autosuggestions "#zsh-autosuggestions")zsh-autosuggestions

> 效率神器，命令自动推断及补全，超级好用

**安装**:

> 仅介绍 oh-my-zsh 下的安装过程
> 
> 1.  克隆代码至本地 oh-my-zsh 插件目录  
>     `git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions`
> 2.  添加到配置文件 vim ~/.zshrc  
>     `plugins=(zsh-autosuggestions)`
> 3.  source ~/.zshrc 打开新的终端界面即可生效

**效果**  
如图: 如果我们曾经输入过 `cd ~/.oh-my-zsh`, 当我们下次输入 `cd ~` 的时候，会自动推荐我们可能需要前往的目录等等，非常好用

[![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/7/12/1648c29f3e55faf3~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)](https://link.juejin.cn/?target=https%3A%2F%2Fblog.richardweitech.cn%2F2018%2F07%2F01%2Fuse-zsh%2Fsuggest.png "https://blog.richardweitech.cn/2018/07/01/use-zsh/suggest.png")

```
zsh-syntax-highlighting复制代码

```

> shell 命令高亮提示

**官网 Demo:**  
Before: [![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/7/12/1648c29f3de065dc~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)](https://link.juejin.cn/?target=https%3A%2F%2Fblog.richardweitech.cn%2F2018%2F07%2F01%2Fuse-zsh%2Fbefore1.png "https://blog.richardweitech.cn/2018/07/01/use-zsh/before1.png")  
After:  [![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/7/12/1648c29f3e17ed0b~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)](https://link.juejin.cn/?target=https%3A%2F%2Fblog.richardweitech.cn%2F2018%2F07%2F01%2Fuse-zsh%2Fafter1.png "https://blog.richardweitech.cn/2018/07/01/use-zsh/after1.png")

Before: [![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/7/12/1648c29f5f082a01~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)](https://link.juejin.cn/?target=https%3A%2F%2Fblog.richardweitech.cn%2F2018%2F07%2F01%2Fuse-zsh%2Fbefore2.png "https://blog.richardweitech.cn/2018/07/01/use-zsh/before2.png")  
After:  [![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/7/12/1648c29f5f21cb94~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)](https://link.juejin.cn/?target=https%3A%2F%2Fblog.richardweitech.cn%2F2018%2F07%2F01%2Fuse-zsh%2Fafter2.png "https://blog.richardweitech.cn/2018/07/01/use-zsh/after2.png")

Before: [![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/7/12/1648c29f6186ca7c~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)](https://link.juejin.cn/?target=https%3A%2F%2Fblog.richardweitech.cn%2F2018%2F07%2F01%2Fuse-zsh%2Fbefore3.png "https://blog.richardweitech.cn/2018/07/01/use-zsh/before3.png")  
After:  [![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/7/12/1648c29f7e407a60~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)](https://link.juejin.cn/?target=https%3A%2F%2Fblog.richardweitech.cn%2F2018%2F07%2F01%2Fuse-zsh%2Fafter3.png "https://blog.richardweitech.cn/2018/07/01/use-zsh/after3.png")

**安装**

1.  克隆代码至本地  
    `git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting`
2.  编辑配置  
    `plugins=(... zsh-syntax-highlighting)`
3.  source ~/.zshrc 打开新的终端界面即可生效