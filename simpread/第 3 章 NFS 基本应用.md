> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/f-ck-need-u/p/7305755.html)

1.1 概述
======

类似 ext 家族、xfs 格式的本地文件系统，它们都是通过单个文件名称空间 (name space) 来包含很多文件，并提供基本的文件管理和空间分配功能。而文件是存放在文件系统中 (上述名称空间内) 的单个命名对象，每个文件都包含了文件实际数据和属性数据。但是，这些类型的文件系统和其内文件都是存放在本地主机上的。

实际上，还有网络文件系统。顾名思义，就是跨网络的文件系统，将远程主机上的文件系统 (或目录) 存放在本地主机上，就像它本身就是本地文件系统一样。在 Windows 环境下有 cifs 协议实现的网络文件系统，在 Unix 环境下，最出名是由 NFS 协议实现的 NFS 文件系统。

NFS 即 network file system 的缩写，nfs 是属于用起来非常简单，研究起来非常难的东西。相信，使用过它或学过它的人都不会认为它的使用有任何难点，只需将远程主机上要共享给客户端的目录导出 (export)，然后在客户端上挂载即可像本地文件系统一样。到目前为止，nfs 已经有 5 个版本，NFSv1 是未公布出来的版本，v2 和 v3 版本目前来说基本已经淘汰，v4 版本是目前使用最多的版本，nfsv4.1 是目前最新的版本。

1.2 RPC 不可不知的原理
===============

要介绍 NFS，必然要先介绍 RPC。RPC 是 remote procedure call 的简写，人们都将其译为 "远程过程调用"，它是一种框架，这种框架在大型公司应用非常多。而 NFS 正是其中一种，此外 NIS、hadoop 也是使用 rpc 框架实现的。

1.2.1 RPC 原理
------------

所谓的 remote procedure call，就是在本地调用远程主机上的 procedure。以本地执行 "cat -n ~/abc.txt" 命令为例，在本地执行 cat 命令时，会发起某些系统调用 (如 open()、read()、close() 等)，并将 cat 的选项和参数传递给这些函数，于是最终实现了文件的查看功能。在 RPC 层面上理解，上面发起的系统调用就是 procedure，每个 procedure 对应一个或多个功能。而 rpc 的全名 remote procedure call 所表示的就是实现远程 procedure 调用，让远程主机去调用对应的 procedure。

上面的 cat 命令只是本地执行的命令，如何实现远程 cat，甚至其他远程命令？通常有两种可能实现的方式：

(1). 使用 ssh 类的工具，将要执行的命令传递到远程主机上并执行。但 ssh 无法直接调用远程主机上 cat 所发起的那些系统调用 (如 open()、read()、close() 等)。

(2). 使用网络 socket 的方式，告诉远程服务进程要调用的函数。但这样的主机间进程通信方式一般都是 daemon 类服务，daemon 类的客户端 (即服务的消费方) 每调用一个服务的功能，都需要编写一堆实现网络通信相关的代码。不仅容易出错，还比较复杂。

