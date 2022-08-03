> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/hguisu/article/details/38700899)

我们通过了解 TCP 各个状态，可以排除和定位网络或系统故障时大有帮助。

一、TCP 网络常用命令
============

**了解 TCP 之前，先了解几个命令：**

linux 查看 tcp 的状态命令：  
1）、netstat -nat  查看 TCP 各个状态的数量  
2）、lsof  -i:port  可以检测到打开[套接字](https://so.csdn.net/so/search?q=%E5%A5%97%E6%8E%A5%E5%AD%97&spm=1001.2101.3001.7020)的状况  
3)、  sar -n SOCK 查看 tcp 创建的连接数  
4)、[tcpdump](https://so.csdn.net/so/search?q=tcpdump&spm=1001.2101.3001.7020) -iany tcp port 9000 对 tcp 端口为 9000 的进行抓包  
5)、tcpdump   dst  port 9000 -w dump9000.pcap  对 tcp 目标端口为 9000 的进行抓包保存 pcap 文件 wireshark 分析。  
6)、tcpdump tcp port 9000  -n -X -s 0 -w tcp.cap 对 tcp/http 目标端口为 9000 的进行抓包保存 pcap 文件 wireshark 分析。

**网络测试常用命令;** 

1）ping: 检测网络连接的正常与否, 主要是测试延时、抖动、丢包率。

     但是很多服务器为了防止攻击，一般会关闭对 ping 的响应。所以 ping 一般作为测试连通性使用。ping 命令后，会接收到对方发送的回馈信息，其中记录着对方的 IP 地址和 TTL。TTL 是该字段指定 IP 包被路由器丢弃之前允许通过的最大网段数量。TTL 是 IPv4 包头的一个 8 bit 字段。例如 IP 包在服务器中发送前设置的 TTL 是 64，你使用 ping 命令后，得到服务器反馈的信息，其中的 TTL 为 56，说明途中一共经过了 8 道路由器的转发，每经过一个路由，TTL 减 1。

2）traceroute：raceroute 跟踪数据包到达网络主机所经过的路由工具

   traceroute hostname

3）pathping：是一个路由跟踪工具，它将 ping 和 tracert 命令的功能与这两个工具所不提供的其他信息结合起来，综合了二者的功能

   pathping www.baidu.com

4）mtr：以结合 ping nslookup tracert 来判断网络的相关特性

5) nslookup: 用于解析域名，一般用来检测本机的 DNS 设置是否配置正确。

二、TCP 建立连接三次握手相关状态
==================

    TCP 是一个面向连接的协议，所以在连接双方发送数据之前，都需要首先建立一条连接。

**1、Client 连接 Server 三次握手过程：**
------------------------------

     当 Client 端调用 socket 函数调用时，相当于 Client 端产生了一个处于 Closed 状态的套接字。  
      **(1)  第一次握手 SYN：**Client 端又调用 connect 函数调用，系统为 Client 随机分配一个端口，连同传入 connect 中的参数 (Server 的 IP 和端口)，这就形成了一个连接四元组，客户端发送一个带 SYN 标志的 TCP 报文到服务器。这是三次握手过程中的报文 1。connect 调用让 Client 端的 socket 处于 **SYN_SENT** 状态，等待服务器确认；SYN：同步序列编号 (Synchronize Sequence Numbers)。

      **(2) 第二次握手 SYN+ACK：** 服务器收到 syn 包，必须确认客户的 SYN（ack=j+1），同时自己也发送一个 SYN 包（syn=k），即 SYN+ACK 包，此时服务器进入 **SYN_RECV** 状态；

      **(3)  第三次握手 ACK：**客户端收到服务器的 SYN+ACK 包，向服务器发送确认包 ACK(ack=k+1)，此包发送完毕，客户器和客务器进入 **ESTABLISHED** 状态，完成三次握手。连接已经可以进行读写操作。

一个完整的三次握手也就是： 请求 --- 应答 --- 再次确认。

TCP 协议通过三个报文段完成连接的建立，这个过程称为三次握手 (three-way handshake)，过程如下图所示。

