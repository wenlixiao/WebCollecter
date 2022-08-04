> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.shuzhiduo.com](https://www.shuzhiduo.com/A/gVdnMqQp5W/)

> 一、概览

一、概览

1、什么是 repo 文件

repo 文件是 Fedora 中 yum 源（软件仓库）的配置文件，通常一个 repo 文件定义了一个或者多个软件仓库的细节内容，例如我们将从哪里下载需要安装或者升级的软件包，repo 文件中的设置内容将被 yum 读取和应用

2、Yum 的工作原理

YUM 的工作原理并不复杂，每一个 RPM 软件的头（header）里面都会纪录该软件的依赖关系，那么如果可以将该头的内容纪录下来并且进行分析，可以知道每个软件在安装之前需要额外安装 哪些基础软件。也就是说，在服务器上面先以分析工具将所有的 RPM 档案进行分析，然后将该分析纪录下来，只要在进行安装或升级时先查询该纪录的文件，就可 以知道所有相关联的软件。

3、YUM 的基本工作流程

3.1、服务器端

在服务器上面存放了所有的 RPM 软件包，然后以相关的功能去分析每个 RPM 文件的依赖性关系，将这些数据记录成文件存放在服务器的某特定目录内。

3.2、客户端

如果需要安装某个软件时，先下载服务器上面记录的依赖性关系文件 (可通过 WWW 或 FTP 方式)，通过对服务器端下载的纪录数据进行分析，然后取得所有相关的软件，一次全部下载下来进行安装。

4、Yum 配置

4.1、Yum 的两部分 main 和 repository

yum 的配置文件分为两部分：main 和 repository

main：定义了全局配置选项，整个 yum 配置文件应该只有一个 main。常位于 / etc/yum.conf      中。

repository：定义了每个源 / 服务器的具体配置，可以有一到多个。常位于 / etc/yum.repo.d      目录下的各文件中。

yum 的配置方式也分两种：

直接配置 / etc 目录下的 yum.conf 文件，增加 repository 片段

在 / etc/yum.repos.d 目录下增加. repo 文件

4.2、/etc/yum.conf

[main]

cachedir=/var/cache/yum  # cachedir：yum 缓存的目录，yum 在此存储下载的 rpm 包和数据库，一般是 / var/cache/yum。

debuglevel=2   # debuglevel：除错级别，0──10, 默认是 2 貌似只记录安装和删除记录

logfile=/var/log/yum.log  # 日志路径

pkgpolicy=newest

# pkgpolicy： 包的策略。一共有两个选项，newest 和 last，这个作用是如果你设置了多个 repository，而同一软件在不同的 repository 中同时存 在，yum 应该安装哪一个，如果是 newest，则 yum 会安装最新的那个版本。如果是 last，则 yum 会将服务器 id 以字母表排序，并选择最后的那个 服务器上的软件安装。一般都是选 newest。

distroverpkg=centos-release

# 指定一个软件包，yum 会根据这个包判断你的发行版本，默认是 redhat-release，也可以是安装的任何针对自己发行版的 rpm 包。

tolerant=1

# tolerent，也有 1 和 0 两个选项，表示 yum 是否容忍命令行发生与软件包有关的错误，比如你要安装 1,2,3 三个包，而其中 3 此前已经安装了，如果你设为 1, 则 yum 不会出现错误信息。默认是 0。

exactarch=1

# exactarch，有两个选项 1 和 0, 代表是否只升级和你安装软件包 cpu 体系一致的包，如果设为 1，则如你安装了一个 i386 的 rpm，则 yum 不会用 1686 的包来升级。

retries=20 # retries，网络连接发生错误后的重试次数，如果设为 0，则会无限重试。

obsoletes=1 # 这是一个 update 的参数，具体请参阅 yum(8)，简单的说就是相当于 upgrade，允许更新陈旧的 RPM 包

gpgcheck=1 # gpgchkeck= 有 1 和 0 两个选择，分别代表是否进行 gpg 校验，以确定 rpm 包的来源是有效和安全的，如果没有这一项，默认是检查的。

exclude=xxx

#exclude 排除某些软件在升级名单之外，可以用通配符，列表中各个项目要用空格隔开，这个对于安装了诸如美化包，中文补丁的朋友特别有用。

keepcache=[1 or 0]

#设置 keepcache=1，yum 在成功安装软件包之后保留缓存的头文件 (headers) 和软件包。默认值为 keepcache=0 不保存

reposdir=[包含 .repo 文件的目录的绝对路径]

# 该选项用户指定 .repo 文件的绝对路径。.repo 文件包含软件仓库的信息 (作用与 /etc/yum.conf 文件中的 [repository] 片段相同)。

4.3、/etc/yum.repo.d/xx.repo

这个字段其实也可以在 yum.conf 里面直接配置

[serverid] # 软件源 / 仓库名，必须有一个独一无二的名称，如果重复，用 enabled 测试是后面覆盖前面

name=Some name for this server

# name，是对 repository 的描述，支持像 $releasever $basearch 这样的变量，

# 可以写成【name=Fedora Core $releasever - $basearch - Released Updates】

$ releasever 变量定义了发行版本，通常是 8，9，10 等数字，$basearch 变 量定义了系统的架构，可以是 i386、x86_64、ppc 等值

# 这两个变量根据当前系统的版本架构不同而有不同的取值，这可以方便 yum 升级的时候选择 适合当前系统的软件包，以下同

baseurl=url://path/to/repository/

#baseurl 是服务器设置中最重要的部分，只有设置正确，才能从上面获取软件。它的格式是：

baseurl=url://server1/path/to/repository/

url://server2/path/to/repository/

url://server3/path/to/repository/

# 其中 url 支持的协议有 http:// ftp:// file:// 三种。baseurl 后可以跟多个 url，你可以自己改为速度比较快的镜像站

# 但 baseurl 只能有一个，也就是说不能像如下格式：

baseurl=url://server1/path/to/repository/

baseurl=url://server2/path/to/repository/

baseurl=url://server3/path/to/repository/

其中 url 指向的目录必须是这个 repository header 目录的上一级，它也支持 $releasever $basearch 这样的变量。

#mirrorlist=http://mirrors.fedoraproject.org/mirrorlist?repo=fedora-$releasever&arch=$basearch

#上面的这一行是指定一个镜像服务器的地址列表，通常是开启的，本例中加了注释符号禁用了，我们可以试试，将 $releasever 和 $basearch 替换成自己对应的版本和架构，例如 10 和 i386，在浏览器中打开，我们就能看到一长串镜可用的镜像服务器地址列表。

url 之后可以加上多个选项，如 gpgcheck、exclude、failovermethod 等，比如：

gpgcheck=1 # 是否进行 gpg 校验

exclude=gaim # 排除某些软件在升级名单之外

#其中 gpgcheck，exclude 的含义和 [main] 部分相同，但只对此服务器起作用

failovermethod=priority

#failovermethode 有两个选项 roundrobin 和 priority，意思分别是有多个 url 可供选择时，yum 选择的次序，roundrobin 是随机选择，如果连接失 败则使用下一个，依次循环，priority 则根据 url 的次序从第一个开始。如果不指明，默认是 roundrobin。

enabled=[1 or 0]

#当某个软件仓库被配置成 enabled=0 时，yum 在安装或升级软件包时不会将该仓库做为软件包提供源。使用这个选项，可以启用或禁用软件仓库。

#通过 yum 的 --enablerepo=[repo_name] 和 --disablerepo=[repo_name] 选项，或者通过 PackageKit 的 "添加 / 删除软件" 工具，也能够方便地启用和禁用指定的软件仓库

4.3、几个变量

$releasever：发行版的版本，从 [main] 部分的 distroverpkg 获取，如果没有，则根据 redhat-release 包进行判断。

$arch：cpu 体系，如 i686,athlon 等

$basearch：cpu 的基本体系组，如 i686 和 athlon 同属 i386，alpha 和 alphaev6 同属 alpha。

4.4、导入每个 reposity 的 GPG key

前面说过，yum 可以使用 gpg 对包进行校验，确保下载包的完整性，所以我们先要到各个 repository 站点找到 gpg key，一般都会放在首页的醒目位置，一些名字诸如 RPM-GPG-KEY.txt 之类的纯文本文件，把它们下载，然后用 rpm --import xxx.txt 命令将它们导入，最好把发行版自带 GPG-KEY 也导入，rpm --import /usr/share/doc/redhat-release-*/RPM-GPG-KEY 官方软件升级用的上。

二、epel

1、epel 是什么

如果既想获得 RHEL 的高质量、高性能、高可靠性，又需要方便易用 (关键是免费) 的软件包更新功能，那么 Fedora Project 推出的 EPEL(Extra Packages for Enterprise Linux)正好适合你。EPEL(http://fedoraproject.org/wiki/EPEL) 是由 Fedora 社区打造，为 RHEL 及衍生发行版如 CentOS、Scientific Linux 等提供高质量软件包的项目。

2、如何获取 EPEL 的软件包

EPEL 包含一个叫做 ‘epel-release’ 的包，其中包含了用于软件包签名的 gpg 密钥和软件源的信息。安装这个包到您的企业版 Linux 上之后，您将可以通过使用类似于 yum 的工具来安装软件包和它们的依赖。在默认情况下，EPEL 仓库的稳定版本是开启的。除了 epel-release 源，还有一个叫做 ‘epel-testing’ 仓库 包含尚未被视作稳定的软件，请自行斟酌开启的风险。

CentOS 用户可以直接通过 yum install epel-release 安装并启用 EPEL 源。

3、使用心得：

1、不用去换原来 yum 源，安装后会产生新 repo

2、epel 会有很多源地址，如果一个下不到，会去另外一个下

3、更新时如果下载的包不全，就不会进行安装。这样的话，依赖关系可以保重

4、安装 yum install yum-priorities

Yum Priorities 插件可以用来强制保护源。它通过给各个源设定不同的优先级，使得系统管理员可以将某些源（比如 Linux 发行版的官方源）设定为最高优先级，从而保证系统的稳定性（同时也可能无法更新到其它源上提供的软件最新版本）。

三、Yum 源更换

1、备份 / etc/yum.repos.d/CentOS-Base.repo

2、下载对应版本 repo 文件, 放入 / etc/yum.repos.d/(操作前请做好相应备份)

Centos7:

wget -P /etc/yum.repos.d http://mirrors.163.com/.help/CentOS7-Base-163.repo

3. 运行以下命令生成缓存

yum clean all

yum makecache

四、Yum 命令

yum 命令选项

--nogpgcheck：禁止进行 gpgcheck

-y: 自动回答为 “yes”

-q：静默模式

--disablerepo=repoidglob：临时禁用此处指定的 repo

--enablerepo=repoidglob：临时启用此处指定的 repo

--noplugins：禁用所有插件  
          yum 源列表

yum repolist [all|enabled|disabled]: 显示仓库列表

yum grouplist: 显示包组

yum list {available|installed|updates} : 显示包列表

yum list vsftpd* 显示和 vsftpd 匹配的包

yum 安装卸载

yum install package

yum restall package: 重做

yum update package：更新包

yum check-update

yum remove package1 [package2]

包组的安装基本和包的安装类似，只是在 install，restall 等操作前加上 group 即可。比如：yum -y groupinstall "Development Tools", 如果有空格，要使用双引号包括。

如果在安装系统时候，没有安装桌面，则可以使用此命令安装：yum -y groupinstall "GNOME Desktop" 即可安装图形界面

yum 查询

yum info 查看程序包信息

yum provides feature1

yum search xxx ：搜索带有某个关键字的安装包

yum 缓存

yum makecache ：构建缓存

yum clean all：清除所有缓存

yum 历史

yum history：显示 yum 操作历史，是按照 / var/log/yum.log 进行的查找

yum history info 6 查看第六条信息

yum history undo 6：撤销第六步，如果第六步是安装，则执行此命令，将删除第六步所安装的程序，。如果第六步是卸载，那么执行此命令，则进行安装卸载掉的程序

yum history redo 6：重做第六步

五、国内开源镜像站

网易 (http://mirrors.163.com/)

阿里 (https://opsx.alibaba.com/mirror)

清华 (https://mirror.tuna.tsinghua.edu.cn/)

六、参考文档

yum 的 repo 文件详解、以及 epel 简介、yum 源的更换 (https://www.cnblogs.com/nineep/p/6795692.html)

yum 配置与使用 (很详细) (https://www.cnblogs.com/xiaochaohuashengmi/archive/2011/10/09/2203916.html)

CentOS 7.0 本地 yum 源地址及 配置 yum 地址优先级 (https://blog.csdn.net/tantexian/article/details/38895449)

yum 源配置及详解 (https://blog.csdn.net/qq_27754983/article/details/73693061)

Linux man pages online (http://www.man7.org/linux/man-pages/index.html)  
————————————————  
版权声明：本文为 CSDN 博主「heavyfish」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。  
原文链接：https://blog.csdn.net/heavyfish/article/details/82808970

[yum 和 epel 的详解的更多相关文章](https://www.shuzhiduo.com/R/gVdnMqQp5W/)
----------------------------------------------------------------

1.  [yum 的 repo 文件详解、以及 epel 简介、yum 源的更换、常用 yum 命令](https://www.shuzhiduo.com/A/Gkz1w8ZZ5R/)
    
    https://www.cnblogs.com/nineep/p/6795692.html       yum 的 repo 文件详解. 以及 epel 简介. yum 源的更换 常用命令如下: yum list  ...
    
2.  [yum 的 repo 文件详解、以及 epel 简介、yum 源的更换](https://www.shuzhiduo.com/A/6pdDV8aRJw/)
    
    一. 什么是 repo 文件        repo 文件是 Fedora 中 yum 源 (软件仓库) 的配置文件, 通常一个 repo 文件定义了一个或者多个软件仓库的细节内容, 例如我们将从哪里下载需要安装或者升级的软件包 ...
    
3.  [企业级本地 yum 源配置方案详解](https://www.shuzhiduo.com/A/gGdXvVapJ4/)
    
    因目前企业生产网络禁止联网, 对于使用 Linux 的我们来说, 非常不方便, 想要使用 yum 源都很困难, 挂 dvd 又不能完全满足要求, 所以自建一个企业级的 yum 源, 定时从公网同步到本地, 然后生产网络直接配置在本 ...
    
4.  [yum 源配置及详解](https://www.shuzhiduo.com/A/E35ppAa85v/)
    
      红帽系列中, 进行软件安装可以有三种方法, 编译安装, rpm 包安装, 和 yum 源安装. 其中 yum 方法安装最简单, 因为它可以自动解决软件包之间的依赖关系... 一. 常用 yum 源 yum 源可以来源于多种文件 ...
    
5.  [yum 和 rpm 命令详解](https://www.shuzhiduo.com/A/q4zV0ajW5K/)
    
    rpm, 全称 RPM Package Manager, 是 RedHat 发布的, 针对特定硬件, 已经编译好的软件包. 安装之后就可以使用, 不需要自行编译, 以及之前对软件和硬件的检测, 目录的配置等动作. yum, ...
    
6.  [Red Hat Enterprise Linux(RHEL) 中 yum 的 repo 文件详解](https://www.shuzhiduo.com/A/pRdByXae5n/)
    
    Yum(全称为 Yellow dog Updater, Modified) 是一个在 Fedora 和 RedHat 以及 CentOS 中的 Shell 前端软件包管理器. 基于 RPM 包管理, 能够从指定的服务器自动下载 ...
    
7.  [yum 是什么？repo 文件详解，epel 简介，yum 源的更换，repo 和 epel 区别](https://www.shuzhiduo.com/A/B0zqL8YXdv/)
    
    yum 是什么? repo 文件详解, epel 简介, yum 源的更换, repo 和 epel 区别 简单概括: repo 和 epel 的关系 repo 是配置源的, 即配置从哪里下载包 (以及依赖关系) 的. epel 是作为桥 ...
    
8.  [Web 性能压力测试工具之 Siege 详解](https://www.shuzhiduo.com/A/ZOJP4Mw2Jv/)
    
    Siege 是一款开源的压力测试工具, 设计用于评估 WEB 应用在压力下的承受能力. 可以根据配置对一个 WEB 站点进行多用户的并发访问, 记录每个用户所有请求过程的相应时间, 并在一定数量的并发访问下重复进行. s ...
    
9.  [yum 命令详解 - yum 仓库配置文件详解](https://www.shuzhiduo.com/A/6pdDN4bKJw/)
    
    yum 安装的优点 1. 必须得有网络, 通过网络获取软件. 2. 管理 rpm 包 3. 自动解决依耐 4. 命令简单好用 5. 生产最佳实践 yum 命令详解 # linux 安装软件的三种方式 1.rpm 安装 2. 源 ...
    

随机推荐
----

1.  [快速搭建 Redis 缓存数据库](https://www.shuzhiduo.com/A/Ae5R9p9YJQ/)
    
    之前一篇随笔——Redis 安装及主从配置已经详细的介绍过 Redis 的安装于配置. 本文要讲的是如何在已经安装过 Redis 的机器上快速的创建出一个新的 Redis 缓存数据库. 一. 环境介绍 1) Linux ...
    
2.  [关于 div 居中](https://www.shuzhiduo.com/A/VGzlWma7db/)
    
    margin : 100px; margin-left: auto; margin-right: auto; 这样子设置 css 样式就可以实现一个 div 居中
    
3.  [也用 Log4Net 之走进 Log4Net （四）](https://www.shuzhiduo.com/A/MAzA2NAoJ9/)
    
    转载地址: http://www.cnblogs.com/dragon/archive/2005/03/24/124254.html 我是转的别人的内容, 我觉得他写的非常好, 所以我把其中三分之二转了过来 ...
    
4.  [android 点滴之标准 SD 卡状态变化事件广播接收者的注冊](https://www.shuzhiduo.com/A/RnJWGmAYJq/)
    
    眼下最完整的, 须要注冊的动作匹配例如以下: IntentFilter intentFilter = new IntentFilter(Intent.ACTION_MEDIA_MOUNTED); int ...
    
5.  [数学常数 e 的含义](https://www.shuzhiduo.com/A/8Bz8WKQydx/)
    
    转载:   http://www.ruanyifeng.com/blog/2011/07/mathematical_constant_e.html 作者: 阮一峰 日期: 2011 年 7 月 9 日 1. ...
    
6.  [(转) [教程] Unity3D 中角色的动画脚本的编写（一）](https://www.shuzhiduo.com/A/gVdnEoLQ5W/)
    
    ps: 这两天研究 unity3d, 对动画处理特别迷糊, 不知 FBX 导入以后, 接下来应该怎么操作, 看到这篇文章, 感觉非常好, 讲解的很详细. 已有好些天没写什么了, 今天想起来该写点东西了. 这次我所介绍的内容 ...
    
7.  [uva 719 Glass Beads（后缀自动机）](https://www.shuzhiduo.com/A/kmzLorNGdG/)
    
    [题目链接] https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=524&am ...
    
8.  [HTML 确认密码](https://www.shuzhiduo.com/A/LPdoDWA2z3/)
    
    html 确认密码   今天准备分享一个小知识点, 就是确认登录界面 <body><form > 输入户名: <input type="text" name ...
    
9.  [【python】gevent 协程例子](https://www.shuzhiduo.com/A/LPdoqx88J3/)
    
    说在前面: 用协程还是多线程需要仔细考量. 我在做实验时请求了 100w 个 ip, 分别用 pool 为 1000 的协程和 64 个线程来跑, 结果是多线程的速度是协程的 10 倍以上. 一个简单的协程例子 #!/usr/bi ...
    
10.  [[Centos7] 无法访问配置好的 nginx](https://www.shuzhiduo.com/A/lk5axxPZJ1/)
    
    Centos7 无法访问配置好的 nginx 临时生效 # 重启虚拟机, 将失效 iptables -I INPUT -p TCP --dport 80 -j ACCEPT 永久有效 # 在防火墙中开放 80 ...