而 rpc 是最好的解决方式。rpc 是一种框架，在此框架中已经集成了网络通信代码和封包、解包方式 (编码、解码)。以下是 rpc 整个过程，以 cat NFS 文件系统中的 a.sh 文件为例。

 ![](https://images2017.cnblogs.com/blog/733013/201708/733013-20170808105429534-2000458733.png)   ![](https://images2017.cnblogs.com/blog/733013/201708/733013-20170808105434455-2083661237.png)

nfs 客户端执行 cat a.sh，由于 a.sh 是 NFS 文件系统内的文件，所以 cat 会发起一些 procedure 调用 (如 open/read/close)，这些 procedure 对应的 ID 号码和对应的参数会发送给 rpc client(可能是单个 procedure ID，也可能是多个 procedure 组合在一起一次性发送给 rpc client，在 NFSv4 上是后者)，rpc client 会将这些数据进行编码封装 (封装和解封装功能由 stub 代码实现)，封装后的消息称为 "call message"，然后将 call message 通过网络发送给 rpc server，rpc server 会对封装的数据进行解封提取，于是就得到了要调用的 procedures ID 和对应的参数，然后将它们交给 NFS 服务进程，最终调用 procedure ID 对应的 procedure 来执行，并返回结果。NFS 服务发起 procedure 调用后，会得到数据 (可能是数据本身，可能是状态消息等)，于是将返回结果交给 rpc server，rpc server 会将这些数据封装，这部分数据称为 "reply message"，然后将 reply message 通过网络发送给 rpc client，rpc client 解封提取，于是得到最终的返回结果。

从上面的过程可以知道，**rpc 的作用是数据封装，rpc client 封装待调用的 procedure ID 及其参数 (其实还有一个 program ID，关于 program，见下文)，rpc server 封装返回的数据。**

举个更简单的例子，使用 google 搜索时，实现搜索功能的 program ID 以及涉及到的 procedure ID 和要搜索的内容就是 rpc client 封装的对象，也是 rpc server 要解封的对象，搜索的结果则是 rpc server 封装的对象，也是 rpc client 要解封的对象。解封后的最终结果即为 google 搜索的结果。

1.2.2 RPC 工具介绍
--------------

在 CentOS 6/7 上，rpc server 由 rpcbind 程序实现，该程序由 rpcbind 包提供。

```
[root@xuexi ~]# yum -y install rpcbind

[root@xuexi ~]# rpm -ql rpcbind | grep bin/
/usr/sbin/rpcbind
/usr/sbin/rpcinfo

```

其中 rpcbind 是 rpc 主程序，在 rpc 服务端该程序必须处于已运行状态，其**默认监听在 111 端口**。rpcinfo 是 rpc 相关信息查询工具。

对于 rpc 而言，其所直接管理的是 programs，programs 由一个或多个 procedure 组成。这些 program 称为 RPC program 或 RPC service。

如下图，其中 NFS、NIS、hadoop 等称为网络服务，它们由多个进程或程序 (program) 组成。例如 NFS 包括 rpc.nfsd、rpc.mountd、rpc.statd 和 rpc.idmapd 等 programs，其中每个 program 都包含了一个或多个 procedure，例如 rpc.nfsd 这个程序包含了如 OPEN、CLOSE、READ、COMPOUND、GETATTR 等 procedure，rpc.mountd 也主要有 MNT 和 UMNT 两个 procedure。

![](https://images2017.cnblogs.com/blog/733013/201708/733013-20170808105337409-1086742676.png)

对于 RPC 而言，它是不知道 NFS/NIS/hadoop 这一层的，它直接管理 programs。每个 program 启动时都需要找 111 端口的 rpc 服务登记注册，然后 RPC 服务会为该 program 映射一个 program number 以及分配一个端口号。其中每个 program 都有一个唯一与之对应的 program number，它们的映射关系定义在 / etc/rpc 文件中。以后 rpc server 将使用 program number 来判断要调用的是哪个 program 中的哪个 procedure(因为这些都是 rpc client 封装在 "call message" 中的)，并将解包后的数据传递给该 program 和 procedure。

例如只启动 rpcbind 时。

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
[root@xuexi ~]# systemctl start rpcbind.service

[root@xuexi ~]# rpcinfo -p localhost
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

其中第一列就是 program number，第二列 vers 表示对应 program 的版本号，最后一列为 RPC 管理的 RPC service 名，其实就是各 program 对应的称呼。

当客户端获取到 rpc 所管理的 service 的端口后，就可以与该端口进行通信了。**但注意，即使客户端已经获取了端口号，客户端仍会借助 rpc 做为中间人进行通信。也就是说，无论何时，客户端和 rpc 所管理的服务的通信都必须通过 rpc 来完成。之所以如此，是因为只有 rpc 才能封装和解封装数据。**

既然客户端不能直接拿着端口号和 rpc service 通信，那还提供端口号干嘛？这个端口号是为 rpc server 提供的，rpc server 解包数据后，会将数据通过此端口交给对应的 rpc service。

1.3 启动 NFS
==========

NFS 本身是很复杂的，它由很多进程组成。这些进程的启动程序由 nfs-utils 包提供。由于 nfs 是使用 RPC 框架实现的，所以需要先安装好 rpcbind。不过安装 nfs-utils 时会自动安装 rpcbind。

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
[root@xuexi ~]# yum -y install nfs-utils

[root@xuexi ~]# rpm -ql nfs-utils | grep /usr/sbin
/usr/sbin/blkmapd
/usr/sbin/exportfs
/usr/sbin/mountstats
/usr/sbin/nfsdcltrack
/usr/sbin/nfsidmap
/usr/sbin/nfsiostat
/usr/sbin/nfsstat
/usr/sbin/rpc.gssd
/usr/sbin/rpc.idmapd
/usr/sbin/rpc.mountd
/usr/sbin/rpc.nfsd
/usr/sbin/rpc.svcgssd
/usr/sbin/rpcdebug
/usr/sbin/showmount
/usr/sbin/sm-notify
/usr/sbin/start-statd

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

其中以 "rpc." 开头的程序都是 rpc service，分别实现不同的功能，启动它们时每个都需要向 rpcbind 进行登记注册。

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
[root@xuexi ~]# systemctl start nfs.service

[root@xuexi ~]# rpcinfo -p localhost
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100024    1   udp  56229  status
    100024    1   tcp  57226  status
    100005    1   udp  20048  mountd
    100005    1   tcp  20048  mountd
    100005    2   udp  20048  mountd
    100005    2   tcp  20048  mountd
    100005    3   udp  20048  mountd
    100005    3   tcp  20048  mountd
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
    100003    3   udp   2049  nfs
    100003    4   udp   2049  nfs
    100227    3   udp   2049  nfs_acl
    100021    1   udp  48609  nlockmgr
    100021    3   udp  48609  nlockmgr
    100021    4   udp  48609  nlockmgr
    100021    1   tcp  50915  nlockmgr
    100021    3   tcp  50915  nlockmgr
    100021    4   tcp  50915  nlockmgr

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

可以看到，每个 program 都启动了不同版本的功能。其中 nfs program 为 rpc.nfsd 对应的 program，为 nfs 服务的主进程，端口号为 2049。mountd 对应的 program 为 rpc.mountd，它为客户端的 mount 和 umount 命令提供服务，即挂载和卸载 NFS 文件系统时会联系 mountd 服务，由 mountd 维护相关挂载信息。nlockmgr 对应的 program 为 rpc.statd，用于维护文件锁和文件委托相关功能，在 NFSv4 以前，称之为 NSM(network status manager)。nfs_acl 和 status，很显然，它们是访问控制列表和状态信息维护的 program。

再看看启动的相关进程信息。

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
[root@xuexi ~]# ps aux | grep -E "[n]fs|[r]pc"
root        748  0.0  0.0      0     0 ?        S<   Jul26   0:00 [rpciod]
rpc        6127  0.0  0.0  64908  1448 ?        Ss   Jul26   0:00 /sbin/rpcbind -w
rpcuser    6128  0.0  0.0  46608  1836 ?        Ss   Jul26   0:00 /usr/sbin/rpc.statd --no-notify
root       6242  0.0  0.0      0     0 ?        S<   Jul26   0:00 [nfsiod]
root       6248  0.0  0.0      0     0 ?        S    Jul26   0:00 [nfsv4.0-svc]
root      17128  0.0  0.0  44860   976 ?        Ss   02:49   0:00 /usr/sbin/rpc.mountd
root      17129  0.0  0.0  21372   420 ?        Ss   02:49   0:00 /usr/sbin/rpc.idmapd
root      17134  0.0  0.0      0     0 ?        S<   02:49   0:00 [nfsd4]
root      17135  0.0  0.0      0     0 ?        S<   02:49   0:00 [nfsd4_callbacks]
root      17141  0.0  0.0      0     0 ?        S    02:49   0:00 [nfsd]
root      17142  0.0  0.0      0     0 ?        S    02:49   0:00 [nfsd]
root      17143  0.0  0.0      0     0 ?        S    02:49   0:00 [nfsd]
root      17144  0.0  0.0      0     0 ?        S    02:49   0:00 [nfsd]
root      17145  0.0  0.0      0     0 ?        S    02:49   0:00 [nfsd]
root      17146  0.0  0.0      0     0 ?        S    02:49   0:00 [nfsd]
root      17147  0.0  0.0      0     0 ?        S    02:49   0:00 [nfsd]
root      17148  0.0  0.0      0     0 ?        S    02:49   0:00 [nfsd]

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

其中有一项 / usr/sbin/rpc.idmapd 进程，该进程是提供服务端的 uid/gid <==> username/groupname 的映射翻译服务。客户端的 uid/gid <==> username/groupname 的映射翻译服务则由 "nfsidmap" 工具实现，详细说明见下文。

1.4 配置导出目录和挂载使用
===============

1.4.1 配置 nfs 导出目录
-----------------

在将服务端的目录共享 (share) 或者说导出 (export) 给客户端之前，需要先配置好要导出的目录。比如何人可访问该目录，该目录是否可写，以何人身份访问导出目录等。

配置导出目录的配置文件为 / etc/exports 或 / etc/exports.d/*.exports 文件，在 nfs 服务启动时，会自动加载这些配置文件中的所有导出项。以下是导出示例：

> /www    172.16.0.0/16(rw,async,no_root_squash)

其中 / www 是导出目录，即共享给客户端的目录；172.16.0.0/16 是访问控制列表 ACL，只有该网段的客户端主机才能访问该导出目录，即挂载该导出目录；紧跟在主机列表后的括号及括号中的内容定义的是该导出目录对该主机的导出选项，例如 (rw,async,no_root_squash) 表示客户端挂载 / www 后，该目录可读写、异步、可保留 root 用户的权限，具体的导出选项稍后列出。

以下是可接收的几种导出方式：

```
/www1    (rw,async,no_root_squash)  # 导出给所有主机，此时称为导出给world
/www2    172.16.1.1(rw,async)       # 仅导出给单台主机172.16.1.1
/www3    172.16.0.0/16(rw,async) 192.168.10.3(rw,no_root_squash)   # 导出给网段172.16.0.0/16，还导出给单台
                                                                   # 主机192.168.10.3，且它们的导出选项不同
/www4    www.a.com(rw,async)        # 导出给单台主机www.a.com主机，但要求能解析该主机名
/www     *.b.com(rw,async)          # 导出给b.com下的所有主机，要求能解析对应主机名

```

以下是常用的一些导出选项说明，更多的导出选项见 man exports：常见的默认项是：ro,sync,root_squash,no_all_squash,wdelay。

<table border="1" cellspacing="0" cellpadding="0"><tbody><tr><td width="160"><p align="center"><strong>导出选项</strong></p><p align="center"><strong>(</strong><strong>加粗标红为默认)</strong></p></td><td width="810"><p align="center"><strong>选项说明</strong></p></td></tr><tr><td width="160"><p align="center">rw、<strong>ro</strong></p></td><td width="810"><p>导出目录可读写还是只读 (read-only)。</p></td></tr><tr><td width="160"><p align="center"><strong>sync</strong>、async</p></td><td width="810"><p>同步共享还是异步共享。异步时，客户端提交要写入的数据到服务端，服务端接收数据后直接响应客户端，但此时数据并不一定已经写入磁盘中，而同步则是必须等待服务端已将数据写入磁盘后才响应客户端。也就是说，给定异步导出选项时，虽然能提升一些性能，但在服务端突然故障或重启时有丢失一部分数据的风险。</p><p>当然，对于只读 (ro) 的导出目录，设置 sync 或 async 是没有任何差别的。</p></td></tr><tr><td width="160"><p align="center">anonuid</p><p align="center">anongid</p></td><td width="810"><p>此为匿名用户 (anonymous) 的 uid 和 gid 值，默认都为 65534，在 / etc/passwd 和 / etc/shadow 中它们对应的用户名为 nfsnobody。该选项指定的值用于身份映射被压缩时。</p></td></tr><tr><td width="160"><p align="center"><strong>root_squash</strong>、</p><p align="center">no_root_squash</p></td><td width="810"><p>是否将发起请求 (即客户端进行访问时) 的 uid/gid=0 的 root 用户映射为 anonymous 用户。即是否压缩 root 用户的权限。</p></td></tr><tr><td width="160"><p align="center">all_squash</p><p align="center"><strong>no_all_squash</strong></p></td><td width="810"><p>是否将发起请求 (即客户端进行访问时) 的所有用户都映射为 anonymous 用户，即是否压缩所有用户的权限。</p></td></tr></tbody></table>

对于 root 用户，将取 (no_)root_squash 和 (no_)all_squash 的交集。例如，no_root_squash 和 all_squash 同时设置时，root 仍被压缩，root_squash 和 no_all_squash 同时设置时，root 也被压缩。

有些导出选项需要配合其他设置。例如，导出选项设置为 rw，但如果目录本身没有 w 权限，或者 mount 时指定了 ro 挂载选项，则同样不允许写操作。

至于别的导出选项，基本无需去关注。

在配置文件写好要导出的目录后，直接重启 nfs 服务即可，它会读取这些配置文件。随后就可以在客户端执行 mount 命令进行挂载。

例如，exports 文件内容如下：

```
/vol/vol0       *(rw,no_root_squash)
/vol/vol2       *(rw,no_root_squash)
/backup/archive *(rw,no_root_squash)

```

1.4.2 挂载 nfs 文件系统
-----------------

然后去客户端上挂载它们。

```
[root@xuexi ~]# mount -t nfs 172.16.10.5:/vol/vol0 /mp1
[root@xuexi ~]# mount 172.16.10.5:/vol/vol2 /mp2
[root@xuexi ~]# mount 172.16.10.5:/backup/archive /mp3

```

挂载时 "-t nfs" 可以省略，因为对于 mount 而言，只有挂载 nfs 文件系统才会写成 host:/path 格式。当然，除了 mount 命令，nfs-utils 包还提供了独立的 mount.nfs 命令，它其实和 "mount -t nfs" 命令是一样的。

mount 挂载时可以指定挂载选项，其中包括 mount 通用挂载选项，如 rw/ro，atime/noatime，async/sync，auto/noauto 等，也包括针对 nfs 文件系统的挂载选项。以下列出几个常见的，更多的内容查看 man nfs 和 man mount。

<table border="1" cellspacing="0" cellpadding="0"><tbody><tr><td><p align="center"><strong>选项</strong></p></td><td><p align="center"><strong>参数意义</strong></p></td><td><p align="center"><strong>默认值</strong></p></td></tr><tr><td><p align="center"><strong>suid</strong></p><p align="center"><strong>nosuid</strong></p></td><td><p>如果挂载的文件系统上有设置了 suid 的二进制程序，</p><p>使用 nosuid 可以取消它的 suid</p></td><td><p align="center"><strong>suid</strong></p></td></tr><tr><td><p align="center"><strong>rw</strong></p><p align="center"><strong>ro</strong></p></td><td><p>尽管服务端提供了 rw 权限，但是挂载时设定 ro，则还是 ro 权限</p><p><strong>权限取交集</strong></p></td><td><p align="center"><strong>rw</strong></p></td></tr><tr><td><p align="center"><strong>exec/noexec</strong></p></td><td><p>是否可执行挂载的文件系统里的二进制文件</p></td><td><p align="center"><strong>exec</strong></p></td></tr><tr><td><p align="center"><strong>user</strong></p><p align="center"><strong>nouser</strong></p></td><td><p>是否运行普通用户进行档案的挂载和卸载</p></td><td><p align="center"><strong>nouser</strong></p></td></tr><tr><td><p align="center"><strong>auto</strong></p><p align="center"><strong>noauto</strong></p></td><td><p>auto 等价于 mount -a，意思是将 / etc/fstab 里设定的全部重挂一遍</p></td><td><p align="center"><strong>auto</strong></p></td></tr><tr><td><p align="center"><strong>sync</strong></p><p align="center"><strong>nosync</strong></p></td><td><p>同步挂载还是异步挂载</p></td><td><p align="center"><strong>async</strong></p></td></tr><tr><td><p align="center"><strong>atime</strong></p><p align="center"><strong>noatime</strong></p></td><td><p>是否修改 atime，对于 nfs 而言，该选项是无效的，理由见下文</p></td><td></td></tr><tr><td><p align="center"><strong>diratime</strong></p><p align="center"><strong>nodiratime</strong></p></td><td><p>是否修改目录 atime，对于 nfs 而言，该挂载选项是无效的，理由见下文</p></td><td></td></tr><tr><td><p align="center"><strong>remount</strong></p></td><td><p>重新挂载</p></td><td></td></tr></tbody></table>

以下是针对 nfs 文件系统的挂载选项。其中没有给出关于缓存选项 (ac/noac、cto/nocto、lookupcache) 的说明，它们可以直接采用默认值，如果想要了解缓存相关内容，可以查看 man nfs。

<table border="1" cellspacing="0" cellpadding="0"><tbody><tr><td><p align="center"><strong>选项</strong></p></td><td><p align="center"><strong>功能</strong></p></td><td><p align="center"><strong>默认值</strong></p></td></tr><tr><td><p align="center"><strong>fg/bg</strong></p></td><td><p><strong>挂载失败后</strong> mount 命令的行为。默认为 fg，表示挂载失败时将直接报错退出，如果是 bg，</p><p>挂载失败后会创建一个子进程不断在后台挂载，而父进程 mount 自身则立即退出并返回 0 状态码。</p></td><td><p align="center"><strong>fg</strong></p></td></tr><tr><td><p align="center"><strong>timeo</strong></p></td><td><p>NFS 客户端等待下一次重发 NFS 请求的时间间隔，单位为十分之一秒。</p><p>基于 TCP 的 NFS 的默认 timeo 的值为 600(60 秒)。</p></td><td></td></tr><tr><td><p align="center"><strong>hard/soft</strong></p></td><td><p>决定 NFS 客户端当 NFS 请求超时时的恢复行为方式。如果是 hard，将无限重新发送 NFS 请求。</p><p>例如在客户端使用 df -h 查看文件系统时就会不断等待。</p><p>设置 soft，当 retrans 次数耗尽时，NFS 客户端将认为 NFS 请求失败，从而使得 NFS 客户端</p><p>返回一个错误给调用它的程序。</p></td><td><p align="center"><strong>hard</strong></p></td></tr><tr><td><p align="center"><strong>retrans</strong></p></td><td><p>NFS 客户端最多发送的请求次数，次数耗尽后将报错表示连接失败。如果 hard 挂载选项生效，</p><p>则会进一步尝试恢复连接。</p></td><td><p align="center"><strong>3</strong></p></td></tr><tr><td><p align="center"><strong>rsize</strong></p><p align="center"><strong>wsize</strong></p></td><td><p>一次读出 (rsize) 和写入 (wsize) 的区块大小。如果网络带宽大，这两个值设置大一点能提升传</p><p>输能力。最好设置到带宽的临界值。</p><p>单位为字节，大小只能为 1024 的倍数，且最大只能设置为 1M。</p></td><td></td></tr></tbody></table>

注意三点：

(1). 所谓的 soft 在特定的环境下超时后会导致静态数据中断。因此，**仅当客户端响应速度比数据完整性更重要时才使用 soft 选项。**使用基于 TCP 的 NFS(除非显示指定使用 UDP，否则现在总是默认使用 TCP) 或增加 retrans 重试次数可以降低使用 soft 选项带来的风险。

如果真的出现 NFS 服务端下线，导致 NFS 客户端无限等待的情况，可以强制将 NFS 文件系统卸载，卸载方法：

```
umount -f -l MOUNT_POINT

```

其中 "-f" 是强制卸载，"-l" 是 lazy umount，表示将该文件系统从当前目录树中剥离，让所有对该文件系统内的文件引用都强制失效。对于丢失了 NFS 服务端的文件系统，卸载时 "-l" 选项是必须的。

(2). 由于 nfs 的客户端挂载后会缓存文件的属性信息，其中包括各种文件时间戳，所以 **mount 指定时间相关的挂载选项是没有意义的**，它们不会有任何效果，包括 **atime/noatime，diratime/nodiratime**，relatime/norelatime 以及 strictatime/nostrictatime 等。具体可见 man nfs 中 "DATA AND METADATA COHERENCE" 段的 "File timestamp maintainence" 说明，或者见本文末尾的翻译。

(3). 如果是要开机挂载 NFS 文件系统，方式自然是写入到 / etc/fstab 文件或将 mount 命令放入 rc.local 文件中。如果是将 / etc/fstab 中，那么在系统环境初始化 (exec /etc/rc.d/rc.sysinit) 的时候会加载 fstab 中的内容，如果挂载 fstab 中的文件系统出错，则会导致系统环境初始化失败，结果是系统开机失败。所以，要开机挂载 nfs 文件系统，则需要在 / etc/fstab 中加入一个挂载选项 **"_rnetdev" 或 "_netdev"(centos 7 中已经没有 "_rnetdev")**，防止无法联系 nfs 服务端时导致开机启动失败。例如：

> 172.16.10.5:/www    /mnt    nfs    defaults,_rnetdev    0    0

当导出目录后，将在 / var/lib/nfs/etab 文件中写入一条对应的导出记录，这是 nfs 维护的导出表，该表的内容会交给 rpc.mountd 进程，并在必要的时候 (mountd 接受到客户端的 mount 请求时)，将此导出表中的内容加载到内核中，内核也单独维护一张导出表。

1.4.3 nfs 伪文件系统
---------------

服务端导出 / vol/vol0、/vol/vol2 和 / backup/archive 后，其中 vol0 和 vol1 是连在一个目录下的，但它们和 archive 目录没有连在一起，nfs 采用伪文件系统的方式来桥接这些不连接的导出目录。桥接的方式是创建那些未导出的连接目录，如伪 vol 目录，伪 backup 目录以及顶级的伪根，如下图所示。

![](https://images2017.cnblogs.com/blog/733013/201708/733013-20170808104934862-1272713222.png)

当客户端挂载后，每次访问导出目录时，其实都是通过找到伪文件系统 (文件系统都有 id，在 nfs 上伪文件系统的 id 称为 fsid) 并定位到导出目录的。

1.5 showmount 命令
================

使用 showmount 命令可以查看某一台主机的导出目录情况。因为涉及到 rpc 请求，所以如果 rpc 出问题，showmount 一样会傻傻地等待。

主要有 3 个选项。

```
showmount [ -ade]  host
-a：以host:dir格式列出客户端名称/IP以及所挂载的目录。但注意该选项是读取NFS服务端/var/lib/nfs/rmtab文件，
  ：而该文件很多时候并不准确，所以showmount -a的输出信息很可能并非准确无误的
-e：显示NFS服务端所有导出列表。
-d：仅列出已被客户端挂载的导出目录。

```

另外 showmount 的结果是排序过的，所以和实际的导出目录顺序可能并不一致。

例如：

```
[root@xuexi ~]# showmount -e 172.16.10.5
Export list for 172.16.10.5:
/backup/archive *
/vol/vol2       *
/vol/vol0       *
/www            172.16.10.4

```

```
[root@xuexi ~]# showmount -d 172.16.10.5
Directories on 172.16.10.5:
/backup/archive
/vol/vol0
/vol/vol2

```

1.6 nfs 身份映射
============

NFS 的目的是导出目录给各客户端，因此导出目录中的文件在服务端和客户端上必然有两套属性、权限集。

例如，服务端导出目录中某 a 文件的所有者和所属组都为 A，但在客户端上不存在 A，那么在客户端上如何显示 a 文件的所有者等属性。再例如，在客户端上，以用户 B 在导出目录中创建了一个文件 b，如果服务端上没有用户 B，在服务端上该如何决定文件 b 的所有者等属性。

所以，NFS 采用 uid/gid <==> username/groupname 映射的方式解决客户端和服务端两套属性问题。由于服务端只能控制它自己一端的身份映射，所以客户端也同样需要身份映射组件。也就是说，服务端和客户端两端都需要对导出的所有文件的所有者和所属组进行映射。

但要注意，服务端的身份映射组件为 rpc.idmapd，它以守护进程方式工作。而客户端使用 nfsidmap 工具进行身份映射。

**服务端映射时以 uid/gid 为基准，**意味着客户端以身份 B(假设对应 uid=Xb，gid=Yb) 创建的文件或修改了文件的所有者属性时，在服务端将从 / etc/passwd(此处不考虑其他用户验证方式) 文件中搜索 uid=Xb，gid=Yb 的用户，如果能搜索到，则设置该文件的所有者和所属组为此 uid/gid 对应的 username/groupname，如果搜索不到，则文件所有者和所属组直接显示为 uid/gid 的值。

**客户端映射时以 username/groupname 为基准，**意味着服务端上文件所有者为 A 时，则在客户端上搜索 A 用户名，如果搜索到，则文件所有者显示为 A，否则都将显示为 nobody。注意，客户端不涉及任何 uid/gid 转换翻译过程，即使客户端上 A 用户的 uid 和服务端上 A 用户的 uid 不同，也仍显示为用户 A。也就是说，客户端上文件所有者只有两种结果，要么和服务端用户同名，要么显示为 nobody。

因此考虑一种特殊情况，客户端上以用户 B(其 uid=B1) 创建文件，假如服务端上没有 uid=B1 的用户，那么创建文件时提交给服务端后，在服务端上该文件所有者将显示为 B1(注意它是一个数值)。再返回到客户端上看，客户端映射时只简单映射 username，不涉及 uid 的转换，因此它认为该文件的所有者为 B1(不是 uid，而是 username)，但客户端上必然没有用户名为 B1 的用户 (尽管有 uid=B1 对应的用户 B)，因此在客户端，此文件所有者将诡异地将显示为 nobody，其诡异之处在于，客户端上以身份 B 创建的文件，结果在客户端上却显示为 nobody。

综上考虑，强烈建议客户端和服务端的用户身份要统一，且尽量让各 uid、gid 能对应上。

1.7 使用 exportfs 命令导出目录
======================

除了启动 nfs 服务加载配置文件 / etc/exports 来导出目录，使用 exportfs 命令也可以直接导出目录，它无需加载配置文件 / etc/exports，当然 exportfs 也可以加载 / etc/exports 文件来导出目录。实际上，nfs 服务启动脚本中就是使用 exportfs 命令来导出 / etc/exports 中内容的。

例如，CentOS 6 上 / etc/init.d/nfs 文件中，导出和卸载导出目录的命令为：

```
[root@xuexi ~]# grep exportfs /etc/init.d/nfs  
        [ -x /usr/sbin/exportfs ] || exit 5
        action $"Starting NFS services: " /usr/sbin/exportfs -r
        cnt=`/usr/sbin/exportfs -v | /usr/bin/wc -l`
                action $"Shutting down NFS services: " /usr/sbin/exportfs -au
        /usr/sbin/exportfs -r

```

在 CentOS 7 上则如下：

```
[root@xuexi ~]# grep exportfs /usr/lib/systemd/system/nfs.service      
ExecStartPre=-/usr/sbin/exportfs -r
ExecStopPost=/usr/sbin/exportfs -au
ExecStopPost=/usr/sbin/exportfs -f
ExecReload=-/usr/sbin/exportfs -r

```

当然，无论如何，nfsd 等守护进程是必须已经运行好的。

以下是 CentOS 7 上 exportfs 命令的用法。注意， CentOS 7 比 CentOS 6 多一些选项。

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
-a     导出或卸载所有目录。
-o options,...
       指定一系列导出选项(如rw,async,root_squash)，这些导出选项在exports(5)的man文档中有记录。
-i     忽略/etc/exports和/etc/exports.d目录下文件。此时只有命令行中给定选项和默认选项会生效。
-r     重新导出所有目录，并同步修改/var/lib/nfs/etab文件中关于/etc/exports和/etc/exports.d/
       *.exports的信息(即还会重新导出/etc/exports和/etc/exports.d/*等导出配置文件中的项)。该
       选项会移除/var/lib/nfs/etab中已经被删除和无效的导出项。
-u     卸载(即不再导出)一个或多个导出目录。
-f     如果/prof/fs/nfsd或/proc/fs/nfs已被挂载，即工作在新模式下，该选项将清空内核中导出表中
       的所有导出项。客户端下一次请求挂载导出项时会通过rpc.mountd将其添加到内核的导出表中。
-v     输出详细信息。
-s     显示适用于/etc/exports的当前导出目录列表。

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

例如：

**(1). 导出 / www 目录给客户端 172.16.10.6。**

```
exportfs 172.16.10.6:/www

```

**(2). 导出 / www 目录给所有人，并指定导出选项。**

```
exportfs :/www -o rw,no_root_squash

```

**(3). 导出 exports 文件中的内容。**

```
exportfs -a

```

**(4). 重新导出所有已导出的目录。包括 exports 文件中和 exportfs 单独导出的目录。**

```
exportfs -ar

```

**(5). 卸载所有已导出的目录，包括 exports 文件中的内容和 exportfs 单独导出的内容。即其本质为清空内核维护的导出表。**

```
exportfs -au

```

**(6). 只卸载某一个导出目录。**

```
exportfs -u 172.16.10.6:/www

```

1.8 RPC 的调试工具 rpcdebug
======================

在很多时候 NFS 客户端或者服务端出现异常，例如连接不上、锁状态丢失、连接非常慢等等问题，都可以对 NFS 进行调试来发现问题出在哪个环节。NFS 有不少进程都可以直接支持调试选项，但最直接的调试方式是调试 rpc，因为 NFS 的每个请求和响应都会经过 RPC 去封装。但显然，调试 RPC 比直接调试 NFS 时更难分析出问题所在。以下只介绍如何调试 RPC。

rpc 单独提供一个调试工具 rpcdebug。

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
[root@xuexi ~]# rpcdebug -vh
usage: rpcdebug [-v] [-h] [-m module] [-s flags...|-c flags...]
       set or cancel debug flags.
 
Module     Valid flags
rpc        xprt call debug nfs auth bind sched trans svcsock svcdsp misc cache all
nfs        vfs dircache lookupcache pagecache proc xdr file root callback client mount fscache pnfs pnfs_ld state all
nfsd       sock fh export svc proc fileop auth repcache xdr lockd all
nlm        svc client clntlock svclock monitor clntsubs svcsubs hostcache xdr all

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

其中：

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
-v：显示更详细信息
-h：显示帮助信息
-m：指定调试模块，有rpc/nfs/nfsd/nlm共4个模块可调试。
  ：顾名思义，调试rpc模块就是直接调试rpc的问题，将记录rpc相关的日志信息；
  ：调试nfs是调试nfs客户端的问题，将记录nfs客户端随之产生的日志信息；
  ：nfsd是调试nfs服务端问题，将记录nfsd随之产生的日志信息；
  ：nlm是调试nfs锁管理器相关问题，将只记录锁相关信息
-s：指定调试的修饰符，每个模块都有不同的修饰符，见上面的usage中"Valid flags"列的信息
-c：清除或清空已设置的调试flage

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

例如设置调试 nfs 客户端的信息。

```
rpcdebug -m nfs -s all

```

当有信息出现时，将记录到 syslog 中。例如以下是客户端挂载 nfs 导出目录产生的信息，存放在 / var/log/messages 中，非常多，所以排解问题时需要有耐心。

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
Jul 29 11:24:04 xuexi kernel: NFS: nfs mount opts='vers=4,addr=172.16.10.9,clientaddr=172.16.10.3'
Jul 29 11:24:04 xuexi kernel: NFS:   parsing nfs mount option 'vers=4'
Jul 29 11:24:04 xuexi kernel: NFS:   parsing nfs mount option 'addr=172.16.10.9'
Jul 29 11:24:04 xuexi kernel: NFS:   parsing nfs mount option 'clientaddr=172.16.10.3'
Jul 29 11:24:04 xuexi kernel: NFS: MNTPATH: '/tmp/testdir'
Jul 29 11:24:04 xuexi kernel: --> nfs4_try_mount()
Jul 29 11:24:04 xuexi kernel: --> nfs4_create_server()
Jul 29 11:24:04 xuexi kernel: --> nfs4_init_server()
Jul 29 11:24:04 xuexi kernel: --> nfs4_set_client()
Jul 29 11:24:04 xuexi kernel: --> nfs_get_client(172.16.10.9,v4)
Jul 29 11:24:04 xuexi kernel: NFS: get client cookie (0xffff88004c561800/0xffff8800364cd2c0)
Jul 29 11:24:04 xuexi kernel: nfs_create_rpc_client: cannot create RPC client. Error = -22
Jul 29 11:24:04 xuexi kernel: --> nfs4_realloc_slot_table: max_reqs=1024, tbl->max_slots 0
Jul 29 11:24:04 xuexi kernel: nfs4_realloc_slot_table: tbl=ffff88004b715c00 slots=ffff880063f32280 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_realloc_slot_table: return 0
Jul 29 11:24:04 xuexi kernel: NFS: nfs4_discover_server_trunking: testing '172.16.10.9'
Jul 29 11:24:04 xuexi kernel: NFS call  setclientid auth=UNIX, 'Linux NFSv4.0 172.16.10.3/172.16.10.9 tcp'
Jul 29 11:24:04 xuexi kernel: NFS reply setclientid: 0
Jul 29 11:24:04 xuexi kernel: NFS call  setclientid_confirm auth=UNIX, (client ID 578d865901000000)
Jul 29 11:24:04 xuexi kernel: NFS reply setclientid_confirm: 0
Jul 29 11:24:04 xuexi kernel: NFS: <-- nfs40_walk_client_list using nfs_client = ffff88004c561800 ({2})
Jul 29 11:24:04 xuexi kernel: NFS: <-- nfs40_walk_client_list status = 0
Jul 29 11:24:04 xuexi kernel: nfs4_schedule_state_renewal: requeueing work. Lease period = 5
Jul 29 11:24:04 xuexi kernel: NFS: nfs4_discover_server_trunking: status = 0
Jul 29 11:24:04 xuexi kernel: --> nfs_put_client({2})
Jul 29 11:24:04 xuexi kernel: <-- nfs4_set_client() = 0 [new ffff88004c561800]
Jul 29 11:24:04 xuexi kernel: <-- nfs4_init_server() = 0
Jul 29 11:24:04 xuexi kernel: --> nfs4_get_rootfh()
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_type: type=040000
Jul 29 11:24:04 xuexi kernel: decode_attr_change: change attribute=4651240235397459983
Jul 29 11:24:04 xuexi kernel: decode_attr_size: file size=4096
Jul 29 11:24:04 xuexi kernel: decode_attr_fsid: fsid=(0x0/0x0)
Jul 29 11:24:04 xuexi kernel: decode_attr_fileid: fileid=2
Jul 29 11:24:04 xuexi kernel: decode_attr_fs_locations: fs_locations done, error = 0
Jul 29 11:24:04 xuexi kernel: decode_attr_mode: file mode=0555
Jul 29 11:24:04 xuexi kernel: decode_attr_nlink: nlink=23
Jul 29 11:24:04 xuexi kernel: decode_attr_owner: uid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_group: gid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_rdev: rdev=(0x0:0x0)
Jul 29 11:24:04 xuexi kernel: decode_attr_space_used: space used=4096
Jul 29 11:24:04 xuexi kernel: decode_attr_time_access: atime=1501990255
Jul 29 11:24:04 xuexi kernel: decode_attr_time_metadata: ctime=1501989952
Jul 29 11:24:04 xuexi kernel: decode_attr_time_modify: mtime=1501989952
Jul 29 11:24:04 xuexi kernel: decode_attr_mounted_on_fileid: fileid=1
Jul 29 11:24:04 xuexi kernel: decode_getfattr_attrs: xdr returned 0
Jul 29 11:24:04 xuexi kernel: decode_getfattr_generic: xdr returned 0
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_supported: bitmask=fdffbfff:00f9be3e:00000000
Jul 29 11:24:04 xuexi kernel: decode_attr_fh_expire_type: expire type=0x0
Jul 29 11:24:04 xuexi kernel: decode_attr_link_support: link support=true
Jul 29 11:24:04 xuexi kernel: decode_attr_symlink_support: symlink support=true
Jul 29 11:24:04 xuexi kernel: decode_attr_aclsupport: ACLs supported=3
Jul 29 11:24:04 xuexi kernel: decode_server_caps: xdr returned 0!
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_lease_time: file size=90
Jul 29 11:24:04 xuexi kernel: decode_attr_maxfilesize: maxfilesize=18446744073709551615
Jul 29 11:24:04 xuexi kernel: decode_attr_maxread: maxread=131072
Jul 29 11:24:04 xuexi kernel: decode_attr_maxwrite: maxwrite=131072
Jul 29 11:24:04 xuexi kernel: decode_attr_time_delta: time_delta=1 0
Jul 29 11:24:04 xuexi kernel: decode_attr_pnfstype: bitmap is 0
Jul 29 11:24:04 xuexi kernel: decode_attr_layout_blksize: bitmap is 0
Jul 29 11:24:04 xuexi kernel: decode_fsinfo: xdr returned 0!
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: <-- nfs4_get_rootfh() = 0
Jul 29 11:24:04 xuexi kernel: Server FSID: 0:0
Jul 29 11:24:04 xuexi kernel: Pseudo-fs root FH at ffff880064c4ad80 is 8 bytes, crc: 0x62d40c52:
Jul 29 11:24:04 xuexi kernel: 01000100 00000000
Jul 29 11:24:04 xuexi kernel: --> nfs_probe_fsinfo()
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_supported: bitmask=fdffbfff:00f9be3e:00000000
Jul 29 11:24:04 xuexi kernel: decode_attr_fh_expire_type: expire type=0x0
Jul 29 11:24:04 xuexi kernel: decode_attr_link_support: link support=true
Jul 29 11:24:04 xuexi kernel: decode_attr_symlink_support: symlink support=true
Jul 29 11:24:04 xuexi kernel: decode_attr_aclsupport: ACLs supported=3
Jul 29 11:24:04 xuexi kernel: decode_server_caps: xdr returned 0!
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_lease_time: file size=90
Jul 29 11:24:04 xuexi kernel: decode_attr_maxfilesize: maxfilesize=18446744073709551615
Jul 29 11:24:04 xuexi kernel: decode_attr_maxread: maxread=131072
Jul 29 11:24:04 xuexi kernel: decode_attr_maxwrite: maxwrite=131072
Jul 29 11:24:04 xuexi kernel: decode_attr_time_delta: time_delta=1 0
Jul 29 11:24:04 xuexi kernel: decode_attr_pnfstype: bitmap is 0
Jul 29 11:24:04 xuexi kernel: decode_attr_layout_blksize: bitmap is 0
Jul 29 11:24:04 xuexi kernel: decode_fsinfo: xdr returned 0!
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: set_pnfs_layoutdriver: Using NFSv4 I/O
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_maxlink: maxlink=255
Jul 29 11:24:04 xuexi kernel: decode_attr_maxname: maxname=255
Jul 29 11:24:04 xuexi kernel: decode_pathconf: xdr returned 0!
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: <-- nfs_probe_fsinfo() = 0
Jul 29 11:24:04 xuexi kernel: <-- nfs4_create_server() = ffff88007746a800
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_supported: bitmask=fdffbfff:00f9be3e:00000000
Jul 29 11:24:04 xuexi kernel: decode_attr_fh_expire_type: expire type=0x0
Jul 29 11:24:04 xuexi kernel: decode_attr_link_support: link support=true
Jul 29 11:24:04 xuexi kernel: decode_attr_symlink_support: symlink support=true
Jul 29 11:24:04 xuexi kernel: decode_attr_aclsupport: ACLs supported=3
Jul 29 11:24:04 xuexi kernel: decode_server_caps: xdr returned 0!
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_type: type=040000
Jul 29 11:24:04 xuexi kernel: decode_attr_change: change attribute=4651240235397459983
Jul 29 11:24:04 xuexi kernel: decode_attr_size: file size=4096
Jul 29 11:24:04 xuexi kernel: decode_attr_fsid: fsid=(0x0/0x0)
Jul 29 11:24:04 xuexi kernel: decode_attr_fileid: fileid=2
Jul 29 11:24:04 xuexi kernel: decode_attr_fs_locations: fs_locations done, error = 0
Jul 29 11:24:04 xuexi kernel: decode_attr_mode: file mode=0555
Jul 29 11:24:04 xuexi kernel: decode_attr_nlink: nlink=23
Jul 29 11:24:04 xuexi kernel: decode_attr_owner: uid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_group: gid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_rdev: rdev=(0x0:0x0)
Jul 29 11:24:04 xuexi kernel: decode_attr_space_used: space used=4096
Jul 29 11:24:04 xuexi kernel: decode_attr_time_access: atime=1501990255
Jul 29 11:24:04 xuexi kernel: decode_attr_time_metadata: ctime=1501989952
Jul 29 11:24:04 xuexi kernel: decode_attr_time_modify: mtime=1501989952
Jul 29 11:24:04 xuexi kernel: decode_attr_mounted_on_fileid: fileid=1
Jul 29 11:24:04 xuexi kernel: decode_getfattr_attrs: xdr returned 0
Jul 29 11:24:04 xuexi kernel: decode_getfattr_generic: xdr returned 0
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: NFS: nfs_fhget(0:38/2 fh_crc=0x62d40c52 ct=1)
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_type: type=00
Jul 29 11:24:04 xuexi kernel: decode_attr_change: change attribute=4651240235397459983
Jul 29 11:24:04 xuexi kernel: decode_attr_size: file size=4096
Jul 29 11:24:04 xuexi kernel: decode_attr_fsid: fsid=(0x0/0x0)
Jul 29 11:24:04 xuexi kernel: decode_attr_fileid: fileid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_fs_locations: fs_locations done, error = 0
Jul 29 11:24:04 xuexi kernel: decode_attr_mode: file mode=00
Jul 29 11:24:04 xuexi kernel: decode_attr_nlink: nlink=1
Jul 29 11:24:04 xuexi kernel: decode_attr_owner: uid=-2
Jul 29 11:24:04 xuexi kernel: decode_attr_group: gid=-2
Jul 29 11:24:04 xuexi kernel: decode_attr_rdev: rdev=(0x0:0x0)
Jul 29 11:24:04 xuexi kernel: decode_attr_space_used: space used=0
Jul 29 11:24:04 xuexi kernel: decode_attr_time_access: atime=0
Jul 29 11:24:04 xuexi kernel: decode_attr_time_metadata: ctime=1501989952
Jul 29 11:24:04 xuexi kernel: decode_attr_time_modify: mtime=1501989952
Jul 29 11:24:04 xuexi kernel: decode_attr_mounted_on_fileid: fileid=0
Jul 29 11:24:04 xuexi kernel: decode_getfattr_attrs: xdr returned 0
Jul 29 11:24:04 xuexi kernel: decode_getfattr_generic: xdr returned 0
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: NFS: nfs_update_inode(0:38/2 fh_crc=0x62d40c52 ct=2 info=0x26040)
Jul 29 11:24:04 xuexi kernel: NFS: permission(0:38/2), mask=0x1, res=0
Jul 29 11:24:04 xuexi kernel: NFS: permission(0:38/2), mask=0x81, res=0
Jul 29 11:24:04 xuexi kernel: NFS: lookup(/tmp)
Jul 29 11:24:04 xuexi kernel: NFS call  lookup tmp
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_type: type=040000
Jul 29 11:24:04 xuexi kernel: decode_attr_change: change attribute=16540743250786234113
Jul 29 11:24:04 xuexi kernel: decode_attr_size: file size=4096
Jul 29 11:24:04 xuexi kernel: decode_attr_fsid: fsid=(0xf199fcb4fb064bf5/0xa1b7a15af0f7cb47)
Jul 29 11:24:04 xuexi kernel: decode_attr_fileid: fileid=391681
Jul 29 11:24:04 xuexi kernel: decode_attr_fs_locations: fs_locations done, error = 0
Jul 29 11:24:04 xuexi kernel: decode_attr_mode: file mode=01777
Jul 29 11:24:04 xuexi kernel: decode_attr_nlink: nlink=5
Jul 29 11:24:04 xuexi kernel: decode_attr_owner: uid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_group: gid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_rdev: rdev=(0x0:0x0)
Jul 29 11:24:04 xuexi kernel: decode_attr_space_used: space used=4096
Jul 29 11:24:04 xuexi kernel: decode_attr_time_access: atime=1501990260
Jul 29 11:24:04 xuexi kernel: decode_attr_time_metadata: ctime=1501990117
Jul 29 11:24:04 xuexi kernel: decode_attr_time_modify: mtime=1501990117
Jul 29 11:24:04 xuexi kernel: decode_attr_mounted_on_fileid: fileid=391681
Jul 29 11:24:04 xuexi kernel: decode_getfattr_attrs: xdr returned 0
Jul 29 11:24:04 xuexi kernel: decode_getfattr_generic: xdr returned 0
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: NFS reply lookup: 0
Jul 29 11:24:04 xuexi kernel: NFS: nfs_fhget(0:38/391681 fh_crc=0xb4775a3f ct=1)
Jul 29 11:24:04 xuexi kernel: --> nfs_d_automount()
Jul 29 11:24:04 xuexi kernel: nfs_d_automount: enter
Jul 29 11:24:04 xuexi kernel: NFS call  lookup tmp
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_type: type=040000
Jul 29 11:24:04 xuexi kernel: decode_attr_change: change attribute=16540743250786234113
Jul 29 11:24:04 xuexi kernel: decode_attr_size: file size=4096
Jul 29 11:24:04 xuexi kernel: decode_attr_fsid: fsid=(0xf199fcb4fb064bf5/0xa1b7a15af0f7cb47)
Jul 29 11:24:04 xuexi kernel: decode_attr_fileid: fileid=391681
Jul 29 11:24:04 xuexi kernel: decode_attr_fs_locations: fs_locations done, error = 0
Jul 29 11:24:04 xuexi kernel: decode_attr_mode: file mode=01777
Jul 29 11:24:04 xuexi kernel: decode_attr_nlink: nlink=5
Jul 29 11:24:04 xuexi kernel: decode_attr_owner: uid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_group: gid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_rdev: rdev=(0x0:0x0)
Jul 29 11:24:04 xuexi kernel: decode_attr_space_used: space used=4096
Jul 29 11:24:04 xuexi kernel: decode_attr_time_access: atime=1501990260
Jul 29 11:24:04 xuexi kernel: decode_attr_time_metadata: ctime=1501990117
Jul 29 11:24:04 xuexi kernel: decode_attr_time_modify: mtime=1501990117
Jul 29 11:24:04 xuexi kernel: decode_attr_mounted_on_fileid: fileid=391681
Jul 29 11:24:04 xuexi kernel: decode_getfattr_attrs: xdr returned 0
Jul 29 11:24:04 xuexi kernel: decode_getfattr_generic: xdr returned 0
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: NFS reply lookup: 0
Jul 29 11:24:04 xuexi kernel: --> nfs_do_submount()
Jul 29 11:24:04 xuexi kernel: nfs_do_submount: submounting on /tmp
Jul 29 11:24:04 xuexi kernel: --> nfs_xdev_mount()
Jul 29 11:24:04 xuexi kernel: --> nfs_clone_server(,f199fcb4fb064bf5:a1b7a15af0f7cb47,)
Jul 29 11:24:04 xuexi kernel: --> nfs_probe_fsinfo()
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_supported: bitmask=fdffbfff:00f9be3e:00000000
Jul 29 11:24:04 xuexi kernel: decode_attr_fh_expire_type: expire type=0x0
Jul 29 11:24:04 xuexi kernel: decode_attr_link_support: link support=true
Jul 29 11:24:04 xuexi kernel: decode_attr_symlink_support: symlink support=true
Jul 29 11:24:04 xuexi kernel: decode_attr_aclsupport: ACLs supported=3
Jul 29 11:24:04 xuexi kernel: decode_server_caps: xdr returned 0!
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_lease_time: file size=90
Jul 29 11:24:04 xuexi kernel: decode_attr_maxfilesize: maxfilesize=18446744073709551615
Jul 29 11:24:04 xuexi kernel: decode_attr_maxread: maxread=131072
Jul 29 11:24:04 xuexi kernel: decode_attr_maxwrite: maxwrite=131072
Jul 29 11:24:04 xuexi kernel: decode_attr_time_delta: time_delta=1 0
Jul 29 11:24:04 xuexi kernel: decode_attr_pnfstype: bitmap is 0
Jul 29 11:24:04 xuexi kernel: decode_attr_layout_blksize: bitmap is 0
Jul 29 11:24:04 xuexi kernel: decode_fsinfo: xdr returned 0!
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: set_pnfs_layoutdriver: Using NFSv4 I/O
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_maxlink: maxlink=255
Jul 29 11:24:04 xuexi kernel: decode_attr_maxname: maxname=255
Jul 29 11:24:04 xuexi kernel: decode_pathconf: xdr returned 0!
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: <-- nfs_probe_fsinfo() = 0
Jul 29 11:24:04 xuexi kernel: Cloned FSID: f199fcb4fb064bf5:a1b7a15af0f7cb47
Jul 29 11:24:04 xuexi kernel: <-- nfs_clone_server() = ffff88006f407000
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_supported: bitmask=fdffbfff:00f9be3e:00000000
Jul 29 11:24:04 xuexi kernel: decode_attr_fh_expire_type: expire type=0x0
Jul 29 11:24:04 xuexi kernel: decode_attr_link_support: link support=true
Jul 29 11:24:04 xuexi kernel: decode_attr_symlink_support: symlink support=true
Jul 29 11:24:04 xuexi kernel: decode_attr_aclsupport: ACLs supported=3
Jul 29 11:24:04 xuexi kernel: decode_server_caps: xdr returned 0!
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_type: type=040000
Jul 29 11:24:04 xuexi kernel: decode_attr_change: change attribute=16540743250786234113
Jul 29 11:24:04 xuexi kernel: decode_attr_size: file size=4096
Jul 29 11:24:04 xuexi kernel: decode_attr_fsid: fsid=(0xf199fcb4fb064bf5/0xa1b7a15af0f7cb47)
Jul 29 11:24:04 xuexi kernel: decode_attr_fileid: fileid=391681
Jul 29 11:24:04 xuexi kernel: decode_attr_fs_locations: fs_locations done, error = 0
Jul 29 11:24:04 xuexi kernel: decode_attr_mode: file mode=01777
Jul 29 11:24:04 xuexi kernel: decode_attr_nlink: nlink=5
Jul 29 11:24:04 xuexi kernel: decode_attr_owner: uid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_group: gid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_rdev: rdev=(0x0:0x0)
Jul 29 11:24:04 xuexi kernel: decode_attr_space_used: space used=4096
Jul 29 11:24:04 xuexi kernel: decode_attr_time_access: atime=1501990260
Jul 29 11:24:04 xuexi kernel: decode_attr_time_metadata: ctime=1501990117
Jul 29 11:24:04 xuexi kernel: decode_attr_time_modify: mtime=1501990117
Jul 29 11:24:04 xuexi kernel: decode_attr_mounted_on_fileid: fileid=391681
Jul 29 11:24:04 xuexi kernel: decode_getfattr_attrs: xdr returned 0
Jul 29 11:24:04 xuexi kernel: decode_getfattr_generic: xdr returned 0
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: NFS: nfs_fhget(0:40/391681 fh_crc=0xb4775a3f ct=1)
Jul 29 11:24:04 xuexi kernel: <-- nfs_xdev_mount() = 0
Jul 29 11:24:04 xuexi kernel: nfs_do_submount: done
Jul 29 11:24:04 xuexi kernel: <-- nfs_do_submount() = ffff880064fdfb20
Jul 29 11:24:04 xuexi kernel: nfs_d_automount: done, success
Jul 29 11:24:04 xuexi kernel: <-- nfs_d_automount() = ffff880064fdfb20
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_type: type=00
Jul 29 11:24:04 xuexi kernel: decode_attr_change: change attribute=16540743250786234113
Jul 29 11:24:04 xuexi kernel: decode_attr_size: file size=4096
Jul 29 11:24:04 xuexi kernel: decode_attr_fsid: fsid=(0x0/0x0)
Jul 29 11:24:04 xuexi kernel: decode_attr_fileid: fileid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_fs_locations: fs_locations done, error = 0
Jul 29 11:24:04 xuexi kernel: decode_attr_mode: file mode=00
Jul 29 11:24:04 xuexi kernel: decode_attr_nlink: nlink=1
Jul 29 11:24:04 xuexi kernel: decode_attr_owner: uid=-2
Jul 29 11:24:04 xuexi kernel: decode_attr_group: gid=-2
Jul 29 11:24:04 xuexi kernel: decode_attr_rdev: rdev=(0x0:0x0)
Jul 29 11:24:04 xuexi kernel: decode_attr_space_used: space used=0
Jul 29 11:24:04 xuexi kernel: decode_attr_time_access: atime=0
Jul 29 11:24:04 xuexi kernel: decode_attr_time_metadata: ctime=1501990117
Jul 29 11:24:04 xuexi kernel: decode_attr_time_modify: mtime=1501990117
Jul 29 11:24:04 xuexi kernel: decode_attr_mounted_on_fileid: fileid=0
Jul 29 11:24:04 xuexi kernel: decode_getfattr_attrs: xdr returned 0
Jul 29 11:24:04 xuexi kernel: decode_getfattr_generic: xdr returned 0
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: NFS: nfs_update_inode(0:40/391681 fh_crc=0xb4775a3f ct=2 info=0x26040)
Jul 29 11:24:04 xuexi kernel: NFS: permission(0:40/391681), mask=0x1, res=0
Jul 29 11:24:04 xuexi kernel: NFS: lookup(/testdir)
Jul 29 11:24:04 xuexi kernel: NFS call  lookup testdir
Jul 29 11:24:04 xuexi kernel: --> nfs4_alloc_slot used_slots=0000 highest_used=4294967295 max_slots=1024
Jul 29 11:24:04 xuexi kernel: <-- nfs4_alloc_slot used_slots=0001 highest_used=0 slotid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_type: type=040000
Jul 29 11:24:04 xuexi kernel: decode_attr_change: change attribute=7393570666420598027
Jul 29 11:24:04 xuexi kernel: decode_attr_size: file size=4096
Jul 29 11:24:04 xuexi kernel: decode_attr_fsid: fsid=(0xf199fcb4fb064bf5/0xa1b7a15af0f7cb47)
Jul 29 11:24:04 xuexi kernel: decode_attr_fileid: fileid=391682
Jul 29 11:24:04 xuexi kernel: decode_attr_fs_locations: fs_locations done, error = 0
Jul 29 11:24:04 xuexi kernel: decode_attr_mode: file mode=0754
Jul 29 11:24:04 xuexi kernel: decode_attr_nlink: nlink=3
Jul 29 11:24:04 xuexi kernel: decode_attr_owner: uid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_group: gid=0
Jul 29 11:24:04 xuexi kernel: decode_attr_rdev: rdev=(0x0:0x0)
Jul 29 11:24:04 xuexi kernel: decode_attr_space_used: space used=4096
Jul 29 11:24:04 xuexi kernel: decode_attr_time_access: atime=1501990266
Jul 29 11:24:04 xuexi kernel: decode_attr_time_metadata: ctime=1497209702
Jul 29 11:24:04 xuexi kernel: decode_attr_time_modify: mtime=1496802409
Jul 29 11:24:04 xuexi kernel: decode_attr_mounted_on_fileid: fileid=391682
Jul 29 11:24:04 xuexi kernel: decode_getfattr_attrs: xdr returned 0
Jul 29 11:24:04 xuexi kernel: decode_getfattr_generic: xdr returned 0
Jul 29 11:24:04 xuexi kernel: nfs4_free_slot: slotid 0 highest_used_slotid 4294967295
Jul 29 11:24:04 xuexi kernel: NFS reply lookup: 0
Jul 29 11:24:04 xuexi kernel: NFS: nfs_fhget(0:40/391682 fh_crc=0xec69f317 ct=1)
Jul 29 11:24:04 xuexi kernel: NFS: dentry_delete(/tmp, 202008c)
Jul 29 11:24:04 xuexi kernel: NFS: clear cookie (0xffff88005eaf4620/0x          (null))
Jul 29 11:24:04 xuexi kernel: NFS: clear cookie (0xffff88005eaf4a40/0x          (null))
Jul 29 11:24:04 xuexi kernel: NFS: releasing superblock cookie (0xffff88007746a800/0x          (null))
Jul 29 11:24:04 xuexi kernel: --> nfs_free_server()
Jul 29 11:24:04 xuexi kernel: --> nfs_put_client({2})
Jul 29 11:24:04 xuexi kernel: <-- nfs_free_server()
Jul 29 11:24:04 xuexi kernel: <-- nfs4_try_mount() = 0

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

1.9 深入 NFS 的方向
==============

本文仅简单介绍了一些 NFS 的基本使用方法，但 NFS 自身其实是很复杂的，想深入也不是一件简单的事。例如，以下列出了几个 NFS 在实现上应该要解决的问题。

(1). 多台客户端挂载同一个导出的目录后，要同时编辑其中同一个文件时，应该如何处理？这是共享文件更新问题，通用的解决方法是使用文件锁。

(2). 客户端上对文件内容做出修改后是否要立即同步到服务端上？这是共享文件的数据缓存问题，体现的方式是文件数据是否要在各客户端上保证一致性。和第一个问题结合起来，就是分布式 (集群) 文件数据一致性问题。

(3). 客户端或服务端进行了重启，或者它们出现了故障，亦或者它们之间的网络出现故障后，它们的对端如何知道它已经出现了故障？这是故障通知或重启通知问题。

(4). 出现故障后，正常重启成功了，那么如何恢复到故障前状态？这是故障恢复问题。

(5). 如果服务端故障后一直无法恢复，客户端是否能自动故障转移到另一台正常工作的 NFS 服务节点？这是高可用问题。(NFS 版本 4(后文将简写为 NFSv4) 可以从自身实现故障转移，当然使用高可用工具如 heartbeat 也一样能实现)

总结起来就几个关键词：**锁、缓存、数据和缓存一致性、通知和故障恢复**。从这些关键字中，很自然地会联想到了集群和分布式文件系统，其实 NFS 也是一种简易的分布式文件系统，但没有像集群或分布式文件系统那样实现几乎完美的锁、缓存一致性等功能。而且 NFSv4 为了体现其特点和性能，在一些通用问题上采用了与集群、分布式文件系统不同的方式，如使用了文件委托 (服务端将文件委托给客户端，由此实现类似锁的功能)、锁或委托的租约等 (就像 DHCP 的 IP 租约一样，在租约期内，锁和委托以及缓存是有效的，租约过期后这些内容都无效)。

由此知，深入 NFS 其实就是在深入分布式文件系统，由于网上对 NFS 深入介绍的文章几乎没有，所以想深入它并非一件简单的事。如果有深入学习的想法，可以阅读 [NFSv4 的 RFC3530 文档](https://www.ietf.org/rfc/rfc3530.txt)的前 9 章以及 [RCP 的 RFC 文档](https://tools.ietf.org/html/rfc5531)。以下是本人的一些 man 文档翻译。

> ```
> 翻译：man rpcbind(rpcbind中文手册)
> 翻译：man nfsd(rpc.nfsd中文手册)
> 翻译：man mountd(rpc.mountd中文手册)
> 翻译：man statd(rpc.statd中文手册)
> 翻译：man sm-notify(sm-notify命令中文手册)
> 翻译：man exportfs(exportfs命令中文手册)
> 
> 
> ```

以下是 man nfs 中关于传输方法和缓存相关内容的翻译。

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
TRANSPORT METHODS
       NFS客户端是通过RPC向NFS服务端发送请求的。RPC客户端会自动发现远程服务端点，处理每请求(per-request)的
       身份验证，调整当客户端和服务端之间出现不同字节字节序(byte endianness)时的请求参数，并且当请求在网络
       上丢失或被服务端丢弃时重传请求。rpc请求和响应数据包是通过网络传输的。
 
       在大多数情形下，mount(8)命令、NFS客户端和NFS服务端可以为挂载点自动协商合适的传输方式以及数据传输时
       的大小。但在某些情况下，需要使用mount选项显式指定这些设置。
 
       对于的传统NFS，NFS客户端只使用UDP传输请求给服务端。尽管实现方式很简单，但基于UDP的NFS有许多限制，在
       一些通用部署环境下，这些限制项会限制平滑运行的能力，还会限制性能。即使是UDP丢包率极小的情况，也将
       导致整个NFS请求丢失；因此，重传的超时时间一般都是亚秒级的，这样可以让客户端从请求丢失中快速恢复，但
       即使如此，这可能会导致无关的网络阻塞以及服务端负载加重。
 
       但是在专门设置了网络MTU值是相对于NFS的输出传输大小时(例如在网络环境下启用了巨型以太网帧)(注：相对的
       意思，例如它们成比例关系)，UDP是非常高效的。在这种环境下，建议修剪rsize和wsize，以便让每个NFS的读或
       写请求都能容纳在几个网络帧(甚至单个帧)中。这会降低由于单个MTU大小的网络帧丢失而导致整个读或写请求丢
       失的概率。
      
       (译者注：UDP是NFSv2和NFSv3所支持的，NFSv4使用TCP，所以以上两段不用考虑)
 
       现代NFS都默认采用TCP传输协议。在几乎所有可想到的网络环境下，TCP都表现良好，且提供了非常好的保证，防
       止因为网络不可靠而引起数据损坏。但使用TCP传输协议，基本上都会考虑设置相关防火墙。
 
       在正常环境下，网络丢包的频率比NFS服务丢包的频率要高的多。因此，没有必要为基于TCP的NFS设置极短的重传
       超时时间。一般基于TCP的NFS的超时时间设置在1分钟和10分钟之间。当客户端耗尽了重传次数(选项retrans的值)，
       它将假定发生了网络分裂，并尝试以新的套接字重新连接服务端。由于TCP自身使得网络传输的数据是可靠的，所
       以，可以安全地使用处在默认值和客户端服务端同时支持的最大值之间的rszie和wsize，而不再依赖于网络MTU大小。
 
DATA AND METADATA COHERENCE
       现在有些集群文件系统为客户端之间提供了非常完美的缓存一致性功能，但对于NFS客户端来说，实现完美的缓
       存一致性是非常昂贵的，特别是大型局域网络环境。因此，NFS提供了稍微薄弱一点的缓存一致性功能，以满足
       大多数文件共享需求。
 
   Close-to-open cache consistency
       一般情况下，文件共享是完全序列化的。首先客户端A打开一个文件，写入一些数据，然后关闭文件然后客户端B
       打开同一个文件，并读取到这些修改后的数据。
      
       当应用程序打开一个存储在NFSv3服务端上的文件时，NFS客户端检查文件在服务端上是否存在，并通过是否发送
       GETATTR或ACCESS请求判断文件是否允许被打开。NFS客户端发送这些请求时，不会考虑已缓存文件属性是否是新
       鲜有效的。
 
       当应用程序关闭文件，NFS客户端立即将已做的修改写入到文件中，以便下一个打开者可以看到所做的改变。这
       也给了NFS客户端一个机会，使得它可以通过文件关闭时的返回状态码向应用程序报告错误。
 
       在打开文件时以及关闭文件刷入数据时的检查行为被称为close-to-open缓存一致性，或简称为CTO。可以通过使
       用nocto选项来禁用整个挂载点的CTO。
      
       (译者注：NFS关闭文件时，客户端会将所做的修改刷入到文件中，然后发送GETATTR请求以确保该文件的属性缓存
       已被更新。之后其他的打开者打开文件时会发送GETATTR请求，根据文件的属性缓存可以判断文件是否被打开并做
       了修改。这可以避免缓存无效)
 
   Weak cache consistency
       客户端的数据缓存仍有几乎包含过期数据。NFSv3协议引入了"weak cache consistency"(WCC)，它提供了一种在单
       个文件被请求之前和之后有效地检查文件属性的方式。这有助于客户端识别出由其他客户端对此文件做出的改变。
 
       当某客户端使用了并发操作，使得同一文件在同一时间做出了多次更新(例如，后台异步写入)，它仍将难以判断
       是该客户端的更新操作修改了此文件还是其他客户端的更新操作修改了文件。
 
   Attribute caching
       使用noac挂载选项可以让多客户端之间实现属性缓存一致性。几乎每个文件系统的操作都会检查文件的属性信息。
       客户端自身会保留属性缓存一段时间以减少网络和服务端的负载。当noac生效时，客户端的文件属性缓存就会被
       禁用，因此每个会检查文件属性的操作都被强制返回到服务端上来操作文件，这表示客户端以牺牲网络资源为代
       价来快速查看文件发生的变化。
 
       不要混淆noac选项和"no data caching"。noac挂载选项阻止客户端缓存文件的元数据，但仍然可能会缓存非元数
       据的其他数据。
 
       NFS协议设计的目的不是为了支持真正完美的集群文件系统缓存一致性。如果要达到客户端之间缓存数据的绝对
       一致，那么应该使用文件锁的方式。
 
   File timestamp maintainence
       NFS服务端负责管理文件和目录的时间戳(atime,ctime,mtime)。当服务端文件被访问或被更新，文件的时间戳也会
       像本地文件系统那样改变。
 
       NFS客户端缓存的文件属性中包括了时间戳属性。当NFS客户端检索NFS服务端文件属性时，客户端文件的时间戳会
       更新。因此，在NFS服务端的时间戳更新后显示在NFS客户端之前可能会有一些延迟。
 
       为了遵守POSIX文件系统标准，Linux NFS客户端需要依赖于NFS服务端来保持文件的mtime和ctime时间戳最新状态。
       实现方式是先让客户端将改变的数据刷入到服务端，然后再输出文件的mtime。
 
       然而，Linux客户端可以很轻松地处理atime的更新。NFS客户端可以通过缓存数据来保持良好的性能，但这意味着
       客户端应用程序读取文件(会更新atime)时，不会反映到服务端，但实际上服务端的atime已经修改了。
 
       由于文件属性缓存行为，Linux NFS客户端mount时不支持一般的atime相关的挂载选项。
 
       特别是mount时指定了atime/noatime，diratime/nodiratime，relatime/norelatime以及strictatime/nostrictatime
       挂载选项，实际上它们没有任何效果。
 
   Directory entry caching
       Linux NFS客户端缓存所有NFS LOOKUP请求的结果。如果所请求目录项在服务端上存在，则查询的结果称为正查询
       结果。如果请求的目录在服务端上不存在(服务端将返回ENOENT)，则查询的结果称为负查询结果。
 
       为了探测目录项是否添加到NFS服务端或从其上移除了，Linux NFS客户端会监控目录的mtime。如果客户端监测到
       目录的mtime发生了改变，客户端将丢弃该目录相关的所有LOOKUP缓存结果。由于目录的mtime是一种缓存属性，因
       此服务端目录mtime发生改变后，客户端可能需要等一段时间才能监测到它的改变。
 
       缓存目录提高了不与其他客户端上的应用程序共享文件的应用程序的性能。但使用目录缓存可能会干扰在多个客户
       端上同时运行的应用程序，并且需要快速检测文件的创建或删除。lookupcache挂载选项允许对目录缓存行为进行一
       些调整。
     
       如果客户端禁用目录缓存，则每次LOOKUP操作都需要与服务端进行验证，那么该客户端可以立即探测到其他客户端
       创建或删除的目录。可以使用lookupcache=none来禁用目录缓存。如果禁用了目录缓存，由于需要额外的NFS请求，
       这会损失一些性能，但禁用目录缓存比使用noac损失的性能要少，且对NFS客户端缓存文件属性没有任何影响。
 
   The sync mount option
       NFS客户端对待sync挂载选项的方式和挂载其他文件系统不同。如果即不指定sync也不指定async，则默认为async。
       async表示异步写入，除了发生下面几种特殊情况，NFS客户端会延迟发送写操作到服务端：
 
              ● 内存压力迫使回收系统内存资源。
              ● 客户端应用程序显式使用了sync类的系统调用，如sync(2),msync(2)或fsync(3)。
              ● 关闭文件时。
              ● 文件被锁/解锁。
 
       换句话说，在正常环境下，应用程序写的数据不会立即保存到服务端。(当然，关闭文件时是立即同步的)
 
       如果在挂载点上指定sync选项，任何将数据写入挂载点上文件的系统调用都会先把数据刷到服务端上，然后系统调
       用才把控制权返回给用户空间。这提供了客户端之间更好的数据缓存一致性，但消耗了大量性能。
 
       在未使用sync挂载选项时，应用程序可以使用O_SYNC修饰符强制立即把单个文件的数据刷入到服务端。
 
   Using file locks with NFS
       网络锁管理器(Network Lock Manager,NLM)协议是一个独立的协议，用于管理NFSv2和NFSv3的文件锁。为了让客户端
       或服务端在重启后能恢复锁，需要使用另一个网络状态管理器(Network Status Manager,NSM)协议。在NFSv4中，NFS
       直协议自身接支持文件锁相关功能，所以NLM和NSM就不再使用了。
 
       在大多数情况下，NLM和NSM服务是自动启动的，并且不需要额外的任何配置。但需要配置NFS客户端使用的是fqdn名
       称，以保证NFS服务端可以找到客户端并通知它们服务端的重启动作。
 
       NLS仅支持advisory文件锁。要锁定NFS文件，可以使用待F_GETLK和F_SETLK的fcntl(2)命令。NFS客户端会转换通过
       flock(2)获取到的锁为advisory文件锁。
 
       当服务端不支持NLM协议，或者当NFS服务端是通过防火墙但阻塞了NLM服务端口时，则需要指定nolock挂载选项。
       当客户端挂载导出的/var目录时，必须使用nolock禁用NLM锁，因为/var目录中包含了NLM锁相关的信息。
       (注，因为NLM仅在nfsv2和nfsv3中支持，所以NFSv4不支持nolock选项)
 
       当仅有一个客户端时，使用nolock选项可以提高一定的性能。
 
   NFS version 4 caching features
       NFSv4上的数据和元数据缓存行为和之前的版本相似。但是NFSv4添加了两种特性来提升缓存行为：change attributes
       以及delegation。(注：nfsv4中取消了weak cache consistency)
 
       change attribute是一种新的被跟踪的文件/目录元数据信息。它替代了使用文件mtime和ctime作为客户端验证缓存内
       容的方式。但注意，change attributes和客户端/服务端文件的时间戳的改变无关。
 
       文件委托(file delegation)是NFSv4的客户端和NFSv4服务端之前的一种合约，它允许客户端临时处理文件，就像没有
       其他客户端正在访问该文件一样。当有其他客户端尝试访问被委托文件时，服务端一定会通知服务端(通过callback请
       求)。一旦文件被委托给某客户端，客户端可以尽可能大地缓存该文件的数据和元数据，而不需要联系服务端。
 
       文件委托有两种方式：read和write。读委托意味着当其他客户端尝试向委托文件写入数据时，服务端通知客户端。而
       写委托意味着其他客户端无论是尝试读还是写，服务端都会通知客户端。
 
       NFSv4上当文件被打开的时候，服务端会授权文件委托，并且当其他客户端想要访问该文件但和已授权的委托冲突时，
       服务端可以在任意时间点重调(recall)委托关系。不支持对目录的委托。
       (译者注：只有读委托和读委托不会冲突)
 
       为了支持委托回调(delegation callback)，在客户端初始联系服务端时，服务端会检查网络并返回路径给客户端。如果
       和客户端的联系无法建立，服务端将不会授权任何委托给客户端。

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

**转载请注明出处：[https://www.cnblogs.com/f-ck-need-u/p/7305755.html](https://www.cnblogs.com/f-ck-need-u/p/7305755.html)**

**如果觉得文章不错，不妨给个打赏，写作不易，各位的支持，能激发和鼓励我更大的写作热情。谢谢！**  

![](https://files.cnblogs.com/files/f-ck-need-u/wealipay.bmp)