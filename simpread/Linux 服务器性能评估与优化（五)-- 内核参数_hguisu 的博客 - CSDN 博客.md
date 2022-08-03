> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/hguisu/article/details/39249775)

网络内容总结（感谢原创）

之前文章[《Linux 服务器性能评估与优化 (一)》](https://blog.csdn.net/hguisu/article/details/39373311)太长，阅读不方便，因此拆分成系列博文：

[《](https://blog.csdn.net/hguisu/article/details/39373311)[Linux 服务器性能评估与优化 (一)--CPU](https://blog.csdn.net/hguisu/article/details/39373311)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/39373311)[Linux 服务器性能评估与优化 (二)-- 内存](https://blog.csdn.net/hguisu/article/details/102620787)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/102620903)[Linux 服务器性能评估与优化 (三)-- 磁盘 i/o](https://blog.csdn.net/hguisu/article/details/102620903)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/102620981)[Linux 服务器性能评估与优化 (四)-- 网络](https://blog.csdn.net/hguisu/article/details/39373311)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/39249775)[Linux 服务器性能评估与优化（五)-- 内核参数](https://blog.csdn.net/hguisu/article/details/39249775)[》](https://blog.csdn.net/hguisu/article/details/39373311)

1、Linux 内核参数优化
--------------

     内核参数是用户和系统内核之间交互的一个接口，通过这个接口，用户可以在系统运行的同时动态更新内核配置，而这些内核参数是通过 Linux Proc 文件系统存在的。因此，可以通过调整 Proc 文件系统达到优化 Linux 性能的目的。

### **一、sysctl 命令**

sysctl 命令用来配置与显示在 / proc/sys 目录中的内核参数．如果想使参数长期保存，可以通过编辑 / etc/sysctl.conf 文件来实现。

 命令格式：

 sysctl [-n] [-e] -w variable=value

 sysctl [-n] [-e] -p (default /etc/sysctl.conf)

 sysctl [-n] [-e] –a

常用参数的意义：

 -w  临时改变某个指定参数的值，如

        # sysctl -w net.ipv4.ip_forward=1

 -a  显示所有的系统参数

 -p 从指定的文件加载系统参数, 默认从 / etc/sysctl.conf 文件中加载，如：

# echo 1 > /proc/sys/net/ipv4/ip_forward

# sysctl -w net.ipv4.ip_forward=1

 以上两种方法都可能立即开启路由功能，但如果系统重启，或执行了

     # service network restart

命令，所设置的值即会丢失，如果想永久保留配置，可以修改 / etc/sysctl.conf 文件，将 net.ipv4.ip_forward=0 改为 net.ipv4.ip_forward=1

### **二、linux 内核参数调整：linux 内核参数调整有两种方式**

    方法一：修改 / proc 下内核参数文件内容，不能使用编辑器来修改内核参数文件，理由是由于内核随时可能更改这些文件中的任意一个，另外，这些内核参数文件都是虚拟文件，实际中不存在，因此不能使用编辑器进行编辑，而是使用 echo 命令，然后从命令行将输出重定向至 /proc 下所选定的文件中。如：将 timeout_timewait 参数设置为 30 秒：

# echo 30 > /proc/sys/net/ipv4/tcp_fin_timeout

参数修改后立即生效，但是重启系统后，该参数又恢复成默认值。因此，想永久更改内核参数，需要修改 / etc/sysctl.conf 文件

    方法二．修改 / etc/sysctl.conf 文件。检查 sysctl.conf 文件，如果已经包含需要修改的参数，则修改该参数的值，如果没有需要修改的参数，在 sysctl.conf 文件中添加参数。如：

   net.ipv4.tcp_fin_timeout=30

保存退出后，可以重启机器使参数生效，如果想使参数马上生效，也可以执行如下命令：

   # sysctl  -p

### **三、sysctl.conf 文件中参数设置及说明**

**1、常见配置**

```
net.ipv4.ip_local_port_range = 1024 65536  
net.core.rmem_max=16777216 
net.core.wmem_max=16777216 
net.ipv4.tcp_rmem=4096 87380 16777216  
net.ipv4.tcp_wmem=4096 65536 16777216  
net.ipv4.tcp_fin_timeout = 30 
net.core.netdev_max_backlog = 30000 
net.ipv4.tcp_no_metrics_save=1 
net.core.somaxconn = 262144 
net.ipv4.tcp_syncookies = 1 
net.ipv4.tcp_max_orphans = 262144 
net.ipv4.tcp_max_syn_backlog = 262144 
net.ipv4.tcp_synack_retries = 2 
net.ipv4.tcp_syn_retries = 2 

```

net.ipv4.ip_local_port_range：用来指定外部连接的端口范围，默认是 32 768 到 61 000，这里设置为 1024 到 65 536。

net.core.rmem_max：指定接收套接字缓冲区大小的最大值，单位是字节。

net.core.wmem_max：指定发送套接字缓冲区大小的最大值，单位是字节。

net.ipv4.tcp_rmem：此参数与 net.ipv4.tcp_wmem 都是用来优化 TCP 接收 / 发送缓冲区的，包含 3 个整数值，分别是 min、default、max。

对于 tcp_rmem，min 表示为 TCP socket 预留的用于接收缓存的最小[内存](https://so.csdn.net/so/search?q=%E5%86%85%E5%AD%98&spm=1001.2101.3001.7020)数量，default 表示为 TCP socket 预留的用于接收缓存的默认的内存值，max 表示用于 TCP socket 接收缓存的内存最大值。

对于 tcp_wmem，min 表示为 TCP socket 预留的用于发送缓存的内存最小值，default 表示为 TCP socket 预留的用于发送缓存的默认的内存值，max 表示用于 TCP socket 发送缓存的内存最大值。

net.ipv4.tcp_fin_timeout：此参数用于减少处于 FIN-WAIT-2 连接状态的时间，使系统可以处理更多的连接。此参数值为整数，单位为秒。

例如，在一个 tcp 会话过程中，在会话结束时，A 首先向 B 发送一个 fin 包，在获得 B 的 ack 确认包后，A 就进入 FIN-WAIT-2 状态等待 B 的 fin 包，然后给 B 发 ack 确认包。net.ipv4.tcp_fin_timeout 参数用来设置 A 进入 FIN-WAIT-2 状态等待对方 fin 包的超时时间。如果时间到了仍未收到对方的 fin 包就主动释放该会话。

net.core.netdev_max_backlog：该参数表示当在每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许发送到队列的数据包的最大数量。

net.ipv4.tcp_syncookies：表示是否打开 SYN Cookie。tcp_syncookies 是一个开关，该参数的功能有助于保护服务器免受 SyncFlood 攻击。默认值为 0，这里设置为 1。

net.ipv4.tcp_max_orphans：表示系统中最多有多少 TCP 套接字不被关联到任何一个用户文件句柄上。如果超过这里设置的数字，连接就会复位并输出警告信息。这个限制仅仅是为了防止简单的 DoS 攻击。此值不能太小。这里设置为 262 144。

net.ipv4.tcp_max_syn_backlog：表示 SYN 队列的长度，预设为 1024，这里设置队列长度为 262 144，以容纳更多的等待连接。

net.ipv4.tcp_synack_retries：这个参数用于设置内核放弃连接之前发送 SYN+ACK 包的数量。

net.ipv4.tcp_syn_retries：此参数表示在内核放弃建立连接之前发送 SYN 包的数量。

### **四、Linux 内核优化之 TCP/IP 相关参数**

　　所有的 TCP/IP 调优参数都位于 / proc/sys/net / 目录。 例如， 下面是最重要的一些调优参数， 后面是它们的含义：  
　　1. /proc/sys/net/core/rmem_max — 最大的 TCP 数据接收缓冲  
　　2. /proc/sys/net/core/wmem_max — 最大的 TCP 数据发送缓冲  
　　3. /proc/sys/net/ipv4/tcp_timestamps — 时间戳在 (请参考 RFC 1323)TCP 的包头增加 12 个字节  
　　4. /proc/sys/net/ipv4/tcp_sack — 有选择的应答  
　　5. /proc/sys/net/ipv4/tcp_window_scaling — 支持更大的 TCP 窗口。 如果 TCP 窗口最大超过 65535(64K)， 必须设置该数值为 1  
　　6. rmem_default — 默认的接收窗口大小  
　　7. rmem_max — 接收窗口的最大大小  
　　8. wmem_default — 默认的发送窗口大小  
　　9. wmem_max — 发送窗口的最大大小  
　　/proc 目录下的所有内容都是临时性的， 所以重启动系统后任何修改都会丢失。  
　　建议在系统启动时自动修改 TCP/IP 参数：  
　　把下面代码增加到 / etc/rc.local 文件， 然后保存文件， 系统重新引导的时候会自动修改下面的 TCP/IP 参数：  
　　echo 256960 > /proc/sys/net/core/rmem_default  
　　echo 256960 > /proc/sys/net/core/rmem_max  
　　echo 256960 > /proc/sys/net/core/wmem_default  
　　echo 256960 > /proc/sys/net/core/wmem_max  
　　echo 0 > /proc/sys/net/ipv4/tcp_timestamps  
　　echo 1 > /proc/sys/net/ipv4/tcp_sack  
　　echo 1 > /proc/sys/net/ipv4/tcp_window_scaling  
　　TCP/IP 参数都是自解释的， TCP 窗口大小设置为 256960, 禁止 TCP 的时间戳 (取消在每个数据包的头中增加 12 字节)， 支持更大的 TCP 窗口和 TCP 有选择的应答。  
　　上面数值的设定是根据互连网连接和最大带宽 / 延迟率来决定。  
　　注： 上面实例中的数值可以实际应用， 但它只包含了一部分参数。  
　　另外一个方法： 使用 /etc/sysctl.conf 在系统启动时将参数配置成您所设置的值：  
　　net.core.rmem_default = 256960  
　　net.core.rmem_max = 256960  
　　net.core.wmem_default = 256960  
　　net.core.wmem_max = 256960  
　　net.ipv4.tcp_timestamps = 0  
　　net.ipv4.tcp_sack =1  
　　net.ipv4.tcp_window_scaling = 1

=========================================================================

tcp_syn_retries：INTEGER  
默认值是 5  
对于一个新建连接，内核要发送多少个 SYN 连接请求才决定放弃。不应该大于 255，默认值是 5，对应于 180 秒左右时间。(对于大负载而物理通信良好的网络而言, 这个值偏高, 可修改为 2. 这个值仅仅是针对对外的连接, 对进来的连接, 是由 tcp_retries1 决定的)  
tcp_synack_retries：INTEGER  
默认值是 5  
对于远端的连接请求 SYN，内核会发送 SYN ＋ ACK 数据报，以确认收到上一个 SYN 连接请求包。这是所谓的三次握手 (threewayhandshake) 机制的第二个步骤。这里决定内核在放弃连接之前所送出的 SYN+ACK 数目。不应该大于 255，默认值是 5，对应于 180 秒左右时间。(可以根据上面的 tcp_syn_retries 来决定这个值)  
tcp_keepalive_time：INTEGER  
默认值是 7200(2 小时)  
当 keepalive 打开的情况下，TCP 发送 keepalive 消息的频率。(由于目前网络攻击等因素, 造成了利用这个进行的攻击很频繁, 曾经也有 cu 的朋友提到过, 说如果 2 边建立了连接, 然后不发送任何数据或者 rst/fin 消息, 那么持续的时间是不是就是 2 小时, 空连接攻击? tcp_keepalive_time 就是预防此情形的. 我个人在做 nat 服务的时候的修改值为 1800 秒)  
**tcp_keepalive_probes：INTEGER**  
默认值是 9  
TCP 发送 keepalive 探测以确定该连接已经断开的次数。(注意: 保持连接仅在 SO_KEEPALIVE 套接字选项被打开是才发送. 次数默认不需要修改, 当然根据情形也可以适当地缩短此值. 设置为 5 比较合适)  
tcp_keepalive_intvl：INTEGER  
默认值为 75  
探测消息发送的频率，乘以 tcp_keepalive_probes 就得到对于从开始探测以来没有响应的连接杀除的时间。默认值为 75 秒，也就是没有活动的连接将在大约 11 分钟以后将被丢弃。(对于普通应用来说, 这个值有一些偏大, 可以根据需要改小. 特别是 web 类服务器需要改小该值, 15 是个比较合适的值)  
tcp_retries1：INTEGER  
默认值是 3  
放弃回应一个 TCP 连接请求前﹐需要进行多少次重试。RFC 规定最低的数值是 3﹐这也是默认值﹐根据 RTO 的值大约在 3 秒 - 8 分钟之间。(注意: 这个值同时还决定进入的 syn 连接)  
tcp_retries2：INTEGER  
默认值为 15  
在丢弃激活 (已建立通讯状况) 的 TCP 连接之前﹐需要进行多少次重试。默认值为 15，根据 RTO 的值来决定，相当于 13-30 分钟(RFC1122 规定，必须大于 100 秒).(这个值根据目前的网络设置, 可以适当地改小, 我的网络内修改为了 5)  
tcp_orphan_retries：INTEGER  
默认值是 7  
在近端丢弃 TCP 连接之前﹐要进行多少次重试。默认值是 7 个﹐相当于 50 秒 - 16 分钟﹐视 RTO 而定。如果您的系统是负载很大的 web 服务器﹐那么也许需要降低该值﹐这类 sockets 可能会耗费大量的资源。另外参的考 tcp_max_orphans。(事实上做 NAT 的时候, 降低该值也是好处显著的, 我本人的网络环境中降低该值为 3)  
tcp_fin_timeout：INTEGER  
默认值是 60  
对于本端断开的 socket 连接，TCP 保持在 FIN-WAIT-2 状态的时间。对方可能会断开连接或一直不结束连接或不可预料的进程死亡。默认值为 60 秒。过去在 2.2 版本的内核中是 180 秒。您可以设置该值﹐但需要注意﹐如果您的机器为负载很重的 web 服务器﹐您可能要冒内存被大量无效数据报填满的风险﹐FIN-WAIT-2 sockets 的危险性低于 FIN-WAIT-1 ﹐因为它们最多只吃 1.5K 的内存﹐但是它们存在时间更长。另外参考 tcp_max_orphans。(事实上做 NAT 的时候, 降低该值也是好处显著的, 我本人的网络环境中降低该值为 30)  
tcp_max_tw_buckets：INTEGER  
默认值是 180000  
系统在同时所处理的最大 timewaitsockets 数目。如果超过此数的话﹐time-wait socket 会被立即砍除并且显示警告信息。之所以要设定这个限制﹐纯粹为了抵御那些简单的 DoS 攻击﹐千万不要人为的降低这个限制﹐不过﹐如果网络条件需要比默认值更多﹐则可以提高它 (或许还要增加内存)。(事实上做 NAT 的时候最好可以适当地增加该值)

  
tcp_tw_recycle：BOOLEAN  
默认值是 0  
打开快速 TIME-WAITsockets 回收。除非得到技术专家的建议或要求﹐请不要随意修改这个值。(做 NAT 的时候，建议打开它)：

> 当 net.ipv4.tcp_tw_recycle= 1：
> 
> 就有可能导致服务器收到 SYN 但是不会向客户端发送 SYN+ACK 包，结果导致客户端重传的发生。
> 
> 原因是打开 tcp_tw_recycle 参数后，服务器会识别这些 SYN 包的时间戳（net.ipv4.tcp_timestamps = 1），如果 SYN 数据包时间戳不是顺序的，导致服务器认为 SYN 包不可信而丢弃, 结果服务器收到 SYN 但是不会向客户端发送 SYN+ACK 包。

tcp_tw_reuse：BOOLEAN  
默认值是 0  
该文件表示是否允许重新应用处于 TIME-WAIT 状态的 socket 用于新的 TCP 连接 (这个对快速重启动某些服务, 而启动后提示端口已经被使用的情形非常有帮助)  
tcp_max_orphans：INTEGER  
缺省值是 8192  
系统所能处理不属于任何进程的 TCP sockets

2、Linux 文件系统优化
==============

    ulimit -a 用来显示当前的各种用户进程限制。  
　　Linux 对于每个用户，系统限制其最大进程数。为提高性能，可以根据设备资源情况，设置各 linux 用户的最大进程数，下面我把某 linux 用户的最大进程数设为 10000 个：  
　　ulimit -u 10000  
　　其他建议设置成无限制 (unlimited) 的一些重要设置是：  
　　数据段长度：ulimit -d unlimited  
　　最大内存大小：ulimit -m unlimited  
　　堆栈大小：ulimit -s unlimited  
　　CPU 时间：ulimit -t unlimited  
　　虚拟内存：ulimit -v unlimited  
　　暂时地，适用于通过 ulimit 命令登录 shell 会话期间。

    永久修改 ulimit：

 ulimit -SHn 65535  
    ubuntu limit 设置需修改三个文件：  
   [/etc/sysctl.conf]  
    fs.file-max = 65535

[/etc/pam.d/common-session]  
    session required pam_limits.so  
    [/etc/security/limits.conf]

```
   *  soft nproc 65535
       *  hard nproc 65535
       *  soft nofile 65535 
       *  hard nofile 65535
```

　　说明：* 代表针对所有用户  
　　noproc 是代表最大进程数  
　　nofile 是代表最大文件打开数  
　　2)、让 SSH 接受 Login 程式的登入，方便在 ssh 客户端查看 ulimit -a 资源限制：  
　　a、vi /etc/ssh/sshd_config  
　　把 UserLogin 的值改为 yes，并把 # 注释去掉  
　　b、重启 sshd 服务：  
　　/etc/init.d/sshd restart  
　　3)、修改所有 linux 用户的环境变量文件：  
　　vi /etc/profile  
　　ulimit -u 10000  
　　ulimit -n 4096  
　　ulimit -d unlimited  
　　ulimit -m unlimied  
　　ulimit -s unlimited  
　　ulimit -t unlimited  
　　ulimit -v unlimited

    我们通过内核参数了解，TCP 连接建立的时候会分配接收缓冲区和发送缓冲区，各 4KB，一共是 8K  
     net.ipv4.tcp_wmem = 4096 16384 4194304

 net.ipv4.tcp_rmem = 4096 87380 4194304

 第一个值是 socket 的发送缓存区分配的最少字节数;  
     第二个值是默认值 (该值会被 net.core.wmem_default 覆盖), 缓存区在系统负载不重的情况下可以增长到这个值；  
     第三个值是发送缓存区空间的最大字节数 (该值会被 net.core.wmem_max 覆盖)。

 就是说，每个 tcp 连接的 socket，根据实际情况测试，如果单单建立稳定连接，缓冲区基本不占内存。

 [https://zhuanlan.zhihu.com/p/25241630?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io](https://zhuanlan.zhihu.com/p/25241630?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

3、Linux 内存调优

　　内存子系统的调优不是很容易，需要不停地监测来保证内存的改变不会对服务器的其他子系统造成负面影响。如果要改变虚拟内存参数 (在 / proc/sys/vm)，建议您每次只改变一个参数然后监测效果。对与虚拟内存的调整包括以下几个项目：  
　　配置 Linux 内核如何更新 dirty buffers 到磁盘。磁盘缓冲区用于暂存磁盘的数据。相对于内存来讲，磁盘缓冲区的速度很慢。因此，如果服务器使用这类内存，性能会成问题。当缓冲区内的数据完全 dirty，使用：sysctl -w vm.bdflush="30 500 0 0 500 3000 60 20 0"  
　　vm.bdflush 有 9 个参数，但是建议您只改变其中的 3 个：  
　　1 nfract, 为排队写入磁盘前，bdflush daemon 允许的缓冲区最大百分比  
　　2 ndirty, 为 bdflush 即刻写的最大缓冲区的值。如果这个值很大，bdflush 需要更多的时间完成磁盘的数据更新。  
　　7 nfract_sync, 发生同步前，缓冲区变 dirty 的最大百分比  
　　配置 kswapd daemon，指定 Linux 的内存页数量  
　　sysctl -w vm.kswapd="1024 32 64"  
　　三个参数的描述如下：  
　　– tries_base 相当于内核每次所的 “页” 的数量的四倍。对于有很多交换信息的系统，增加这个值可以改进性能。  
　　– tries_min 是每次 kswapd swaps 出去的 pages 的最小数量。  
　　– swap_cluster 是 kswapd 即刻写如的 pages 数量。数值小，会提高磁盘 I/O 的性能; 数值大可能也会对请求队列产生负面影响。  
　　如果要对这些参数进行改动，请使用工具 vmstat 检查对性能的影响。其它可以改进性能的虚拟内存参数为：  
　　_ buffermem  
　　_ freepages  
　　_ overcommit_memory  
　　_ page-cluster  
　　_ pagecache  
　　_ pagetable_cache

4、Linux 网络调优
============

操作系统安装完毕，就要对网络子系统进行调优。对其它子系统的影响：影响 CPU 利用率，尤其在有大量 TCP 连接、块尺寸又非常小时，内存的使用会  
明显增加。  
**1）如何预防性能下降**  
　　如下的 sysctl 命令用于改变安全设置，但是它也可以防止网络性能的下降。这些命令被设置为缺省值。  
◆关闭如下参数可以防止黑客对服务器 IP 地址的攻击  
       sysctl -w net.ipv4.conf.eth0.accept_source_route=0  
　　sysctl -w net.ipv4.conf.lo.accept_source_route=0  
　　sysctl -w net.ipv4.conf.default.accept_source_route=0  
　　sysctl -w net.ipv4.conf.all.accept_source_route=0  
**◆开启 TCP SYN cookies****，**保护服务器避免受 syn-flood 攻击，包括服务取决 denial-of-service (DoS) 或者分布式服务拒绝 distributed denial-of-  
　service (DDoS) (仅适用 Red Hat Enterprise Linux AS)  
　　sysctl -w net.ipv4.tcp_syncookies=1  
◆以下命令使服务器忽略来自被列入网关的服务器的重定向。因重定向可以被用来进行攻击，所以我们只接受有可靠来源的重定向。  
　　sysctl -w net.ipv4.conf.eth0.secure_redirects=1  
　　sysctl -w net.ipv4.conf.lo.secure_redirects=1  
　　sysctl -w net.ipv4.conf.default.secure_redirects=1  
　　sysctl -w net.ipv4.conf.all.secure_redirects=1  
　　另外，你可以配置接受或拒绝任何 ICMP 重定向。ICMP 重定向是器传输信息的机制。比如，当网关接收到来自所接网络主机的 Internet 数据报时，网关可以发送重定向信息到一台主机。网关检查路由表获得下一个网关的地址，第二个网关将数据报路由到目标网络。关闭这些重定向得命令如下：  
　　sysctl -w net.ipv4.conf.eth0.accept_redirects=0  
　　sysctl -w net.ipv4.conf.lo.accept_redirects=0  
　　sysctl -w net.ipv4.conf.default.accept_redirects=0  
　　sysctl -w net.ipv4.conf.all.accept_redirects=0  
◆如果这个服务器不是一台路由器，那么它不会发送重定向，所以可以关闭该功能：  
　　sysctl -w net.ipv4.conf.eth0.send_redirects=0  
　　sysctl -w net.ipv4.conf.lo.send_redirects=0  
　　sysctl -w net.ipv4.conf.default.send_redirects=0  
　　sysctl -w net.ipv4.conf.all.send_redirects=0  
◆配置服务器拒绝接受广播风暴或者 smurf 攻击 attacks:  
　　sysctl -w net.ipv4.icmp_echo_ignore_broadcasts=1  
◆忽略所有 icmp 包或者 pings:  
      sysctl -w net.ipv4.icmp_echo_ignore_all=1  
◆有些路由器针对广播祯发送无效的回应，每个都产生警告并在内核产生日志。这些回应可以被忽略：  
　　sysctl -w net.ipv4.icmp_ignore_bogus_error_responses=1  
 

**2）针对 TCP 和 UDP 的调优**  
　　下边的命令用来对连接数量非常大的服务器进行调优。  
◆对于同时支持很多连接的服务器，新的连接可以重新使用 TIME-WAIT 套接字。 这对于 Web 服务器非常有效：  
　　sysctl -w net.ipv4.tcp_tw_reuse=1  
　　如果你使用该命令，还要启动 TIME-WAIT 套接字状态的快速循环功能：  
　　sysctl -w net.ipv4.tcp_tw_recycle=1  
　　每个 TCP 传输都包含远程客户端的信息缓存，所以有利于提高性能。缓存中存放 round-trip 时间、最大 segment 大小、拥塞窗口的信息。  
◆参数 tcp_fin_timeout 是套接字关闭时，保持 FIN-WAIT-2 状态的时间。

     一个 TCP 连接以 three-segment SYN 序列开始， 以 three-segment FIN 序列结束。均不保留数据。通过改变 tcp_fin_timeout 的值， 从 FIN 序列到内存可以空闲出来处理新连接的时间缩短了，使性能得到改进。改变这个值的前要经过认真的监测, 避免因为死套接字造成内存溢出。  
　　sysctl -w net.ipv4.tcp_fin_timeout=30  
◆服务器的一个问题是，同一时刻的大量 TCP 连接里有很多的连接被打开但是没有使用。 TCP 的 keepalive 功能检测到这些连接，缺省情况下，在 2 小时之后丢掉. 2 个小时的可能导致内存过度使用，降低性能。因此改成 1800 秒 (30 分钟) 是个更好的选择：  
　　sysctl -w net.ipv4.tcp_keepalive_time=1800  
◆对于所有的队列，设置最大系统发送缓存 (wmem) 和接收缓存(rmem) 到 8MB  
　　sysctl -w net.ipv4.core.wmem_max=8388608  
　　sysctl -w net.ipv4.core.rmem_max=8388608  
　　这些设置指定了创建 TCP 套接字时为其分配的内存容量。 另外，使用如下命令发送和接收缓存。该命令设定了三个值：最小值、初始值和最大值：  
　　sysctl -w net.ipv4.tcp_rmem="4096 87380 8388608"  
　　sysclt -w net.ipv4.tcp.wmem="4096 87380 8388608"  
　　第三个值必须小于或等于 wmem_max 和 rmem_max。  
◆(SUSE LINUX Enterprise Server 适用) 通过保留路径验证来源数据包。缺省情况下，路由器转发所有的数据包，即便是明显的异常网络流量。通过启动和是的过滤功能，丢掉这些数据包：  
　　sysctl -w net.ipv4.conf.eth0.rp_filter=1  
　　sysctl -w net.ipv4.conf.lo.rp_filter=1  
　　sysctl -w net.ipv4.conf.default.rp_filter=1  
　　sysctl -w net.ipv4.conf.all.rp_filter=1  
◆当服务器负载繁重或者是有很多客户端都是超长延时的连接故障，可能会导致 half-open 连接数量的增加。这对于 Web 服务器很来讲很平常，尤其有很多拨号客户时。这些 half-open 连接保存在 backlog connections 队列中。将这个值最少设置为 4096 (缺省为 1024)。 即便是服务器不接收这类连接，设置这个值还能防止受到 denial-of-service (syn-flood) 的攻击。  
　　sysctl -w net.ipv4.tcp_max_syn_backlog=4096  
◆设置 ipfrag 参数，尤其是 NFS 和 Samba 服务器。这里，我们可以设置用于重新组合 IP 碎片的最大、最小内存。当 ipfrag_high_thresh 值被指派，碎片会被丢弃直到达到 ipfrag_low_thres 值。当 TCP 数据包传输发生错误时，开始碎片整理。有效的数据包保留在内存，同时损坏的数据包被转发。例如，设置可用内存范围从 256 MB 到 384 MB  
　　sysctl -w net.ipv4.ipfrag_low_thresh=262144  
　　sysctl -w net.ipv4.ipfrag_high_thresh=393216

5、Linux 网络安全设置
--------------

**1）TCP SYN Flood 攻击**  
　　TCP SYN Flood 是一种常见，而且有效的远端 (远程) 拒绝服务 (Denial of Service) 攻击方式，它透过一定的操作破坏 TCP 三次握手建立正常连接，占用并耗费系统资源，使得提供 TCP 服务的主机系统无法正常工作。由於 TCP SYN Flood 是透过网路底层对服务器 Server 进行攻击的，它可以在任意改变自己的网路 IP 地址的同时，不被网路上的其他设备所识别，这样就给防范网路犯罪部门追查犯罪来源造成很大的困难。系统检查  
　　一般情况下，可以一些简单步骤进行检查，来判断系统是否正在遭受 TCP SYN Flood 攻击。  
　　1、服务端无法提供正常的 TCP 服务。连接请求被拒绝或超时。  
　　2、透过 netstat -an 命令检查系统，发现有大量的 SYN_RECV 连接状态。  
　　3. iptables 的设置，引用自 CU 防止同步包洪水 (Sync Flood)  
　　# iptables -A FORWARD -p tcp --syn -m limit --limit 1/s -j ACCEPT  
　　也有人写作  
　　#iptables -A INPUT -p tcp --syn -m limit --limit 1/s -j ACCEPT  
　　--limit 1/s 限制 syn 并发数每秒 1 次，可以根据自己的需要修改  
　　防止各种端口扫描  
　　# iptables -A FORWARD -p tcp --tcp-flags SYN,ACK,FIN,RST RST -m limit --limit 1/s -j ACCEPT  
　　Ping 洪水攻击 (Ping of Death)  
 **# iptables -A FORWARD -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT**

2）Linux 上的 NAT 与 iptables

     谈起 Linux 上的 NAT，大多数人会跟你提到 iptables。原因是因为 iptables 是目前在 linux 上实现 NAT 的一个非常好的接口。它通过和内核级直接操作网络包，效率和稳定性都非常高。这里简单列举一些 NAT 相关的 iptables 实例命令，可能对于大多数实现有多帮助。

# 案例 1：实现网关的 MASQUERADE

# 具体功能：内网网卡是 eth1，外网 eth0，使得内网指定本服务做网关可以访问外网

EXTERNAL="eth0"

INTERNAL="eth1"

# 这一步开启 ip 转发支持，这是 NAT 实现的前提

echo 1 > /proc/sys/net/ipv4/ip_forward

iptables -t nat -A POSTROUTING -o $EXTERNAL -j MASQUERADE

# 案例 2：实现网关的简单端口映射

具体功能：实现外网通过访问网关的外部 ip:80，可以直接达到访问私有网络内的一台主机 192.168.1.10:80 效果

LOCAL_EX_IP=11.22.33.44 #设定网关的外网卡 ip，对于多 ip 情况，参考《如何让你的 Linux 网关更强大》系列文章

LOCAL_IN_IP=192.168.1.1  # 设定网关的内网卡 ip

INTERNAL="eth1" #设定内网卡

这一步开启 ip 转发支持，这是 NAT 实现的前提

echo 1 > /proc/sys/net/ipv4/ip_forward

加载需要的 ip 模块，下面两个是 ftp 相关的模块，如果有其他特殊需求，也需要加进来

modprobe ip_conntrack_ftp

modprobe ip_nat_ftp

这一步实现目标地址指向网关外部 ip:80 的访问都吧目标地址改成 192.168.1.10:80

iptables -t nat -A PREROUTING -d $LOCAL_EX_IP -p tcp --dport 80 -j DNAT --to 192.168.1.10

这一步实现把目标地址指向 192.168.1.10:80 的数据包的源地址改成网关自己的本地 ip，这里是 192.168.1.1

iptables -t nat -A POSTROUTING -d 192.168.1.10 -p tcp --dport 80 -j SNAT --to $LOCAL_IN_IP

在 FORWARD 链上添加到 192.168.1.10:80 的允许，否则不能实现转发

iptables -A FORWARD -o $INTERNAL -d 192.168.1.10 -p tcp --dport 80 -j ACCEPT

通过上面重要的三句话之后，实现的效果是，通过网关的外网 ip:80 访问，全部转发到内网的 192.168.1.10:80 端口，实现典型的端口映射

特别注意，所有被转发过的数据都是源地址是网关内网 ip 的数据包，所以 192.168.1.10 上看到的所有访问都好像是网关发过来的一样，而看不到外部 ip

一个重要的思想：数据包根据 “从哪里来，回哪里去” 的策略来走，所以不必担心回头数据的问题

现在还有一个问题，网关自己访问自己的外网 ip:80，是不会被 NAT 到 192.168.1.10 的，这不是一个严重的问题，但让人很不爽，解决的方法如下：

iptables -t nat -A OUTPUT -d $LOCAL_EX_IP -p tcp --dport 80 -j DNAT --to 192.168.1.10

获取系统中的 NAT 信息和诊断错误

了解 / proc 目录的意义

      在 Linux 系统中，/proc 是一个特殊的目录，proc 文件系统是一个伪文件系统，它只存在内存当中，而不占用外存空间。它包含当前系统的一些参数（variables）和状态（status）情况。它以文件系统的方式为访问系统内核数据的操作提供接口

     通过 / proc 可以了解到系统当前的一些重要信息，包括磁盘使用情况，内存使用状况，硬件信息，网络使用情况等等，很多系统监控工具（如 HotSaNIC）都通过 / proc 目录获取系统数据。

另一方面通过直接操作 / proc 中的参数可以实现系统内核参数的调节，比如是否允许 ip 转发，syn-cookie 是否打开，tcp 超时时间等。

获得参数的方式：

第一种：cat /proc/xxx/xxx，如 cat /proc/sys/net/ipv4/conf/all/rp_filter

第二种：sysctl xxx.xxx.xxx，如 sysctl net.ipv4.conf.all.rp_filter

改变参数的方式：

第一种：echo value > /proc/xxx/xxx，如 echo 1 > /proc/sys/net/ipv4/conf/all/rp_filter

第二种：sysctl [-w] variable=value，如 sysctl [-w] net.ipv4.conf.all.rp_filter=1

以上设定系统参数的方式只对当前系统有效，重起系统就没了，想要保存下来，需要写入 / etc/sysctl.conf 文件中

通过执行 man 5 proc 可以获得一些关于 proc 目录的介绍

查看系统中的 NAT 情况

和 NAT 相关的系统变量

/proc/slabinfo：内核缓存使用情况统计信息（Kernel slab allocator statistics）

/proc/sys/net/ipv4/ip_conntrack_max：系统支持的最大 ipv4 连接数，默认 65536（事实上这也是理论最大值）

/proc/sys/net/ipv4/netfilter/ip_conntrack_tcp_timeout_established 已建立的 tcp 连接的超时时间，默认 432000，也就是 5 天

和 NAT 相关的状态值

/proc/net/ip_conntrack：当前的前被跟踪的连接状况，nat 翻译表就在这里体现（对于一个网关为主要功能的 Linux 主机，里面大部分信息是 NAT 翻译表）

/proc/sys/net/ipv4/ip_local_port_range：本地开放端口范围，这个范围同样会间接限制 NAT 表规模

# 1. 查看当前系统支持的最大连接数

cat /proc/sys/net/ipv4/ip_conntrack_max 

# 值：默认 65536，同时这个值和你的内存大小有关，如果内存 128M，这个值最大 8192，1G 以上内存这个值都是默认 65536

# 影响：这个值决定了你作为 NAT 网关的工作能力上限，所有局域网内通过这台网关对外的连接都将占用一个连接，如果这个值太低，将会影响吞吐量

# 2. 查看 tcp 连接超时时间

cat /proc/sys/net/ipv4/netfilter/ip_conntrack_tcp_timeout_established 

# 值：默认 432000（秒），也就是 5 天

# 影响：这个值过大将导致一些可能已经不用的连接常驻于内存中，占用大量链接资源，从而可能导致 NAT ip_conntrack: table full 的问题

# 建议：对于 NAT 负载相对本机的 NAT 表大小很紧张的时候，可能需要考虑缩小这个值，以尽早清除连接，保证有可用的连接资源；如果不紧张，不必修改

# 3. 查看 NAT 表使用情况（判断 NAT 表资源是否紧张）

# 执行下面的命令可以查看你的网关中 NAT 表情况

cat /proc/net/ip_conntrack

# 4. 查看本地开放端口的范围

cat /proc/sys/net/ipv4/ip_local_port_range

# 返回两个值，最小值和最大值

# 下面的命令帮你明确一下 NAT 表的规模

wc -l /proc/net/ip_conntrack

#或者

grep ip_conntrack /proc/slabinfo | grep -v expect | awk '{print $1','$2;}'

# 下面的命令帮你明确可用的 NAT 表项，如果这个值比较大，那就说明 NAT 表资源不紧张

grep ip_conntrack /proc/slabinfo | grep -v expect | awk '{print $1','$3;}'

# 下面的命令帮你统计 NAT 表中占用端口最多的几个 ip，很有可能这些家伙再做一些 bt 的事情，嗯 bt 的事情:-)

cat /proc/net/ip_conntrack | cut -d '' -f 10 | cut -d'=' -f 2 | sort | uniq -c | sort -nr | head -n 10

# 上面这个命令有点瑕疵 cut -d' ' -f10 会因为命令输出有些行缺项而造成统计偏差，下面给出一个正确的写法：

cat /proc/net/ip_conntrack | perl -pe s/^$.*?$src/src/g | cut -d '' -f1 | cut -d'=' -f2 | sort | uniq -c | sort -nr | head -n 10

感谢您的支持，我会继续努力的! 扫码打赏，你说多少就多少

![](https://img-blog.csdn.net/20170331111807982)         ![](https://img-blog.csdn.net/20170331111745092)