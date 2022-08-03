> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [guisu.blog.csdn.net](https://guisu.blog.csdn.net/article/details/102620981)

> 之前文章《Linux 服务器性能评估与优化 (一)》太长，阅读不方便，因此拆分成系列博文：《Linux 服务器性能评估与优化 (一)--CPU》《Linux 服务器性能评估与优化 (二)-- 内存》《Linux 服务器......

之前文章[《Linux 服务器性能评估与优化 (一)》](https://blog.csdn.net/hguisu/article/details/39373311)太长，阅读不方便，因此拆分成系列博文：

[《](https://blog.csdn.net/hguisu/article/details/39373311)[Linux 服务器性能评估与优化 (一)--CPU](https://blog.csdn.net/hguisu/article/details/102620981)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/39373311)[Linux 服务器性能评估与优化 (二)-- 内存](https://blog.csdn.net/hguisu/article/details/102620787)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/102620903)[Linux 服务器性能评估与优化 (三)-- 磁盘 i/o](https://blog.csdn.net/hguisu/article/details/102620903)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/102620981)[Linux 服务器性能评估与优化 (四)-- 网络](https://blog.csdn.net/hguisu/article/details/39373311)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/39249775)[Linux 服务器性能评估与优化 (五)-- 内核参数](https://blog.csdn.net/hguisu/article/details/39249775)[》](https://blog.csdn.net/hguisu/article/details/39373311)

1、网络性能评估
--------

 网络是所有子系统中最难监测的一个， 因为网络比较抽象， 在监测时有很多在系统可控制之外的因素如延迟，冲突，拥塞和丢包等对监测产生影响。

测量网络性能的五项指标是：

*   可用性（availability）
*   响应时间（response time）
*   网络利用率（network utilization）
*   网络吞吐量（network throughput）
*   网络带宽容量（network bandwidth capacity）

**1． 可用性**

测试网络性能的第一步是确定网络是否正常工作，最简单的方法是使用 ping 命令。通过向远端的机器发送 icmp echo request，并等待接收 icmp echo reply 来判断远端的机器是否连通，网络是否正常工作。

Ping 命令有非常丰富的命令选项，比如 -c 可以指定发送 echo request 的个数，-s 可以指定每次发送的 ping 包大小。

网络设备内部一般有多个缓冲池，不同的缓冲池使用不同的缓冲区大小，分别用来处理不同大小的分组（packet）。例如交换机中通常具有三种类型的包缓冲：一类针对小的分组，一类针对中等大小的分组，还有一类针对大的分组。为了测试这样的网络设备，测试工具必须要具有发送不同大小分组的能力。Ping 命令的 -s 就可以使用在这种场合。

**2． 响应时间**

Ping 命令的 echo request/reply 一次往返所花费时间就是响应时间。有很多因素会影响到响应时间，如网段的负荷，网络主机的负荷，广播风暴，工作不正常的网络设备等等。

在网络工作正常时，记录下正常的响应时间。当用户抱怨网络的反应时间慢时，就可以将现在的响应时间与正常的响应时间对比，如果两者差值的波动很大，就能说明网络设备存在故障。

**3． 网络利用率**

网络利用率是指网络被使用的时间占总时间（即被使用的时间 + 空闲的时间）的比例。比如，Ethernet 虽然是共享的，但同时却只能有一个报文在传输。因此在任一时刻，Ethernet 或者是 100% 的利用率，或者是 0% 的利用率。

计算一个网段的网络利用率相对比较容易，但是确定一个网络的利用率就比较复杂。因此，网络测试工具一般使用网络吞吐量和网络带宽容量来确定网络中两个节点之间的性能。

**4． 网络吞吐量**

网络吞吐量是指在某个时刻，在网络中的两个节点之间，提供给网络应用的剩余带宽。

网络吞吐量可以帮组寻找网络路径中的瓶颈。比如，即使 client 和 server 都被分别连接到各自的 100M Ethernet 上，但是如果这两个 100M 的 Ethernet 被 10M 的 Ethernet 连接起来，那么 10M 的 Ethernet 就是网络的瓶颈。

网络吞吐量非常依赖于当前的网络负载情况。因此，为了得到正确的网络吞吐量，最好在不同时间（一天中的不同时刻，或者一周中不同的天）分别进行测试，只有这样才能得到对网络吞吐量的全面认识。

有些网络应用程序在开发过程的测试中能够正常运行，但是到实际的网络环境中却无法正常工作（由于没有足够的网络吞吐量）。这是因为测试只是在空闲的网络环境中，没有考虑到实际的网络环境中还存在着其它的各种网络流量。所以，网络吞吐量定义为剩余带宽是有实际意义的。

**5． 网络带宽容量**

与网络吞吐量不同，网络带宽容量指的是在网络的两个节点之间的最大可用带宽。这是由组成网络的设备的能力所决定的。

测试网络带宽容量有两个困难之处：在网络存在其它网络流量的时候，如何得知网络的最大可用带宽；在测试过程中，如何对现有的网络流量不造成影响。网络测试工具一般采用 packet pairs 和 packet trains 技术来克服这样的困难。

2、常用的命令工具
---------

**（1）通过 ping 命令检测网络的连通性**

**（2）通过 netstat –i 组合检测网络接口状况**

**（3）通过 netstat –r 组合检测系统的路由表信息**

**（4）通过 sar –n 组合显示系统的网络运行状态**

**netstat -antl  查看所有 tcp status：**

注意：可以通过 netstat 查看是否 timewait 过多的情况，导致端口不够用，在短连接服务中且大并发情况下，要不系统的 net.ipv4.tcp_tw_reuse、net.ipv4.tcp_tw_recycle 两个选项打开，允许端口重用。

**linux 查看 tcp 的状态命令：**  
1)、netstat -nat  查看 TCP 各个状态的数量  
2)、lsof  -i:port  可以检测到打开套接字的状况  
3)、  sar -n SOCK 查看 tcp 创建的连接数  
4)、tcpdump -iany tcp port 9000 对 tcp 端口为 9000 的进行抓包  
5)、tcpdump   dst  port 9000 -w dump9000.pcap  对 tcp 目标端口为 9000 的进行抓包保存 pcap 文件 wireshark 分析。  
6)、tcpdump tcp port 9000  -n -X -s 0 -w tcp.cap 对 tcp/http 目标端口为 9000 的进行抓包保存 pcap 文件 wireshark 分析。