![](https://img-blog.csdnimg.cn/20200712121141545.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hndWlzdQ==,size_16,color_FFFFFF,t_70)

**Server 对应的函数接口：**  
     当 Server 端调用 socket 函数调用时，相当于 Server 端产生了一个处于 Closed 状态的监听套接字  
      Server 端调用 bind 操作，将监听套接字与指定的地址和端口关联，然后又调用 listen 函数，系统会为其分配未完成队列和完成队列，此时的监听套接字可以接受 Client 的连接，监听套接字状态处于 LISTEN 状态。  
     当 Server 端调用 accept 操作时，会从完成队列中取出一个已经完成的 client 连接，同时在 server 这段会产生一个会话套接字，用于和 client 端套接字的通信，这个会话套接字的状态是 ESTABLISH。

        从图中可以看出，当客户端调用 connect 时，触发了连接请求，向服务器发送了 SYN J 包，这时 connect 进入阻塞状态；服务器监听到连接请求，即收到 SYN J 包，调用 accept 函数接收请求向客户端发送 SYN K ，ACK J+1，这时 accept 进入阻塞状态；客户端收到服务器的 SYN K ，ACK J+1 之后，这时 connect 返回，并对 SYN K 进行确认；服务器收到 ACK K+1 时，accept 返回，至此三次握手完毕，连接建立。

我们可以通过 wireshark 抓包，可以清晰看到查看具体的流程：

**第一次握手：**syn 的 seq= 0  
**第二次握手：**服务器收到 syn 包，必须确认客户的 SYN（ACK=j+1=1）=，同时自己也发送一个 SYN 包（syn=0）

**第三次握手：**客户端收到服务器的 SYN+ACK 包，向服务器发送确认包 ACK(ack=k+1)

![](https://img-blog.csdnimg.cn/20200712114643333.png)

**2、建立连接的具体状态：**
----------------

 **1)、LISTENING：侦听来自远方的 TCP 端口的连接请求.**

     首先服务端需要打开一个 socket 进行监听，状态为 LISTEN。

    有提供某种服务才会处于 LISTENING 状态，TCP 状态变化就是某个端口的状态变化，提供一个服务就打开一个端口，例如：提供 www 服务默认开的是 80 端口，提供 ftp 服务默认的端口为 21，当提供的服务没有被连接时就处于 LISTENING 状态。FTP 服务启动后首先处于侦听（LISTENING）状态。处于侦听 LISTENING 状态时，该端口是开放的，等待连接，但还没有被连接。就像你房子的门已经敞开的，但还没有人进来。  
    看 LISTENING 状态最主要的是看本机开了哪些端口，这些端口都是哪个程序开的，关闭不必要的端口是保证安全的一个非常重要的方面，服务端口都对应一个服务（应用程序），停止该服务就关闭了该端口，例如要关闭 21 端口只要停止 IIS 服务中的 FTP 服务即可。关于这方面的知识请参阅其它文章。  
    如果你不幸中了服务端口的木马，木马也开个端口处于 LISTENING 状态。

  **2)、SYN-SENT：客户端 SYN_SENT 状态**

         再发送连接请求后等待匹配的连接请求: 客户端通过应用程序调用 connect 进行 active open. 于是客户端 tcp 发送一个 SYN 以请求建立一个连接. 之后状态置为 SYN_SENT. /*The socket is actively attempting to establish a connection. 在发送连接请求后等待匹配的连接请求 */  
     当请求连接时客户端首先要发送同步信号给要访问的机器，此时状态为 SYN_SENT，如果连接成功了就变为 ESTABLISHED，正常情况下 SYN_SENT 状态非常短暂。例如要访问网站 http://www.baidu.com, 如果是正常连接的话，用 TCPView 观察 IEXPLORE.EXE（IE）建立的连接会发现很快从 SYN_SENT 变为 ESTABLISHED，表示连接成功。SYN_SENT 状态快的也许看不到。  
     如果发现有很多 SYN_SENT 出现，那一般有这么几种情况，一是你要访问的网站不存在或线路不好，二是用扫描软件扫描一个网段的机器，也会出出现很多 SYN_SENT，另外就是可能中了病毒了，例如中了 "冲击波"，病毒发作时会扫描其它机器，这样会有很多 SYN_SENT 出现。

  
 **3)、SYN-RECEIVED：服务器端握手状态 SYN_RCVD**

       再收到和发送一个连接请求后等待对方对连接请求的确认

     当服务器收到客户端发送的同步信号时，将标志位 ACK 和 SYN 置 1 发送给客户端，此时服务器端处于 SYN_RCVD 状态，如果连接成功了就变为 ESTABLISHED，正常情况下 SYN_RCVD 状态非常短暂。  
   如果发现有很多 SYN_RCVD 状态，那你的机器有可能被 SYN Flood 的 DoS(拒绝服务攻击) 攻击了。

     SYN Flood 的攻击原理是：

      在进行三次握手时，攻击软件向被攻击的服务器发送 SYN 连接请求（握手的第一步），但是这个地址是伪造的，如攻击软件随机伪造了 51.133.163.104、65.158.99.152 等等地址。服务器在收到连接请求时将标志位 ACK 和 SYN 置 1 发送给客户端（握手的第二步），但是这些客户端的 IP 地址都是伪造的，服务器根本找不到客户机，也就是说握手的第三步不可能完成。

       这种情况下服务器端一般会重试（再次发送 SYN+ACK 给客户端）并等待一段时间后丢弃这个未完成的连接，这段时间的长度我们称为 SYN Timeout，一般来说这个时间是分钟的数量级（大约为 30 秒 - 2 分钟）；一个用户出现异常导致服务器的一个线程等待 1 分钟并不是什么很大的问题，但如果有一个恶意的攻击者大量模拟这种情况，服务器端将为了维护一个非常大的半连接列表而消耗非常多的资源 ---- 数以万计的半连接，即使是简单的保存并遍历也会消耗非常多的 CPU 时间和内存，何况还要不断对这个列表中的 IP 进行 SYN+ACK 的重试。此时从正常客户的角度看来，服务器失去响应，这种情况我们称做：**服务器端受到了 SYN Flood 攻击（SYN 洪水攻击**）

  
 **4)****、****ESTABLISHED：代表一个打开的连接。**

    ESTABLISHED 状态是表示两台机器正在传输数据，观察这个状态最主要的就是看哪个程序正在处于 ESTABLISHED 状态。

    服务器出现很多 ESTABLISHED 状态： netstat -nat |grep 9502 或者使用 lsof  -i:9502 可以检测到。

    当客户端未主动 close 的时候就断开连接：即客户端发送的 FIN 丢失或未发送:  
    这时候若客户端断开的时候发送了 FIN 包，则服务端将会处于 CLOSE_WAIT 状态；  
    这时候若客户端断开的时候未发送 FIN 包，则服务端处还是显示 ESTABLISHED 状态；  
    结果客户端重新连接服务器。  
    而新连接上来的客户端（也就是刚才断掉的重新连上来了）在服务端肯定是 ESTABLISHED; 如果客户端重复的上演这种情况，那么服务端将会出现大量的假的 ESTABLISHED 连接和 CLOSE_WAIT 连接。  
    最终结果就是新的其他客户端无法连接上来，但是利用 netstat 还是能看到一条连接已经建立，并显示 ESTABLISHED，但始终无法进入程序代码。

三、 TCP 关闭四次握手的相关状态
==================

　  由于 TCP 连接是全双工的，因此每个方向都必须单独进行关闭。这原则是当一方完成它的数据发送任务后就能发送一个 FIN 来终止这个方向的连接。收到一个 FIN 只意味着这一方向上没有数据流动，一个 TCP 连接在收到一个 FIN 后仍能发送数据。首先进行关闭的一方将执行主动关闭，而另一方执行被动关闭。

**1、四次握手关闭连接的过程**
-----------------

           建立一个连接需要三次握手，而终止一个连接要经过四次握手，这是由 TCP 的半关闭 (half-close) 造成的，如图：

      ![](https://img-blog.csdnimg.cn/20200712013831419.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hndWlzdQ==,size_16,color_FFFFFF,t_70)

**调用过程和对应函数接口：**

*   **1)  客户端发送一个 FIN: 用来关闭客户 A 到服务器 B 的数据传送,** 当 client 想要关闭它与 server 之间的连接。client（某个应用进程）首先调用 close 主动关闭连接，这时 TCP 发送一个 FIN M；client 端处于 **FIN_WAIT1** 状态。
    
*   **2) 服务器确认客户端 FIN:** 当 server 端接收到 FIN M 之后，执行被动关闭。对这个 FIN 进行确认，返回给 client ACK。确认序号为收到的序号加 1（报文段 5）。和 SYN 一样，一个 FIN 将占用一个序号。当 server 端返回给 client ACK 后，client 处于 **FIN_WAIT2** 状态，server 处于 **CLOSE_WAIT** 状态。它的接收也作为文件结束符传递给应用进程，因为 FIN 的接收     意味着应用进程在相应的连接上再也接收不到额外数据；
    
*   **3)  服务器 B 发送一个 FIN 关闭与客户端 A 的连接:** 一段时间之后，当 server 端检测到 client 端的关闭操作 (read 返回为 0)。接收到文件结束符的 server 端调用 close 关闭它的 socket。这导致 server 端的 TCP 也发送一个 FIN N；此时 server 的状态为 **LAST_ACK。**
    
