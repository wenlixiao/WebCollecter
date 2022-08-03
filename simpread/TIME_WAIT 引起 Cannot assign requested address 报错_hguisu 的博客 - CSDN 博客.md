> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/hguisu/article/details/10241519)

1.  问题描述


------------

     有时候用 redis 客户端（php 或者 java 客户端）连接 Redis 服务器，报错：“Cannot assign requested address。”

     原因是客户端频繁的连接服务器，由于每次连接都在很短时间内结束，导致很多的 TIME_WAIT。所以新的连接没办法绑定端口，即 “Cannot assign requested address”。

     我们可以通过 netstat -nat | grep 127.0.0.1:6380 查看连接 127.0.0.1:6380 的状态。你会发现很多 TIME_WAIT。

     很多人想到要用修改内核参数来解决：

     执行命令修改如下 2 个内核参数    
     sysctl -w net.ipv4.tcp_timestamps=1  开启对于 TCP 时间戳的支持, 若该项设置为 0，则下面一项设置不起作用  
     sysctl -w net.ipv4.tcp_tw_recycle=1  表示开启 TCP 连接中 TIME-WAIT sockets 的快速回收

     其实不然，根本没有理解出现这个问题的本质原因。首先我们了解 Redis 处理客户端连接的机制和 TCP 的 TIME_WAIT.

2.  Redis 处理客户端连接机制


-----------------------

（参考：http://redis.io/topics/clients）

1、建立连接（TCP 连接）：

      Redis 通过监听一个 TCP 端口或者 Unix socket 的方式来接收来自客户端的连接，当一个连接建立后，Redis 内部会进行以下一些操作：

*        首先，客户端 socket 会被设置为非阻塞模式，因为 Redis 在网络事件处理上采用的是非阻塞多路复用模型。
*        然后为这个 socket 设置 TCP_NODELAY 属性，禁用 Nagle 算法
*        然后创建一个 readable 的文件事件用于监听这个客户端 socket 的数据发送

     当客户端连接被初始化后，Redis 会查看目前的连接数，然后对比配置好的 maxclients 值，如果目前连接数已经达到最大连接数 maxclients 了，那么说明这个连接不能再接收，Redis 会直接返回客户端一个连接错误，并马上关闭掉这个连接。

2、服务器处理顺序

     如果有多个客户端连接上 Redis，并且都向 Redis 发送命令，那么 Redis 服务端会先处理哪个客户端的请求呢？答案其实并不确定，主要与两个因素有关，一是客户端对应的 socket 对应的数字的大小，二是 kernal 报告各个客户端事件的先后顺序。

Redis 处理一个客户端传来数据的步骤如下：

*         它对触发事件的 socket 调用一次 read()，只读一次（而不是把这个 socket 上的消息读完为止），是为了防止由于某个别客户端持续发送太多命令，导致其它客户端的请求长时间得不到处理的情况。
*   当然，当这一次 read() 调用完成后，它里面无论包含多少个命令，都会被一次性顺序地执行。这样就保证了对各个客户端命令的公平对待。
*     
    
*   3、关于最大连接数 maxclients

       在 Redis2.4 中，最大连接数是被直接硬编码在代码里面的，而在 2.6 版本中这个值变成可配置的。maxclients 的默认值是 10000，你也可以在 redis.conf 中对这个值进行修改。

      当然，这个值只是 Redis 一厢情愿的值，Redis 还会照顾到系统本身对进程使用的文件描述符数量的限制。在启动时 Redis 会检查系统的 soft limit，以查看打开文件描述符的个数上限。如果系统设置的数字，小于咱们希望的最大连接数加 32，那么这个 maxclients 的设置将不起作用，Redis 会按系统要求的来设置这个值。（加 32 是因为 Redis 内部会使用最多 32 个文件描述符，所以连接能使用的相当于所有能用的描述符号减 32）。

       当上面说的这种情况发生时（maxclients 设置后不起作用的情况），Redis 的启动过程中将会有相应的日志记录。比如下面命令希望设置最大客户端数量为 100000，所以 Redis 需要 100000+32 个文件描述符，而系统的最大文件描述符号设置为 10144，所以 Redis 只能将 maxclients 设置为 10144 – 32 = 10112。

$ ./redis-server --maxclients 100000  
[41422] 23 Jan 11:28:33.179 # Unable to set the max number of files limit to 100032 (Invalid argument), setting the max clients configuration to 10112.

        所以说当你想设置 maxclients 值时，最好顺便修改一下你的系统设置，当然，养成看日志的好习惯也能发现这个问题。