3、**网络吞吐量监测**
-------------

监测网络吞吐量最好的办法是在两个系统之间发送流量并统计其延迟和速度。

**3）测试工具：使用 iptraf 监测本地吞吐量**

简单安装：yum install -y iptraf  

在 centos7.2 使用的命令是 iptraf-ng：

使用 iptraf-ng -d eth0

iptraf 工具可提供以太网卡的吞吐量情况：  
# iptraf -d eth0

![](https://img-blog.csdn.net/20140918232714593?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGd1aXN1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

上面的数据显示被测试系统正以 61mbps（7.65M）频率发送数据，相比于 100mbps 网络这有点低。

使用 iptraf 命令找出流量使用情况和接口、端口信息

输入命令： iptraf 

然后 iptraf 会给出如下所示的输出。结果给出了两样东西，源地址和网络端口号。在第一次出现的 welcome 屏幕上按下 Enter，就可以看见具体的选项了。一旦你选择了在所有接口之上的 “IP traffic monitor” 选项：

![](https://img-blog.csdn.net/20170622214925992?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGd1aXN1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

你会看到如下的输出结果。

![](https://img-blog.csdn.net/20170622215157039?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGd1aXN1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

4、**使用 netperf 监测**
-------------------

与 iptraf 的动态监测不一样的是 netperf 使用可控方式测试网络， 这一点对测试一个客户端到一个高负载服务器之间的吞吐量很有帮助，netperf 工具是以 C/S 模式运行。

首先需要在服务器上运行 netperf 服务端：

在服务端启动 server# netserver

Unable to start netserver with  'IN(6)ADDR_ANY' port '12865' and family AF_UNSPEC

### **4.1、在 client 端测试吞吐量**

# /usr/local/bin/netperf -H 10.165.112.121 -l 10 [-t TCP_STREAM]

![](https://img-blog.csdnimg.cn/20191017183730354.png)  
吞吐量的测试结果为 900.42Mbits / 秒  
从 netperf 的结果输出中，我们可以知道以下的一些信息：  
1） 远端系统（即 server）使用大小为 87380 字节的 socket 接收缓冲  
2） 本地系统（即 client）使用大小为 16384 字节的 socket 发送缓冲  
3） 向远端系统发送的测试分组大小为 16384 字节  
4） 测试经历的时间为 10.03 秒  
5） 吞吐量的测试结果为 900.42Mbits / 秒  
在默认情况下，netperf 向发送的测试分组大小设置为本地系统所使用的 socket 发送缓冲大小。  
TCP_STREAM 方式下与测试相关的局部参数如下表所示：  
参数 说明  
-W  send,recv  设置本地, 远端系统接收测试分组的大小: Set the number of send,recv buffers  
通过修改以上的参数，并观察结果的变化，我们可以确定是什么因素影响了连接的吞吐量。例如，如果怀疑路由器由于缺乏足够的缓冲区空间，使得转发大的分组时存在问题，就可以增加测试分组（-W）的大小，以观察吞吐量的变化：  
/usr/local/bin/netperf -H 10.165.112.121 -l 10 -W 2048

![](https://img-blog.csdnimg.cn/2019101719025199.png)  
在这里，测试分组的大小减少到 2048 字节，而吞吐量却没有很大的变化（与前面例子中测试分组大小为 16K 字节相比）。相反，如果吞吐量有了较大的提升，则说明在网络中间的路由器确实存在缓冲区的问题。

### **4.2、测试请求 / 应答（request/response）网络流量的性能**

另一类常见的网络流量类型是应用在 client/server 结构中的 request/response 模式。在每次交易（transaction）中，client 向 server 发出小的查询分组，server 接收到请求，经处理后返回大的结果数据。

**1． TCP_RR**  
TCP_RR 方式的测试对象是多次 TCP request 和 response 的交易过程，但是它们发生在同一个 TCP 连接中，这种模式常常出现在数据库应用中。数据库的 client 程序与 server 程序建立一个 TCP 连接以后，就在这个连接中传送数据库的多次交易过程。

/usr/local/bin/netperf -H 10.165.112.121 -l 10 -t TCP_RR  
![](https://img-blog.csdnimg.cn/20191017191002739.png)  
Netperf 输出的结果也是由两行组成。  
第一行显示本地系统的情况，  
第二行显示的是远端系统的信息。  
数据显示网络可支持交易速率 psh/ack 为 3642.74 次 / 秒。

注意到这里每次交易中的 request (PSH) 和 response (ACK) 分组的大小都为 1K 个字节，不具有很大的实际意义。

可以通过测试相关的参数来改变 request 和 response 分组的大小，TCP_RR 方式下的参数如下表所示：  
参数 说明  
-r req,resp 设置 request 和 reponse 分组的大小

通过使用 - r 参数，下面使用 32 个字节的请求和 2048 个字节 的应答, 我们可以进行更有实际意义的测试：  
netperf -H 10.165.112.121 -l 10 -t TCP_RR -r 32,2048

![](https://img-blog.csdnimg.cn/20191017194848506.png)

**2． TCP_CRR**

与 TCP_RR 不同，TCP_CRR 为每次交易建立一个新的 TCP 连接。最典型的应用就是 HTTP，每次 HTTP 交易是在一条单独的 TCP 连接中进行的。因此，由于需要不停地建立新的 TCP 连接，并且在交易结束后拆除 TCP 连接，交易率一定会受到很大的影响。

netperf -H 10.165.112.121 -l 10 -t TCP_CRR -r 2048,32768

![](https://img-blog.csdnimg.cn/20191017194927718.png)  
请求交易速率也明显的降低了，只有 1607.33 次 / 秒。  
 

5、sar 查看网卡性能
------------

sar 查看网卡性能：sar -n DEV 1 100

Linux 2.6.32-431.20.3.el6.x86_64 (iZ25ug3hg9iZ)         09/18/2014      _x86_64_        (4 CPU)

04:01:23 PM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s  
04:01:24 PM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00  
04:01:24 PM      eth0      4.04      0.00      0.16      0.00      0.00      0.00      0.00  
04:01:24 PM      eth1     26.26      0.00      1.17      0.00      0.00      0.00      0.00

各列含义：

IFACELAN 接口

rxpck/s 每秒钟接收的数据包

txpck/s 每秒钟发送的数据包

**rxbyt/s 或者 rxkB/s 每秒钟接收的字节数（上传速度，网卡入流量）**

**txbyt/s 或者 txkB/s 每秒钟发送的字节数（下载速度，网卡出流量）**

rxcmp/s 每秒钟接收的压缩数据包

txcmp/s 每秒钟发送的压缩数据包

rxmcst/s 每秒钟接收的多播数据包

其实中间的 IFACELAN bond0 是虚拟设备。在 RH 中，多个物理网卡帮定为一个逻辑 bonding 设备，通过把多个物理网卡帮定为一个逻辑设备，可以实现增加带宽吞吐量，提供冗余。如何进行虚拟化和模式选择，请参考下面两篇文章。

6、实际遇到的问题
---------

### 我们实际遇到的问题：

我们在服务器 tcpdump 抓包发现存在的 TCP Retransmission 的现象

![](https://img-blog.csdnimg.cn/20191018170304631.png)

![](https://img-blog.csdnimg.cn/20191018170202183.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hndWlzdQ==,size_16,color_FFFFFF,t_70)

从图中可以看到 TCP 三次握手的第一步 SYN 出现重传。客户端发送 SYN 后如果没有收到服务器放的 ACK 确认，就会导致重传的发生，因为客户端机器认为远程机器没有收到包，而发生重新发送 SYN 包的事件。

既然在服务器上抓包能捕获 SYN 的请求，那就说明服务器端接收到了请求但是没有回应 ACK 包：

我们查看内核参数中打开了 net.ipv4.tcp_tw_recycle = 1, 在 tcp_tw_recycle 模式下，服务器判断是无效连接的条件是：

**1、来自对端的 tcp syn 请求携带时间戳**

**2、本机在 MSL 时间内接收过来自同一台 ip 机器的 tcp 数据**

**3、新连接的时间戳小于上次 tcp 数据的时间戳**

以上条件满足时，连接请求会被拒绝

因此在高并发的情况下，就有可能导致服务器收到 SYN 但是不会向客户端发送 SYN+ACK 包。因为 MSL 时间内接收过来自同一台 ip 机器的 tcp 数据，导致服务器认为包不可信而丢弃。

 我们通过 netperf 监测 TCP 的应答 / 响应性能对比：

**打开 net.ipv4.tcp_tw_recycle = 1，请求交易速率只有 594.97 次 / 秒：**

![](https://img-blog.csdnimg.cn/20191021144520243.png)

**关闭 net.ipv4.tcp_tw_recycle = 0，请求交易速率提高到了 1406.29 次 / 秒：**

![](https://img-blog.csdnimg.cn/2019102114464134.png)