*   **4) 客户端 A 发回 ACK 报文确认:** 当 client 收到来自 server 的 FIN 后 。 client 端的套接字处于 **TIME_WAIT** 状态，它会向 server 端再发送一个 ack 确认，此时 server 端收到 ack 确认后，此套接字处于 CLOSED 状态。
    

**这样每个方向上都有一个 FIN 和 ACK。**

**2、四次握手关闭连接的具体状态**
-------------------

 **1）FIN-WAIT-1：**

 等待远程 TCP 连接中断请求，或先前的连接中断请求的确认

       主动关闭 (active close) 端应用程序调用 close，于是其 TCP 发出 FIN 请求主动关闭连接，之后进入 FIN_WAIT1 状态./* The socket is closed, and the connection is shutting down. 等待远程 TCP 的连接中断请求，或先前的连接中断请求的确认 */

      如果服务器出现 shutdown 再重启，使用 netstat -nat 查看，就会看到很多 **FIN-WAIT-1** 的状态。就是因为服务器当前有很多客户端连接，直接关闭服务器后，无法接收到客户端的 ACK。

  
 **2）FIN-WAIT-2：从远程 TCP 等待连接中断请求**

       主动关闭端接到 ACK 后，就进入了 FIN-WAIT-2 ./* Connection is closed, and the socket is waiting for a shutdown from the remote end. 从远程 TCP 等待连接中断请求 */

        这就是著名的半关闭的状态了，这是在关闭连接时，客户端和服务器两次握手之后的状态。在这个状态下，应用程序还有接受数据的能力，但是已经无法发送数据，但是也有一种可能是，客户端一直处于 FIN_WAIT_2 状态，而服务器则一直处于 WAIT_CLOSE 状态，而直到应用层来决定关闭这个状态。

  
 **3）CLOSE-WAIT：等待从本地用户发来的连接中断请求**

         被动关闭 (passive close) 端 TCP 接到 FIN 后，就发出 ACK 以回应 FIN 请求(它的接收也作为文件结束符传递给上层应用程序), 并进入 CLOSE_WAIT. /* The remote end has shut down, waiting for the socket to close. 等待从本地用户发来的连接中断请求 */

        
 **4）CLOSING：等待远程 TCP 对连接中断的确认**

比较少见./* Both sockets are shut down but we still don't have all our data sent. 等待远程 TCP 对连接中断的确认 */

实际情况中应该是很少见，属于一种比较罕见的例外状态。正常情况下，当一方发送 FIN 报文后，按理来说是应该先收到（或同时收到）对方的 ACK 报文，再收到对方的 FIN 报文。但是 CLOSING 状态表示一方发送 FIN 报文后，并没有收到对方的 ACK 报文，反而却也收到了对方的 FIN 报文。什么情况下会出现此种情况呢？  
有两种情况可能导致这种状态：  
其一，如果双方几乎在同时关闭连接，那么就可能出现双方同时发送 FIN 包的情况；  
其二，如果 ACK 包丢失而对方的 FIN 包很快发出，也会出现 FIN 先于 ACK 到达。

 **4）LAST-ACK：等待原来的发向远程 TCP 的连接中断请求的确认**

被动关闭端一段时间后，接收到文件结束符的应用程序将调用 CLOSE 关闭连接。这导致它的 TCP 也发送一个 FIN, 等待对方的 ACK. 就进入了 LAST-ACK . /* The remote end has shut down, and the socket is closed. Waiting for acknowledgement. 等待原来发向远程 TCP 的连接中断请求的确认 */