具体的设置方法就看你个人的需求了，你可以只修改此次会话的限制，也可以直接通过 sysctl 修改系统的默认设置。如：

ulimit -Sn 100000 # This will only work if hard limit is big enough.  
sysctl -w fs.file-max=100000

4、输出缓冲区大小限制

       对于 Redis 的输出（也就是命令的返回值）来说，其大小经常是不可控的，可能是一个简单的命令，能够产生体积庞大的返回数据。另外也有可能因为执行命令太多，产生的返回数据的速率超过了往客户端发送的速率，这时也会产生消息堆积，从而造成输出缓冲区越来越大，占用过多内存，甚至导致系统崩溃。

      所以 Redis 设置了一些保护机制来避免这种情况的出现，这些机制作用于不同种类的客户端，有不同的输出缓冲区大小限制，限制方式有两种：

*         一种是大小限制，当某一个客户端的缓冲区超过某一大小时，直接关闭掉这个客户端连接
*        另一种是当某一个客户端的缓冲区持续一段时间占用空间过大时，也直接关闭掉客户端连接

对于不同客户端的策略如下：

*          对普通客户端来说，限制为 0，也就是不限制，因为普通客户端通常采用阻塞式的消息应答模式，如：发送请求，等待返回，再发请求，再等待返回。这种模式通常不会导致输出缓冲区的堆积膨胀。
*          对于 Pub/Sub 客户端来说，大小限制是 32m，当输出缓冲区超过 32m 时，会关闭连接。持续性限制是，当客户端缓冲区大小持续 60 秒超过 8m，也会导致连接关闭。
*          而对于 Slave 客户端来说，大小限制是 256m，持续性限制是当客户端缓冲区大小持续 60 秒超过 64m 时，关闭连接。

上面三种规则都是可配置的。可以通过 CONFIG SET 命令或者修改 redis.conf 文件来配置。

5、输入缓冲区大小限制

      Redis 对输入缓冲区大小的限制比较暴力，当客户端传输的请求大小超过 1G 时，服务端会直接关闭连接。这种方式可以有效防止一些客户端或服务端 bug 导致的输入缓冲区过大的问题。

6、Client 超时

      对当前的 Redis 版本来说，服务端默认是不会关闭长期空闲的客户端的。但是你可以修改默认配置来设置你希望的超时时间。比如客户端超过多长时间无交互，就直接关闭。同理，这也可以通过 CONFIG SET 命令或者修改 redis.conf 文件来配置。

      值得注意的是，超时时间的设置，只对普通客户端起作用，对 Pub/Sub 客户端来说，长期空闲状态是正常的。

      另外，实际的超时时间可能不会像设定的那样精确，这是因为 Redis 并不会采用计时器或者轮训遍历的方法来检测客户端超时，而是通过一种渐近式的方式来完成，每次检查一部分。所以导致的结果就是，可能你设置的超时时间是 10s，但是真实执行的时间是超时 12s 后客户端才被关闭。

式。

3.  TCP 的 TIME_WAIT 状态


--------------------------

    主动关闭的 Socket 端会进入 TIME_WAIT 状态，并且持续 2MSL 时间长度，MSL 就是 maximum segment lifetime(最大分节生命期），在 windows 下默认 240 秒，MSL 是一个 IP 数据包能在互联网上生存的最长时间，超过这个时间将在网络中消失。MSL 在 RFC 1122 上建议是 2 分钟，而源自 berkeley 的 TCP 实现传统上使用 30 秒，因而，TIME_WAIT 状态一般维持在 1-4 分钟。

**TIME_WAIT 状态存在的理由：**

**1）可靠地实现 TCP 全双工连接的终止：（即在 TIME_WAIT 下等待 2MSL，只是为了尽最大努力保证四次握手正常关闭）。**

  TCP 协议规定，对于已经建立的连接，网络双方要进行四次握手才能成功断开连接，如果缺少了其中某个步骤，将会使连接处于假死状态，连接本身占用的资源不会被释放。

    在进行关闭连接四路握手协议时，最后的 ACK 是由主动关闭端发出的，如果这个最终的 ACK 丢失，服务器将重发最终的 FIN，因此客户端必须维护状态信息允许它重发最终的 ACK。如果不维持这个状态信息，那么客户端将响应 RST 分节，因而，要实现 TCP 全双工连接的正常终止，必须处理终止序列四个分节中任何一个分节的丢失情况，主动关闭的客户端必须维持状态信息进入 TIME_WAIT 状态。

    我们看客户端主动关闭服务器被动关闭四次握手的流程：

1、 客户端发送 FIN 报文段，进入 FIN_WAIT_1 状态。

2、 服务器端收到 FIN 报文段，发送 ACK 表示确认，进入 CLOSE_WAIT 状态。

3、 客户端收到 FIN 的确认报文段，进入 FIN_WAIT_2 状态。

4、 服务器端发送 FIN 报文端，进入 LAST_ACK 状态。

5、 客户端收到 FIN 报文端，发送 FIN 的 ACK，同时进入 TIME_WAIT 状态，启动 TIME_WAIT 定时器，超时时间设为 2MSL。

6、 服务器端收到 FIN 的 ACK，进入 CLOSED 状态。

7、 客户端在 2MSL 时间内没收到对端的任何响应，TIME_WAIT 超时，进入 CLOSED 状态。

      如果不考虑报文延迟、丢失，确认延迟、丢失等情况，TIME_WAIT 的确没有存在的必要。当网络在不理想的情况下通常会有报文的丢失延迟发生，让我们看下面的一个特例：

     客户端进入发送收到四次握手关闭的最后一个 ACK 后，进入 TIME_WAIT 同时发送 ACK，如果其不停留 2MSL 时间，而是马上关闭连接，销毁连接上的资源，当发送如下情况时，将不能正常的完成四次握手关闭：

客户端发送的 ACK 在网路上丢失，这样服务器端收不到最后的 ACK，重传定时器超时，将重传 FIN 到客户端，由于客户端关于该连接的所有资源都释放，收到重传的 FIN 后，它没有关于这个 FIN 的任何信息，所以向服务器端发送一个 RST 报文端，服务器端收到 RST 后，认为搞连接出现了异常（而非正常关闭）。

所以，在 TIME_WAIT 状态下等待 2MSL 时间端，是为了能够正确处理第一个 ACK（最长生存时间为 MSL）丢失的情况下，能够收到对端重传的 FIN（最长生存时间为 MSL），然后重传 ACK。

     是否只要主动关闭方在 TIME_WAIT 状态下停留 2MSL，四次握手关闭就一定正常完成呢？

     答案是否定的？可以考虑如下的情况， 

     TIME_WAIT 状态下发送的 ACK 丢失，LAST_ACK 时刻设定的重传定时器超时，发送重传的 FIN，很不幸，这个 FIN 也丢失，主动关闭方在 TIME_WAIT 状态等待 2MSL 没收到任何报文段，进入 CLOSED 状态，当此时被动关闭方并没有收到最后的 ACK。所以即使要主动关闭方在 TIME_WAIT 状态下停留 2MSL，也不一定表示四次握手关闭就一定正常完成。

**2）确保老的报文段在网络中消失，不会影响新建立的连接** 

        考虑如下的情况，主动关闭方在 TIME_WAIT 状态下发送的 ACK 由于网络延迟的原因没有按时到底（但并没有超过 MSL 的时间），导致被动关闭方重传 FIN，在 FIN 重传后，延迟的 ACK 到达，被动关闭方进入 CLOSED 状态，如果主动关闭方在 TIME_WAIT 状态下发送 ACK 后马上进入 CLOSED 状态（也就是没有等待）2MSL 时间，则上述的连接已不存在：

       现在考虑下面的情况，假设客户端（192.186.0.1:23） 到服务器 192.168.1.1:6380）的 TCP 连接, 由于连接已关闭，我们可以马上建立一个相同的 IP 地址和端口之间的 TCP 连接，并且这个连接也是客户端（192.186.0.1:23） 到服务器 192.168.1.1:6380），那么当上一个连接的重传 FIN 到达主动关闭方时，被新的连接所接受，这将导致新的连接被复位，很显然，这不是我们希望看到的事情。

       新的连接要建立，必须是在主动关闭方和被动关闭方都进入到 CLOSED 状态之后才有可能。所以，最有可能导致旧的报文段影响新的连接的情况是：

      在 TIME_WAIT 状态之前，主动关闭方发送的报文端在网络中延迟，但是 TIME_WAIT 设定为 2MSL 时，这些报文端必然会在网络中消失（最大生存时间为 MSL）。被动关闭方最有可能影响新连接的报文段就是我们上面讨论的情况，对方 ACK 延迟到达，在此之前重传的 FIN, 这个报文端发送之后，TIME_WAIT 的定时器超时时间肯定大于 MSL，在 1MSL 时间内，这个 FIN 要么在网络中因为生成时间到达而消失，要么到达主动关闭方被这确的处理，不会影响新建立的连接。

    新的 SCTP 协议通过在消息头部添加验证标志避免了 TIME_WAIT 状态。

**3）有关内核级别的 keepalive 和 time_wait 的优化调整**

有关内核级别的 keepalive 和 time_wait 的优化调整  
vi /etc/sysctl  
net.ipv4.tcp_tw_reuse = 1  
net.ipv4.tcp_tw_recycle = 1  
net.ipv4.tcp_keepalive_time = 1800  
net.ipv4.tcp_fin_timeout = 30  
net.core.netdev_max_backlog =8096  
  
修改完记的使用 sysctl -p 让它生效  
以上参数的注解  
/proc/sys/net/ipv4/tcp_tw_reuse  
该文件表示是否允许重新应用处于 TIME-WAIT 状态的 [socket](https://so.csdn.net/so/search?q=socket&spm=1001.2101.3001.7020) 用于新的 TCP 连接。  
  
/proc/sys/net/ipv4/tcp_tw_recycle  
recyse 是加速 TIME-WAIT sockets 回收  
  
对 tcp_tw_reuse 和 tcp_tw_recycle 的修改，可能会出现. warning, got duplicate tcp line warning, got BOGUS tcp line. 上面这二个参数指的是存在这两个完全一样的 TCP 连接，这会发生在一个连接被迅速的断开并且重新连接的情况，而且使用的端口和地址相同。但基本 上这样的事情不会发生，无论如何，使能上述设置会增加重现机会。这个提示不会有人和危害，而且也不会降低系统性能，目前正在进行工作  
  
/proc/sys/net/ipv4/tcp_keepalive_time  
表示当 keepalive 起用的时候, TCP 发送 keepalive 消息的频度。缺省是 2 小时  
  
/proc/sys/net/ipv4/tcp_fin_timeout 最佳值和 BSD 一样为 30  
fin_wait1 状态是在发起端主动要求关闭 tcp 连接，并且主动发送 fin 以后，等待接收端回复 ack 时候的状态。对于本端断开的 socket 连接，TCP 保持在 FIN-WAIT-2 状态的时间。对方可能会断开连接或一直不结束连接或不可预料的进程死亡。  
  
/proc/sys/net/core/netdev_max_backlog  
该文件指定了，在接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目  

**4）time_wait 的优化处理**  

Linux 系统中 TCP 是面向连接的, 在实际应用中通常都需要检测连接是否还可用. 如果不可用, 可分为:

a. 连接的对端正常关闭.

b. 连接的对端非正常关闭, 这包括对端设备掉电, 程序崩溃, 网络被中断等. 这种情况是不能也无法通知对端的, 所以连接会一直存在, 浪费国家的资源.

TCP 协议栈有个 keepalive 的属性, 可以主动探测 socket 是否可用, 不过这个属性的默认值很大.

全局设置可更改 / etc/sysctl.conf, 加上:

net.ipv4.tcp_keepalive_intvl = 20  
net.ipv4.tcp_keepalive_probes = 3  
net.ipv4.tcp_keepalive_time = 60

  
在程序中设置如下:

int keepAlive = 1; // 开启 keepalive 属性  
int keepIdle = 60; // 如该连接在 60 秒内没有任何数据往来, 则进行探测  
int keepInterval = 5; // 探测时发包的时间间隔为 5 秒  
int keepCount = 3; // 探测尝试的次数. 如果第 1 次探测包就收到响应了, 则后 2 次的不再发.

setsockopt(rs, SOL_SOCKET, SO_KEEPALIVE, (void *)&keepAlive, sizeof(keepAlive));

setsockopt(rs, SOL_TCP, TCP_KEEPIDLE, (void*)&keepIdle, sizeof(keepIdle));

setsockopt(rs, SOL_TCP, TCP_KEEPINTVL, (void *)&keepInterval, sizeof(keepInterval));

setsockopt(rs, SOL_TCP, TCP_KEEPCNT, (void *)&keepCount, sizeof(keepCount));

4.  解决问题


------------

    我们了解 Redis 处理

客户端连接的机制和 TCP 的 TIME_WAIT. 我们可以重现上述问题，我们快速建立 2000 个连接，

```
<?php
$num = 2000;
for($i=0; $i<$num; $i++) {
	$redis = new Redis();
	$redis->connect('127.0.0.1',6379);
	//sleep(1);
}
sleep(10);
```

然后查看状态：

netstat -nat | grep 127.0.0.1:6379

你会发现很多 TIME_WAIT。

如果 $num 加大到 40000 或者，报错：Cannot assign requested address。

    因此如果客户端（php）连接 redis 出现这个问题，说明你程序出现 bug 了。你某个循环里面实例化 Redis 了（即每次都 new Redis）, 造成每一次循环都建立一个连接。

   解决这个问题不是修改内核参数，而是把连接 redis 封装成单实例，确保在同一进程内，连接 redis 是唯一实例。

```
class Class_Redis {
 
	private $_redis;
	private static $_instance = null;
	
	private  function __construct() {
		$this->_redis = new Redis();
		$this->_redis->connect('127.0.0.1',6379);
 
	}
	
	public static function getInstance() {
		if(self::$_instance === null) {
			self::$_instance = new self();
		}
		return self::$_instance;
	
	}
 
	
	public  function getRedis() {
		return $this->_redis;
	}
 
}
```

5.  了解 redis 性能相关指标


-----------------------

我们通过 info 命令输出的数据可分为 10 个类别，分别是：

*   server
*   clients
*   memory
*   persistence：RDB 和 AOF 的相关信息
*   stats：一般统计信息
*   replication：主 / 从复制信息
*   cpu：CPU 计算量统计信息
*   commandstats：Redis 命令统计信息
*   cluster： Redis 集群信息
*   keyspace：[据库](http://lib.csdn.net/base/14 "MySQL知识库")相关的统计信息

info 命令可以添加参数来获取单个分类下的数据。比如输入 info memory 命令，会只返回与内存相关的数据。

### 1、内存指标

used_memory 字段数据表示的是：由 Redis 分配器分配的内存总量，以字节（byte）为单位。 其中 used_memory_human 上的数据和 used_memory 是一样的值，它以 M 为单位显示，仅为了方便阅读。  
# Memory  
used_memory:1062992  
used_memory_human:1.01M  
used_memory_rss:6537216  
used_memory_peak:1973720  
used_memory_peak_human:1.88M  
used_memory_lua:33792  
mem_fragmentation_ratio:6.15  
mem_allocator:jemalloc-3.6.0

used_memory：是 Redis 使用的内存总量，它包含了实际缓存占用的内存和 Redis 自身运行所占用的内存 (如元数据、lua)。它是由 Redis 使用内存分配器分配的内存，所以这个数据并没有把内存碎片浪费掉的内存给统计进去。

其他字段代表的含义，都以字节为单位：

*   used_memory_rss：从操作系统上显示已经分配的内存总量（俗称常驻集大小）。这个值和 top 、 ps 等命令的输出一致。
*   used_memory_peak：Redis 的内存消耗峰值（以字节为单位）  
    
*   mem_fragmentation_ratio： 内存碎片率。used_memory_rss 和 used_memory 之间的比率
*   used_memory_lua： Lua 脚本引擎所使用的内存大小。
*   mem_allocator： 在编译时指定的 Redis 使用的内存分配器，可以是 libc、jemalloc、tcmalloc。

  
      在理想情况下，  used_memory_rss 的值应该只比  used_memory 稍微高一点儿。       当  rss > used ，且两者的值相差较大时，表示存在（内部或外部的）内存碎片。内存碎片的比率可以通过  mem_fragmentation_ratio 的值看出。       当  used > rss  时， 表示 Redis 的部分内存被操作系统换出到交换空间了，在这种情况下，操作可能会产生明显的延迟。

       Because Redis does not have control over how its allocations are mapped to memory pages, high used_memory_rss is often the result of a spike in memory usage.

     当 Redis 释放内存时，分配器可能会，也可能不会，将内存返还给操作系统。      如果 Redis 释放了内存，却没有将内存返还给操作系统，那么  used_memory 的值可能和操作系统显示的 Redis 内存占用并不一致。 查看  used_memory_peak 的值可以验证这种情况是否发生。下面详细说明:  

### 2、内存碎片率

      info 信息中的 mem_fragmentation_ratio 给出了内存碎片率的数据指标，它是由操系统分配的内存除以 Redis 分配的内存得出：

 used_memory_rss 和 used_memory 之间的比率  

    used_memory 和 used_memory_rss 数字都包含的内存分配有：

*   用户定义的数据：内存被用来存储 key-value 值。
*   内部开销： 存储内部 Redis 信息用来表示不同的数据类型。

      used_memory_rss 的 rss 是 Resident Set Size 的缩写，表示该进程所占物理内存的大小，是操作系统分配给 Redis 实例的内存大小。除了用户定义的数据和内部开销以外，used_memory_rss 指标还包含了内存碎片的开销，内存碎片是由操作系统低效的分配 / 回收物理内存导致的。  
    操作系统负责分配物理内存给各个应用进程，Redis 使用的内存与物理内存的映射是由操作系统上虚拟内存管理分配器完成的。  
   举个例子来说，Redis 需要分配连续内存块来存储 1G 的数据集，这样的话更有利，但可能物理内存上没有超过 1G 的连续内存块，那操作系统就不得不      使用多个不连续的小内存块来分配并存储这 1G 数据，也就导致内存碎片的产生。  
   内存分配器另一个复杂的层面是，它经常会预先分配一些内存块给引用，这样做会使加快应用程序的运行。

       跟踪内存碎片率对理解 Redis 实例的资源性能是非常重要的。

       内存碎片率稍大于 1 是合理的：这个值表示内存碎片率比较低，也说明 redis 没有发生内存交换。

      如果内存碎片率超过 1.5：那就说明 Redis 消耗了实际需要物理内存的 150%，其中 50% 是内存碎片率。

     若是内存碎片率低于 1 的话：说明 Redis 内存分配超出了物理内存，操作系统正在进行内存交换。内存交换会引起非常明显的响应延迟。

### 3、用内存碎片率超过 1.5

      倘若内存碎片率超过了 1.5，那可能是操作系统或 Redis 实例中内存管理变差的表现。下面的方法解决内存管理变差的问题，并提高 Redis 性能：

 **1. 重启 Redis 服务器：**如果内存碎片率超过 1.5，重启 Redis 服务器可以让额外产生的内存碎片失效并重新作为新内存来使用，使操作系统恢复高效的内存管理。额外碎片的产生是由于 Redis 释放了内存块，但内存分配器并没有返回内存给操作系统，这个内存分配器是在编译时指定的，可以是 libc、jemalloc 或者 tcmalloc。 通过比较 used_memory_peak, used_memory_rss 和 used_memory_metrics 的数据指标值可以检查额外内存碎片的占用。从名字上可以看出，used_memory_peak 是过去 Redis 内存使用的峰值，而不是当前使用内存的值。如果 used_memory_peak 和 used_memory_rss 的值大致上相等，而且二者明显超过了 used_memory 值，这说明额外的内存碎片正在产生。 

      在重启服务器之前，需要在 Redis-cli 工具上输入 shutdown save 命令，意思是强制让 Redis 数据库执行保存操作并关闭 Redis 服务，这样做能保证在执行 Redis 关闭时不丢失任何数据。 在重启后，Redis 会从硬盘上加载持久化的文件，以确保数据集持续可用。

 **2. 修改内存分配器：**  
      Redis 支持 glibc’s malloc、jemalloc11、tcmalloc 几种不同的内存分配器，每个分配器在内存分配和碎片上都有不同的实现。不建议普通管理员修改 Redis 默认内存分配器，因为这需要完全理解这几种内存分配器的差异，也要重新编译 Redis。这个方法更多的是让其了解 Redis 内存分配器所做的工作，当然也是改善内存碎片问题的一种办法。

### 3、用内存碎片率低于 1：内存交换的性能问题

           如果一个 Redis 实例的内存使用率超过可用最大内存 (used_memory> used_memory_rss 操作系统分配可用最大内存)，那么操作系统开始进行内存与 swap 空间交换，把内存中旧的或不再使用的内容写入硬盘上（硬盘上的这块空间叫 Swap 分区），以便腾出新的物理内存给新页或活动页 (page) 使用。   
           在硬盘上进行读写操作要比在内存上进行读写操作，时间上慢了近 5 个数量级，内存是 0.1μs 单位、而硬盘是 10ms。如果 Redis 进程上发生内存交换，那么 Redis 和依赖 Redis 上数据的应用会受到严重的性能影响。

           若是在使用 Redis 期间没有开启 rdb 快照或 aof 持久化策略，那么缓存数据在 Redis 崩溃时就有丢失的危险。因为当 Redis 内存使用率超过可用内存的 95% 时，部分数据开始在内存与 swap 空间来回交换，这时就可能有丢失数据的危险。  
          当开启并触发快照功能时，Redis 会 fork 一个子进程把当前内存中的数据完全复制一份写入到硬盘上。因此若是当前使用内存超过可用内存的 45% 时触发快照功能，那么此时进行的内存交换会变的非常危险 (可能会丢失数据)。 倘若在这个时候实例上有大量频繁的更新操作，问题会变得更加严重。

通过减少 Redis 的内存占用率，来避免这样的问题，或者使用下面的技巧来避免内存交换发生：

1.  假如缓存数据小于 4GB，就使用 32 位的 Redis 实例。因为 32 位实例上的指针大小只有 64 位的一半，它的内存空间占用空间会更少些。 这有一个坏处就是，假设物理内存超过 4GB，那么 32 位实例能使用的内存仍然会被限制在 4GB 以下。 要是实例同时也共享给其他一些应用使用的话，那可能需要更高效的 64 位 Redis 实例，这种情况下切换到 32 位是不可取的。 不管使用哪种方式，Redis 的 dump 文件在 32 位和 64 位之间是互相兼容的， 因此倘若有减少占用内存空间的需求，可以尝试先使用 32 位，后面再切换到 64 位上。
    
2.  尽可能的使用 Hash 数据结构。因为 Redis 在储存小于 100 个字段的 Hash 结构上，其存储效率是非常高的。所以在不需要集合 (set) 操作或 list 的 push/pop 操作的时候，尽可能的使用 Hash 结构。比如，在一个 web 应用程序中，需要存储一个对象表示用户信息，使用单个 key 表示一个用户，其每个属性存储在 Hash 的字段里，这样要比给每个属性单独设置一个 key-value 要高效的多。 通常情况下倘若有数据使用 string 结构，用多个 key 存储时，那么应该转换成单 key 多字段的 Hash 结构。 如上述例子中介绍的 Hash 结构应包含，单个对象的属性或者单个用户各种各样的资料。Hash 结构的操作命令是 HSET(key, fields, value)和 HGET(key, field)，使用它可以存储或从 Hash 中取出指定的字段。
    
3.  设置 key 的过期时间。一个减少内存使用率的简单方法就是，每当存储对象时确保设置 key 的过期时间。倘若 key 在明确的时间周期内使用或者旧 key 不大可能被使用时，就可以用 Redis 过期时间命令 (expire,expireat, pexpire, pexpireat) 去设置过期时间，这样 Redis 会在 key 过期时自动删除 key。 假如你知道每秒钟有多少个新 key-value 被创建，那可以调整 key 的存活时间，并指定阀值去限制 Redis 使用的最大内存。
    
4.  回收 key。在 Redis 配置文件中 (一般叫 Redis.conf)，通过设置“maxmemory” 属性的值可以限制 Redis 最大使用的内存，修改后重启实例生效。 也可以使用客户端命令 config set maxmemory 去修改值，这个命令是立即生效的，但会在重启后会失效，需要使用 config rewrite 命令去刷新配置文件。 若是启用了 Redis 快照功能，应该设置 “maxmemory” 值为系统可使用内存的 45%，因为快照时需要一倍的内存来复制整个数据集，也就是说如果当前已使用 45%，在快照期间会变成 95%(45%+45%+5%)，其中 5% 是预留给其他的开销。 如果没开启快照功能，maxmemory 最高能设置为系统可用内存的 95%。
    

        当内存使用达到设置的最大阀值时，需要选择一种 key 的回收策略，可在 Redis.conf 配置文件中修改 “maxmemory-policy” 属性值。 若是 Redis 数据集中的 key 都设置了过期时间，那么 “volatile-ttl” 策略是比较好的选择。但如果 key 在达到最大内存限制时没能够迅速过期，或者根本没有设置过期时间。那么设置为 “allkeys-lru” 值比较合适，它允许 Redis 从整个数据集中挑选最近最少使用的 key 进行删除(LRU 淘汰算法)。Redis 还提供了一些其他淘汰策略，如下：

*   volatile-lru：使用 LRU 算法从已设置过期时间的数据集合中淘汰数据。
*   volatile-ttl：从已设置过期时间的数据集合中挑选即将过期的数据淘汰。
*   volatile-random：从已设置过期时间的数据集合中随机挑选数据淘汰。
*   allkeys-lru：使用 LRU 算法从所有数据集合中淘汰数据。
*   allkeys-random：从数据集合中任意选择数据淘汰
*   no-enviction：禁止淘汰数据。

          通过设置 maxmemory 为系统可用内存的 45% 或 95%(取决于持久化策略)和设置 “maxmemory-policy” 为“volatile-ttl”或“allkeys-lru”(取决于过期设置)，可以比较准确的限制 Redis 最大内存使用率，在绝大多数场景下使用这 2 种方式可确保 Redis 不会进行内存交换。倘若你担心由于限制了内存使用率导致丢失数据的话，可以设置 noneviction 值禁止淘汰数据。

 **2. 限制内存交换：**  如果内存碎片率低于 1，Redis 实例可能会把部分数据交换到硬盘上。内存交换会严重影响 Redis 的性能，所以应该增加可用物理内存或减少实 Redis 内存占用。

5.  回收 key


--------------

      info 信息中的 evicted_keys 字段显示的是，因为 maxmemory 限制导致 key 被回收删除的数量。关于 maxmemory 的介绍见前面章节，回收 key 的情况只会发生在设置 maxmemory 值后，不设置会发生内存交换。 当 Redis 由于内存压力需要回收一个 key 时，Redis 首先考虑的不是回收最旧的数据，而是在最近最少使用的 key 或即将过期的 key 中随机选择一个 key，从数据集中删除。

      这可以在配置文件中设置 maxmemory-policy 值为 “volatile-lru” 或“volatile-ttl”，来确定 Redis 是使用 lru 策略还是过期时间策略。 倘若所有的 key 都有明确的过期时间，那过期时间回收策略是比较合适的。若是没有设置 key 的过期时间或者说没有足够的过期 key，那设置 lru 策略是比较合理的，这可以回收 key 而不用考虑其过期状态。

### 根据 key 回收定位性能问题

跟踪 key 回收是非常重要的，因为通过回收 key，可以保证合理分配 Redis 有限的内存资源。如果 evicted_keys 值经常超过 0，那应该会看到客户端命令响应延迟时间增加，因为 Redis 不但要处理客户端过来的命令请求，还要频繁的回收满足条件的 key。  
需要注意的是，回收 key 对性能的影响远没有内存交换严重，若是在强制内存交换和设置回收策略做一个选择的话，选择设置回收策略是比较合理的，因为把内存数据交换到硬盘上对性能影响非常大 (见前面章节)。

### 减少回收 key 以提升性能

减少回收 key 的数量是提升 Redis 性能的直接办法，下面有 2 种方法可以减少回收 key 的数量：

**1. 增加内存限制：**倘若开启快照功能，maxmemory 需要设置成物理内存的 45%，这几乎不会有引发内存交换的危险。若是没有开启快照功能，设置系统可用内存的 95% 是比较合理的，具体参考前面的快照和 maxmemory 限制章节。如果 maxmemory 的设置是低于 45% 或 95%(视持久化策略)，通过增加 maxmemory 的值能让 Redis 在内存中存储更多的 key，这能显著减少回收 key 的数量。 若是 maxmemory 已经设置为推荐的阀值后，增加 maxmemory 限制不但无法提升性能，反而会引发内存交换，导致延迟增加、性能降低。 maxmemory 的值可以在 Redis-cli 工具上输入 config set maxmemory 命令来设置。  
需要注意的是，这个设置是立即生效的，但重启后丢失，需要永久化保存的话，再输入 config rewrite 命令会把内存中的新配置刷新到配置文件中。

**2. 对实例进行分片：**分片是把数据分割成合适大小，分别存放在不同的 Redis 实例上，每一个实例都包含整个数据集的一部分。通过分片可以把很多服务器联合起来存储数据，相当于增加总的物理内存，使其在没有内存交换和回收 key 的策略下也能存储更多的 key。假如有一个非常大的数据集，maxmemory 已经设置，实际内存使用也已经超过了推荐设置的阀值，那通过数据分片能明显减少 key 的回收，从而提高 Redis 的性能。 分片的实现有很多种方法，下面是 Redis 实现分片的几种常见方式：

*   a. Hash 分片：一个比较简单的方法实现，通过 Hash 函数计算出 key 的 Hash 值，然后值所在范围对应特定的 Redis 实例。
*   b. 代理分片：客户端把请求发送到代理上，代理通过分片配置表选择对应的 Redis 实例。 如 Twitter 的 Twemproxy，豌豆荚的 codis。
*   c. 一致性 Hash 分片： 参见前面博客《[一致性 Hash 分片详解](http://www.cnblogs.com/mushroom/p/4472369.html)》
*   d. 虚拟桶分片：参见前面博客《[虚拟桶分详解](http://www.cnblogs.com/mushroom/p/4542772.html)》