使用并发压力测试的时候，突然断开压力测试客户端，服务器会看到很多 **LAST-ACK。**

  
 **5）TIME-WAIT：等待足够的时间以确保远程 TCP 接收到连接中断请求的确认**

          在主动关闭端接收到 FIN 后，TCP 就发送 ACK 包，并进入 TIME-WAIT 状态。/* The socket is waiting after close to handle packets still in the network. 等待足够的时间以确保远程 TCP 接收到连接中断请求的确认 */

         TIME_WAIT 等待状态，这个状态又叫做 2MSL 状态，说的是在 TIME_WAIT2 发送了最后一个 ACK 数据报以后，要进入 TIME_WAIT 状态，这个状态是防止最后一次握手的数据报没有传送到对方那里而准备的（注意这不是四次握手，这是第四次握手的保险状态）。这个状态在很大程度上保证了双方都可以正常结束。

        由于插口的 2MSL 状态（插口是 IP 和端口对的意思，socket），使得应用程序在 2MSL 时间内是无法再次使用同一个插口的，对于客户程序还好一些，但是对于服务程序，例如 httpd，它总是要使用同一个端口来进行服务，而在 2MSL 时间内，启动 httpd 就会出现错误（插口被使用）。为了避免这个错误，服务器给出了一个平静时间的概念，这是说在 2MSL 时间内，虽然可以重新启动服务器，但是这个服务器还是要平静的等待 2MSL 时间的过去才能进行下一次连接。

          详情请看：[**TIME_WAIT 引起 Cannot assign requested address 报错**](http://blog.csdn.net/hguisu/article/details/10241519)  
 

 **6）CLOSED：没有任何连接状态**

被动关闭端在接受到 ACK 包后，就进入了 closed 的状态。连接结束./* The socket is not being used. 没有任何连接状态 */

四、TCP 状态迁移路线图
=============

client/server 两条路线讲述 TCP 状态迁移路线图：

![](https://img-blog.csdn.net/20140814205533640?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGd1aXN1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

       这是一个看起来比较复杂的状态迁移图，因为它包含了两个部分 --- 服务器的状态迁移和客户端的状态迁移，如果从某一个角度出发来看这个图，就会清晰许多，这里面的服务器和客户端都不是绝对的，发送数据的就是客户端，接受数据的就是服务器。

**客户端应用程序的状态迁移图**

        客户端的状态可以用如下的流程来表示：

        CLOSED->SYN_SENT->ESTABLISHED->FIN_WAIT_1->FIN_WAIT_2->TIME_WAIT->CLOSED

         以上流程是在程序正常的情况下应该有的流程，从书中的图中可以看到，在建立连接时，当客户端收到 SYN 报文的 ACK 以后，客户端就打开了数据交互地连接。而结束连接则通常是客户端主动结束的，客户端结束应用程序以后，需要经历 FIN_WAIT_1，FIN_WAIT_2 等状态，这些状态的迁移就是前面提到的结束连接的四次握手。

**服务器的状态迁移图**

        服务器的状态可以用如下的流程来表示：

         CLOSED->LISTEN->SYN 收到 ->ESTABLISHED->CLOSE_WAIT->LAST_ACK->CLOSED

        在建立连接的时候，服务器端是在第三次握手之后才进入数据交互状态，而关闭连接则是在关闭连接的第二次握手以后（注意不是第四次）。而关闭以后还要等待客户端给出最后的 ACK 包才能进入初始的状态。  
 

**其他状态迁移**

还有一些其他的状态迁移，这些状态迁移针对服务器和客户端两方面的总结如下

LISTEN->SYN_SENT，对于这个解释就很简单了，服务器有时候也要打开连接的嘛。

SYN_SENT->SYN 收到，服务器和客户端在 SYN_SENT 状态下如果收到 SYN 数据报，则都需要发送 SYN 的 ACK 数据报并把自己的状态调整到 SYN 收到状态，准备进入 ESTABLISHED

SYN_SENT->CLOSED，在发送超时的情况下，会返回到 CLOSED 状态。

SYN_收到 ->LISTEN，如果受到 RST 包，会返回到 LISTEN 状态。

SYN_收到 ->FIN_WAIT_1，这个迁移是说，可以不用到 ESTABLISHED 状态，而可以直接跳转到 FIN_WAIT_1 状态并等待关闭。

![](https://img-blog.csdn.net/20140813230501105?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGd1aXN1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

怎样牢牢地将这张图刻在脑中呢？那么你就一定要对这张图的每一个状态，及转换的过程有深刻的认识，不能只停留在一知半解之中。下面对这张图的 11 种状态详细解析一下，以便加强记忆！不过在这之前，先回顾一下 TCP 建立连接的三次握手过程，以及关闭连接的四次握手过程。

五、具体问题
======

**1．为什么建立连接协议是三次握手，而关闭连接却是四次握手呢？**

>         这是因为服务端的 LISTEN 状态下的 SOCKET 当收到 SYN 报文的建连请求后，它可以把 ACK 和 SYN（ACK 起应答作用，而 SYN 起同步作用）放在一个报文里来发送。但关闭连接时，当收到对方的 FIN 报文通知时，它仅仅表示对方没有数据发送给你了；但未必你所有的数据都全部发送给对方了，所以你可以未必会马上会关闭 SOCKET, 也即你可能还需要发送一些数据给对方之后，再发送 FIN 报文给对方来表示你同意现在可以关闭连接了，所以它这里的 ACK 报文和 FIN 报文多数情况下都是分开发送的。
> 
> ![](https://img-blog.csdnimg.cn/2020021722420123.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hndWlzdQ==,size_16,color_FFFFFF,t_70)

[2、什么是 2MSL](http://blog.csdn.net/xiaofei0859/article/details/6044694)

> MSL 是 Maximum Segment Lifetime 英文的缩写，中文可以译为 “报文最大生存时间”，他是任何报文在网络上存在的最长时间，超过这个时间报文将被丢弃。因为 tcp 报文（segment）是 ip 数据报（datagram）的数据部分，具体称谓请参见《数据在网络各层中的称呼》一文，而 ip 头中有一个 TTL 域，TTL 是 time to live 的缩写，中文可以译为 “生存时间”，这个生存时间是由源主机设置初始值但不是存的具体时间，而是存储了一个 ip 数据报可以经过的最大路由数，每经过一个处理他的路由器此值就减 1，当此值为 0 则数据报将被丢弃，同时发送 ICMP 报文通知源主机。RFC 793 中规定 MSL 为 2 分钟，实际应用中常用的是 30 秒，1 分钟和 2 分钟等。
> 
>     2MSL 即两倍的 MSL，TCP 的 TIME_WAIT 状态也称为 2MSL 等待状态，当 TCP 的一端发起主动关闭，在发出最后一个 ACK 包后，即第 3 次握手完成后发送了第四次握手的 ACK 包后就进入了 TIME_WAIT 状态，必须在此状态上停留两倍的 MSL 时间，等待 2MSL 时间主要目的是怕最后一个 ACK 包对方没收到，那么对方在超时后将重发第三次握手的 FIN 包，主动关闭端接到重发的 FIN 包后可以再发一个 ACK 应答包。在 TIME_WAIT 状态时两端的端口不能使用，要等到 2MSL 时间结束才可继续使用。当连接处于 2MSL 等待阶段时任何迟到的报文段都将被丢弃。不过在实际应用中可以通过设置 SO_REUSEADDR 选项达到不必等待 2MSL 时间结束再使用此端口。
> 
> 这是因为虽然双方都同意关闭连接了，而且握手的 4 个报文也都协调和发送完毕，按理可以直接回到 CLOSED 状态（就好比从 SYN_SEND 状态到 ESTABLISH 状态那样）：
> 
> 一方面是可靠的实现 TCP 全双工连接的终止，也就是当最后的 ACK 丢失后，被动关闭端会重发 FIN，因此主动关闭端需要维持状态信息，以允许它重新发送最终的 ACK。
> 
> 另一方面，但是因为我们必须要假想网络是不可靠的，你无法保证你最后发送的 ACK 报文会一定被对方收到，因此对方处于 LAST_ACK 状态下的 SOCKET 可能会因为超时未收到 ACK 报文，而重发 FIN 报文，所以这个 TIME_WAIT 状态的作用就是用来重发可能丢失的 ACK 报文。
> 
> TCP 在 2MSL 等待期间，定义这个连接 (4 元组) 不能再使用，任何迟到的报文都会丢弃。设想如果没有 2MSL 的限制，恰好新到的连接正好满足原先的 4 元组，这时候连接就可能接收到网络上的延迟报文就可能干扰最新建立的连接。

**3．为什么 TIME_WAIT 状态还需要等 2MSL 后才能返回到 CLOSED 状态？**

> **第一，**保证可靠的实现 TCP 全双工链接的终止：为了保证主动端 A 发送的最后一个 ACK 报文能够到达被动段 B，确保被动端能进入 CLOSED 状态。
> 
> 对照上图，在四次挥手协议中，当 B 向 A 发送 Fin+Ack 后，A 就需要向 B 发送 ACK+Seq 报文，A 这时候就处于 TIME_WAIT 状态，但是这个报文 ACK 有可能会发送失败而丢失，因而使处在 LAST-ACK 状态的 B 收不到对已发送的 FIN+ACK 报文段的确认。B 会超时重传这个 FIN+ACK 报文段，而 A 就能在 2MSL 时间内收到这个重传的 FIN+ACK 报文段。如果 A 在 TIME-WAIT 状态不等待一段时间，而是在发送完 ACK 报文段后就立即释放连接，就无法收到 B 重传的 FIN+ACK 报文段，因而也不会再发送一次确认报文段。这样 B 就无法按照正常的步骤进入 CLOSED 状态。
> 
>   
> **第二，**防止已失效的连接请求报文段出现在本连接中。**A 在发送完 ACK 报文段后，再经过 2MSL 时间**，就可以使本连接持续的时间所产生的所有报文段都从网络中消失。这样就可以使下一个新的连接中不会出现这种旧的连接请求的报文段。
> 
> 假设在 A 的 XXX1 端口和 B 的 80 端口之间有一个 TCP 连接。我们关闭这个连接，过一段时间后在相同的 IP 地址和端口建立另一个连接。后一个链接成为前一个的化身。因为它们的 IP 地址和端口号都相同。TCP 必须防止来自某一个连接的老的重复分组在连 接已经终止后再现，从而被误解成属于同一链接的某一个某一个新的化身。为做到这一点，TCP 将不给处于 TIME_WAIT 状态的链接发起新的化身。既然 TIME_WAIT 状态的持续时间是 MSL 的 2 倍，这就足以让某个方向上的分组最多存活 msl 秒即被丢弃，另一个方向上的应答最多存活 msl 秒也被丢弃。 通过实施这个规则，我们就能保证每成功建立一个 TCP 连接时。来自该链接先前化身的重复分组都已经在网络中消逝了。

4、解决 linux 发现系统存在大量 TIME_WAIT 状态

> 如果 linux 发现系统存在大量 TIME_WAIT 状态的连接，可以通过调整内核参数解决：vi /etc/sysctl.conf 加入以下内容：  
> net.ipv4.tcp_max_tw_buckets=5000 #TIME-WAIT Socket 最大数量  
> #注意：不建议开启該设置，NAT 模式下可能引起连接 RST  
> net.ipv4.tcp_tw_reuse = 1 #表示开启重用。允许将 TIME-WAIT sockets 重新用于新的 TCP 连接，默认为 0，表示关闭；  
> net.ipv4.tcp_tw_recycle = 1 #表示开启 TCP 连接中 TIME-WAIT sockets 的快速回收，默认为 0，表示关闭。  
>  然后执行 /sbin/sysctl -p 让参数生效。

六. TCP 的 FLAGS 说明
=================

在 TCP 层，有个 FLAGS 字段，这个字段有以下几个标识：SYN, FIN, ACK, PSH, RST, URG.  
其中，对于我们日常的分析有用的就是前面的五个字段。  
**一、字段含义：**  
**1、SYN 表示建立连接：**

步序列编号 (Synchronize Sequence Numbers) 栏有效。该标志仅在三次握手建立 TCP 连接时有效。它提示 TCP 连接的服务端检查序列编号，该序列编号为 TCP 连接初始端 (一般是客户端) 的初始序列编号。在这里，可以把 TCP 序列编号看作是一个范围从 0 到 4，294，967，295 的 32 位计数器。通过 TCP 连接交换的数据中每一个字节都经过序列编号。在 TCP 报头中的序列编号栏包括了 TCP 分段中第一个字节的序列编号。

**2、FIN 表示关闭连接：**  
**3、ACK 表示响应：**

确认编号 (Acknowledgement Number) 栏有效。大多数情况下该标志位是置位的。TCP 报头内的确认编号栏内包含的确认编号 (w+1，Figure-1) 为下一个预期的序列编号，同时提示远端系统已经成功接收所有数据。

**4、PSH 表示有 DATA 数据传输：**

   
**5、RST 表示连接重置：** 复位标志有效。用于复位相应的 TCP 连接。

**一、字段组合含义：**

**![](https://img-blog.csdn.net/20180622143944315?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hndWlzdQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)**

其中，ACK 是可能与 SYN，FIN 等同时使用的，比如 SYN 和 ACK 可能同时为 1，它表示的就是建立连接之后的响应，  
如果只是单个的一个 SYN，它表示的只是建立连接。  
TCP 的几次握手就是通过这样的 ACK 表现出来的。  
但 SYN 与 FIN 是不会同时为 1 的，因为前者表示的是建立连接，而后者表示的是断开连接。  
RST 一般是在 FIN 之后才会出现为 1 的情况，表示的是连接重置。  
一般地，当出现 FIN 包或 RST 包时，我们便认为客户端与[服务器](http://www.yunsec.net/a/zhuanti/enterprise/server/)端断开了连接；

![](https://img-blog.csdn.net/20180403133313746?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hndWlzdQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

  RST 与 ACK 标志位都置一了，并且具有 ACK number，非常明显，这个报文在释放 TCP 连接的同时，完成了对前面已接收报文的确认。

而当出现 SYN 和 SYN＋ACK 包时，我们认为客户端与服务器建立了一个连接。  
PSH 为 1 的情况，一般只出现在 DATA 内容不为 0 的包中，也就是说 PSH 为 1 表示的是有真正的 TCP 数据包内容被传递。  
TCP 的连接建立和连接关闭，都是通过请求－响应的模式完成的。

七. TCP 通信中服务器处理客户端意外断开
======================

     引用地址：http://blog.csdn.net/kkkkkxiaofei/article/details/12966407

      如果 TCP 连接被对方正常关闭，也就是说，对方是正确地调用了 closesocket(s) 或者 shutdown(s) 的话，那么上面的 Recv 或 Send 调用就能马上返回，并且报错。这是由于 close socket(s) 或者 shutdown(s) 有个正常的关闭过程，会告诉对方 “TCP 连接已经关闭，你不需要再发送或者接受消息了”。

但是，如果意外断开，客户端（3g 的移动设备）并没有正常关闭 socket。双方并未按照协议上的四次挥手去断开连接。

那么这时候正在执行 Recv 或 Send 操作的一方就会因为没有任何连接中断的通知而一直等待下去，也就是会被长时间卡住。

       像这种如果一方已经关闭或异常终止连接，而另一方却不知道，我们将这样的 TCP 连接称为半打开的。

       解决意外中断办法都是利用保活机制。而保活机制分又可以让底层实现也可自己实现。

     1、自己编写心跳包程序

     简单的说也就是在自己的程序中加入一条线程，定时向对端发送数据包，查看是否有 ACK，如果有则连接正常，没有的话则连接断开

     2、启动 TCP 编程里的 keepAlive 机制

一、双方拟定心跳（自实现）

     一般由客户端发送心跳包，服务端并不回应心跳，只是定时轮询判断一下与上次的时间间隔是否超时（超时时间自己设定）。服务器并不主动发送是不想增添服务器的通信量，减少压力。

但这会出现三种情况：

情况 1.

       客户端由于某种网络延迟等原因很久后才发送心跳（它并没有断），这时服务器若利用自身设定的超时判断其已经断开，而后去关闭 socket。若客户端有重连机制，则客户端会重新连接。若不确定这种方式是否关闭了原本正常的客户端，则在 ShutDown 的时候一定要选择 send, 表示关闭发送通道，服务器还可以接收一下，万一客户端正在发送比较重要的数据呢，是不？

情况 2.

       客户端很久没传心跳，确实是自身断掉了。在其重启之前，服务端已经判断出其超时，并主动 close，则四次挥手成功交互。

情况 3.

      客户端很久没传心跳，确实是自身断掉了。在其重启之前，服务端的轮询还未判断出其超时，在未主动 close 的时候该客户端已经重新连接。

       这时候若客户端断开的时候发送了 FIN 包，则服务端将会处于 CLOSE_WAIT 状态；

       这时候若客户端断开的时候未发送 FIN 包，则服务端处还是显示 ESTABLISHED 状态；

       而新连接上来的客户端（也就是刚才断掉的重新连上来了）在服务端肯定是 ESTABLISHED; 这时候就有个问题，若利用轮询还未检测出上条旧连接已经超时（这很正常，timer 总有个间隔吧），而在这时，客户端又重复的上演情况 3，那么服务端将会出现大量的假的 ESTABLISHED 连接和 CLOSE_WAIT 连接。

        最终结果就是新的其他客户端无法连接上来，但是利用 netstat 还是能看到一条连接已经建立，并显示 ESTABLISHED，但始终无法进入程序代码。个人最初感觉导致这种情况是因为假的 ESTABLISHED 连接和 CLOSE_WAIT 连接会占用较大的系统资源，程序无法再次创建连接（因为每次我发现这个问题的时候我只连了 10 个左右客户端却已经有 40 多条无效连接）。而最近几天测试却发现有一次程序内只连接了 2，3 个设备，但是有 8 条左右的虚连接，此时已经连接不了新客户端了。这时候我就觉得我想错了，不可能这几条连接就占用了大量连接把，如果说几十条还有可能。但是能肯定的是，这个问题的产生绝对是设备在不停的重启，而服务器这边又是简单的轮询，并不能及时处理，暂时还未能解决。

二、利用 KeepAlive

          其实 keepalive 的原理就是 TCP 内嵌的一个心跳包,

         以服务器端为例，如果当前 server 端检测到超过一定时间（默认是 7,200,000 milliseconds，也就是 2 个小时）没有数据传输，那么会向 client 端发送一个 keep-alive packet（该 keep-alive packet 就是 ACK 和当前 TCP 序列号减一的组合），此时 client 端应该为以下三种情况之一：

        1. client 端仍然存在，网络连接状况良好。此时 client 端会返回一个 ACK。server 端接收到 ACK 后重置计时器（复位存活定时器），在 2 小时后再发送探测。如果 2 小时内连接上有数据传输，那么在该时间基础上向后推延 2 个小时。

        2. 客户端异常关闭，或是网络断开。在这两种情况下，client 端都不会响应。服务器没有收到对其发出探测的响应，并且在一定时间（系统默认为 1000 ms）后重复发送 keep-alive packet，并且重复发送一定次数（2000 XP 2003 系统默认为 5 次, Vista 后的系统默认为 10 次）。

         3. 客户端曾经崩溃，但已经重启。这种情况下，服务器将会收到对其存活探测的响应，但该响应是一个复位，从而引起服务器对连接的终止。

       对于应用程序来说，2 小时的空闲时间太长。因此，我们需要手工开启 Keepalive 功能并设置合理的 Keepalive 参数。

全局设置可更改 /etc/sysctl.conf, 加上:  
net.ipv4.tcp_keepalive_intvl = 20  
net.ipv4.tcp_keepalive_probes = 3  
net.ipv4.tcp_keepalive_time = 60

在程序中设置如下:

```
 #include <sys/socket.h>
  #include <netinet/in.h>
  #include <arpa/inet.h>
  #include <sys/types.h>
  #include <netinet/tcp.h>
 
  int keepAlive = 1; // 开启keepalive属性
  int keepIdle = 60; // 如该连接在60秒内没有任何数据往来,则进行探测 
  int keepInterval = 5; // 探测时发包的时间间隔为5 秒
  int keepCount = 3; // 探测尝试的次数.如果第1次探测包就收到响应了,则后2次的不再发.
 
  setsockopt(rs, SOL_SOCKET, SO_KEEPALIVE, (void *)&keepAlive, sizeof(keepAlive));
  setsockopt(rs, SOL_TCP, TCP_KEEPIDLE, (void*)&keepIdle, sizeof(keepIdle));
  setsockopt(rs, SOL_TCP, TCP_KEEPINTVL, (void *)&keepInterval, sizeof(keepInterval));
  setsockopt(rs, SOL_TCP, TCP_KEEPCNT, (void *)&keepCount, sizeof(keepCount)); 
```

  
在程序中表现为, 当 tcp 检测到对端 socket 不再可用时 (不能发出探测包, 或探测包没有收到 ACK 的响应包),select 会返回 socket 可读, 并且在 recv 时返回 - 1, 同时置上 errno 为 ETIMEDOUT.

八. Linux 错误信息 (errno) 列表
========================

经常出现的错误：

22：参数错误，比如 ip 地址不合法，没有目标端口等

101：网络不可达，比如不能 ping 通

111：链接被拒绝，比如目标关闭链接等

115：当链接设置为非阻塞时，目标没有及时应答，返回此错误，socket 可以继续使用。比如 socket 连接

附录：Linux 的错误码表（errno table)

_ 124 EMEDIUMTYPE_ Wrong medium type  
_ 123 ENOMEDIUM__ No medium found  
_ 122 EDQUOT___  Disk quota exceeded  
_ 121 EREMOTEIO__ Remote I/O error  
_ 120 EISNAM___  Is a named type file  
_ 119 ENAVAIL___ No XENIX semaphores available  
_ 118 ENOTNAM___ Not a XENIX named type file  
_ 117 EUCLEAN___ Structure needs cleaning  
_ 116 ESTALE___  Stale NFS file handle  
_ 115 EINPROGRESS  +Operation now in progress

操作正在进行中。一个阻塞的操作正在执行。

  
_ 114 EALREADY__  Operation already in progress  
_ 113 EHOSTUNREACH  No route to host  
_ 112 EHOSTDOWN__ Host is down  
_ 111 ECONNREFUSED  Connection refused

1、拒绝连接。一般发生在连接建立时。

拔服务器端网线测试，客户端设置 keep alive 时，recv 较快返回 0， 先收到 ECONNREFUSED (Connection refused) 错误码，其后都是 ETIMEOUT。

2、an error returned from connect(), so it can only occur in a client (if a client is defined as the party that initiates the connection

  
_ 110 ETIMEDOUT_  +Connection timed out  
_ 109 ETOOMANYREFS  Too many references: cannot splice  
_ 108 ESHUTDOWN__ Cannot send after transport endpoint shutdown  
_ 107 ENOTCONN__  Transport endpoint is not connected

在一个没有建立连接的 socket 上，进行 read，write 操作会返回这个错误。出错的原因是 socket 没有标识地址。Setsoc 也可能会出错。

还有一种情况就是收到对方发送过来的 RST 包, 系统已经确认连接被断开了。

  
_ 106 EISCONN___ Transport endpoint is already connected

一般是 socket 客户端已经连接了，但是调用 connect，会引起这个错误。

  
_ 105 ENOBUFS___ No buffer space available  
_ 104 ECONNRESET_  Connection reset by peer

连接被远程主机关闭。有以下几种原因：远程主机停止服务，重新启动; 当在执行某些操作时遇到失败，因为设置了 “keep alive” 选项，连接被关闭，一般与 ENETRESET 一起出现。

1、在客户端服务器程序中，客户端异常退出，并没有回收关闭相关的资源，服务器端会先收到 ECONNRESET 错误，然后收到 EPIPE 错误。

2、连接被远程主机关闭。有以下几种原因：远程主机停止服务，重新启动; 当在执行某些操作时遇到失败，因为设置了 “keep alive” 选项，连接被关闭，一般与 ENETRESET 一起出现。

3、远程端执行了一个 “hard” 或者 “abortive” 的关闭。应用程序应该关闭 socket，因为它不再可用。当执行在一个 UDP socket 上时，这个错误表明前一个 send 操作返回一个 ICMP“port unreachable”信息。

4、如果 client 关闭连接, server 端的 select 并不出错 (不返回 - 1, 使用 select 对唯一一个 socket 进行 non- blocking 检测), 但是写该 socket 就会出错, 用的是 send. 错误号: ECONNRESET. 读 (recv)socket 并没有返回错误。

5、该错误被描述为 “connection reset by peer”，即 “对方复位连接”，这种情况一般发生在服务进程较客户进程提前终止。

  **  主动关闭调用过程如下：**

**![](https://img-blog.csdn.net/20170710181743823?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaGd1aXN1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)**

 **服务器端主动关闭：**

   1）当服务器的服务因为某种原因，进程提前终止时会向客户 TCP 发送 FIN 分节，服务器端处于 **FIN_WAIT1** 状态。

   2）客户 TCP 回应 ACK 后，服务 TCP 将转入 **FIN_WAIT2** 状态。

   3）此时如果客户进程没有处理该 FIN （如阻塞在其它调用上而没有关闭 Socket 时），则客户 TCP 将处于 **CLOSE_WAIT** 状态。

   4）当客户进程再次向 FIN_WAIT2 状态的服务 TCP 发送数据时，则服务 TCP 将立刻响应 RST。

  一般来说，这种情况还可以会引发另外的应用程序异常，客户进程在发送完数据后，往往会等待从网络 IO 接收数据，很典型的如 read 或 readline 调用，此时由于执行时序的原因，如果该调用发生在 RST 分节收到前执行的话，那么结果是客户进程会得到一个非预期的 EOF 错误。此时一般会输出 “server terminated prematurely”－“服务器过早终止” 错误。

  
_ 103 ECONNABORTED  Software caused connection abort

1、软件导致的连接取消。一个已经建立的连接被 host 方的软件取消，原因可能是数据传输超时或者是协议错误。

2、该错误被描述为 “software caused connection abort”，即“软件引起的连接中止”。原因在于当服务和客户进程在完成用于 TCP 连接的“三次握手” 后，客户 TCP 却发送了一个 RST （复位）分节，在服务进程看来，就在该连接已由 TCP 排队，等着服务进程调用 accept 的时候 RST 却到达了。POSIX 规定此时的 errno 值必须 ECONNABORTED。源自 Berkeley 的实现完全在内核中处理中止的连接，服务进程将永远不知道该中止的发生。服务器进程一般可以忽略该错误，直接再次调用 accept。

当 TCP 协议接收到 RST 数据段，表示连接出现了某种错误，函数 read 将以错误返回，错误类型为 ECONNERESET。并且以后所有在这个套接字上的读操作均返回错误。错误返回时返回值小于 0。

  
_ 102 ENETRESET__ Network dropped connection on reset

网络重置时丢失连接。

由于设置了 "keep-alive" 选项，探测到一个错误，连接被中断。在一个已经失败的连接上试图使用 setsockopt 操作，也会返回这个错误。

  
_ 101 ENETUNREACH_ Network is unreachable

网络不可达。Socket 试图操作一个不可达的网络。这意味着 local 的软件知道没有路由到达远程的 host。  
_ 100 ENETDOWN__  Network is down  
_  99 EADDRNOTAVAIL Cannot assign requested address  
_  98 EADDRINUSE_  Address already in use  
_  97 EAFNOSUPPORT  Address family not supported by protocol  
_  96 EPFNOSUPPORT  Protocol family not supported  
_  95 EOPNOTSUPP_  Operation not supported  
_  94 ESOCKTNOSUPPORT Socket type not supported

Socket 类型不支持。指定的 socket 类型在其 address family 中不支持。如可选选中选项 SOCK_RAW，但实现并不支持 SOCK_RAW sockets。

  
_  93 EPROTONOSUPPORT Protocol not supported

不支持的协议。系统中没有安装标识的协议，或者是没有实现。如函数需要 SOCK_DGRAM socket，但是标识了 stream protocol.。

  
_  92 ENOPROTOOPT_ Protocol not available

该错误不是一个 Socket 连接相关的错误。errno 给出该值可能由于，通过 getsockopt 系统调用来获得一个套接字的当前选项状态时，如果发现了系统不支持的选项参数就会引发该错误。  
_  91 EPROTOTYPE_  Protocol wrong type for socket

协议类型错误。标识了协议的 Socket 函数在不支持的 socket 上进行操作。如 ARPA Internet

UDP 协议不能被标识为 SOCK_STREAM socket 类型。

  
_  90 EMSGSIZE__ +Message too long

消息体太长。

发送到 socket 上的一个数据包大小比内部的消息缓冲区大，或者超过别的网络限制，或是用来接收数据包的缓冲区比数据包本身小。

  
_  89 EDESTADDRREQ  Destination address required

需要提供目的地址。

在一个 socket 上的操作需要提供地址。如往一个 ADDR_ANY 地址上进行 sendto 操作会返回这个错误。

  
_  88 ENOTSOCK__  Socket operation on non-socket

在非 socket 上执行 socket 操作。

  
_  87 EUSERS___  Too many users  
_  86 ESTRPIPE__  Streams pipe error  
_  85 ERESTART__  Interrupted system call should be restarted  
_  84 EILSEQ___  Invalid or incomplete multibyte or wide character  
_  83 ELIBEXEC__  Cannot exec a shared library directly  
_  82 ELIBMAX___ Attempting to link in too many shared libraries  
_  81 ELIBSCN___ .lib section in a.out corrupted  
_  80 ELIBBAD___ Accessing a corrupted shared library  
_  79 ELIBACC___ Can not access a needed shared library  
_  78 EREMCHG___ Remote address changed  
_  77 EBADFD___  File descriptor in bad state  
_  76 ENOTUNIQ__  Name not unique on network  
_  75 EOVERFLOW__ Value too large for defined data type  
_  74 EBADMSG__  +Bad message  
_  73 EDOTDOT___ RFS specific error  
_  72 EMULTIHOP__ Multihop attempted  
_  71 EPROTO___  Protocol error  
_  70 ECOMM____ Communication error on send  
_  69 ESRMNT___  Srmount error  
_  68 EADV____  Advertise error  
_  67 ENOLINK___ Link has been severed  
_  66 EREMOTE___ Object is remote  
_  65 ENOPKG___  Package not installed  
_  64 ENONET___  Machine is not on the network  
_  63 ENOSR____ Out of streams resources  
_  62 ETIME____ Timer expired  
_  61 ENODATA___ No data available  
_  60 ENOSTR___  Device not a stream  
_  59 EBFONT___  Bad font file format  
_  57 EBADSLT___ Invalid slot  
_  56 EBADRQC___ Invalid request code  
_  55 ENOANO___  No anode  
_  54 EXFULL___  Exchange full  
_  53 EBADR____ Invalid request descriptor  
_  52 EBADE____ Invalid exchange  
_  51 EL2HLT___  Level 2 halted  
_  50 ENOCSI___  No CSI structure available  
_  49 EUNATCH___ Protocol driver not attached  
_  48 ELNRNG___  Link number out of range  
_  47 EL3RST___  Level 3 reset  
_  46 EL3HLT___  Level 3 halted  
_  45 EL2NSYNC__  Level 2 not synchronized  
_  44 ECHRNG___  Channel number out of range  
_  43 EIDRM____ Identifier removed  
_  42 ENOMSG___  No message of desired type  
_  40 ELOOP____ Too many levels of symbolic links  
_  39 ENOTEMPTY_  +Directory not empty  
_  38 ENOSYS___ +Function not implemented  
_  37 ENOLCK___ +No locks available  
_  36 ENAMETOOLONG +File name too long  
_  35 EDEADLK__  +Resource deadlock avoided  
_  34 ERANGE___ +Numerical result out of range  
_  33 EDOM____ +Numerical argument out of domain  
_  32 EPIPE___  +Broken pipe

接收端关闭 (缓冲中没有多余的数据), 但是发送端还在 write:

1、Socket 关闭，但是 socket 号并没有置 - 1。继续在此 socket 上进行 send 和 recv，就会返回这种错误。这个错误会引发 SIGPIPE 信号，系统会将产生此 EPIPE 错误的进程杀死。所以，一般在网络程序中，首先屏蔽此消息，以免发生不及时设置 socket 进程被杀死的情况。

2、write(..) on a socket that has been closed at the other end will cause a SIGPIPE.

3、错误被描述为 “broken pipe”，即 “管道破裂”，这种情况一般发生在客户进程不理会（或未及时处理）Socket 错误，继续向服务 TCP 写入更多数据时，内核将向客户进程发送 SIGPIPE 信号，该信号默认会使进程终止（此时该前台进程未进行 core dump）。结合上边的 ECONNRESET 错误可知，向一个 FIN_WAIT2 状态的服务 TCP（已 ACK 响应 FIN 分节）写入数据不成问题，但是写一个已接收了 RST 的 Socket 则是一个错误。

  
_  31 EMLINK___ +Too many links  
_  30 EROFS___  +Read-only file system  
_  29 ESPIPE___ +Illegal seek  
_  28 ENOSPC___ +No space left on device  
_  27 EFBIG___  +File too large  
_  26 ETXTBSY___ Text file busy  
_  25 ENOTTY___ +Inappropriate ioctl for device  
_  24 EMFILE___ +Too many open files

打开了太多的 socket。对进程或者线程而言，每种实现方法都有一个最大的可用 socket 数目处理，或者是全局的，或者是局部的。

_  23 ENFILE___ +Too many open files in system  
_  22 EINVAL___ +Invalid argument

无效参数。提供的参数非法。有时也会与 socket 的当前状态相关，如一个 socket 并没有进入 listening 状态，此时调用 accept，就会产生 EINVAL 错误。

  
_  21 EISDIR___ +Is a directory  
_  20 ENOTDIR__  +Not a directory  
_  19 ENODEV___ +No such device  
_  18 EXDEV___  +Invalid cross-device link  
_  17 EEXIST___ +File exists  
_  16 EBUSY___  +Device or resource busy  
_  15 ENOTBLK___ Block device required  
_  14 EFAULT___ +Bad address 地址错误  
_  13 EACCES___ +Permission denied  
_  12 ENOMEM___ +Cannot allocate memory  
_  11 EAGAIN___ +Resource temporarily unavailable

在读数据的时候, 没有数据在底层缓冲的时候会遇到, 一般的处理是循环进行读操作, 异步模式还会等待读事件的发生再读

   1、Send 返回值小于要发送的数据数目，会返回 EAGAIN 和 EINTR。

   2、recv 返回值小于请求的长度时说明缓冲区已经没有可读数据，但再读不一定会触发 EAGAIN，有可能返回 0 表示 TCP 连接已被关闭。

   3、当 socket 是非阻塞时, 如返回此错误, 表示写缓冲队列已满, 可以做延时后再重试.

   4、在 Linux 进行非阻塞的 socket 接收数据时经常出现 Resource temporarily unavailable，errno 代码为 11(EAGAIN)，表明在非阻塞模式下调用了阻塞操作，在该操作没有完成就返回这个错误，这个错误不会破坏 socket 的同步，不用管它，下次循环接着 recv 就可以。对非阻塞 socket 而言，EAGAIN 不是一种错误。

  
_  10 ECHILD___ +No child processes  
__ 9 EBADF___  +Bad file descriptor  
__ 8 ENOEXEC__  +Exec format error  
__ 7 E2BIG___  +Argument list too long  
__ 6 ENXIO___  +No such device or address  
__ 5 EIO____  +Input/output error  
__ 4 EINTR___  +Interrupted system call

    阻塞的操作被取消阻塞的调用打断。如设置了发送接收超时，就会遇到这种错误。

    只能针对阻塞模式的 socket。读，写阻塞的 socket 时，-1 返回，错误号为 INTR。另外，如果出现 EINTR 即 errno 为 4，错误描述 Interrupted system call，操作也应该继续。如果 recv 的返回值为 0，那表明连接已经断开，接收操作也应该结束。

  
__ 3 ESRCH___  +No such process  
__ 2 ENOENT___ +No such file or directory  
__ 1 EPERM___  +Operation not permitted