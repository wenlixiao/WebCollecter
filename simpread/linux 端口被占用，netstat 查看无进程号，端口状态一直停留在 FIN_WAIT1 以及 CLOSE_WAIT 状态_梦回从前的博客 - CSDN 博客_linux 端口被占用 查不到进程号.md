> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/microGP/article/details/86588973)

**环境信息：**
---------

CentOS 6.5

**现象：**
-------

同事启动程序发现端口被占用，netstat 查看之后发现如下现象：

![](https://img-blog.csdnimg.cn/20190122143537498.jpg)

发现端口处于 FIN_WAIT1 状态以及 CLOSE_WAIT 状态，无法释放

问题分析：
-----

FIN_WAIT1 以及 CLOSE_WAIT 状态的原理以及产生的原因大家自行 baidu，下面就说下怎么解决上述问题，释放端口

### FIN_WAIT1：

**1、sysctl -a |grep tcp_max_orph（记下 net.ipv4.tcp_max_orphans  的值  第三步需要赋给 orig_orphans）** 

**2、sysctl -w net.ipv4.tcp_max_orphans=0   然后等待 FIN_WAIT1 的消失，可以用 netstat -np|grep 9080  反复查看，直到没有任何条目**

**3、sysctl -w net.ipv4.tcp_max_orphans=$orig_orphans**

执行完毕查看：

![](https://img-blog.csdnimg.cn/20190122144136215.jpg)

### CLOSE_WAIT：

当客户端因为某种原因先于服务端发出了 FIN 信号，就会导致服务端被动关闭，若服务端不主动关闭 socket 发 FIN 给 Client，此时服务端 Socket 会处于 CLOSE_WAIT 状态（而不是 LAST_ACK 状态）。通常来说，一个 CLOSE_WAIT 会维持至少 2 个小时的时间（系统默认超时时间的是 7200 秒，也就是 2 小时）。如果服务端程序因某个原因导致系统造成一堆 CLOSE_WAIT 消耗资源，那么通常是等不到释放那一刻，系统就已崩溃。因此，解决这个问题的方法还可以通过修改 TCP/IP 的参数来缩短这个时间，于是修改 **tcp_keepalive_* 系列参数：**   
**tcp_keepalive_time：**   
/proc/sys/net/ipv4/tcp_keepalive_time   
INTEGER，默认值是 7200(2 小时)   
当 keepalive 打开的情况下，TCP 发送 keepalive 消息的频率。建议修改值为 1800 秒。   
  
**tcp_keepalive_probes：INTEGER**   
/proc/sys/net/ipv4/tcp_keepalive_probes   
INTEGER，默认值是 9   
TCP 发送 keepalive 探测以确定该连接已经断开的次数。(注意: 保持连接仅在 SO_KEEPALIVE 套接字选项被打开是才发送. 次数默认不需要修改, 当然根据情形也可以适当地缩短此值. 设置为 5 比较合适)   
  
**tcp_keepalive_intvl：INTEGER**   
/proc/sys/net/ipv4/tcp_keepalive_intvl   
INTEGER，默认值为 75   
当探测没有确认时，重新发送探测的频度。探测消息发送的频率（在认定连接失效之前，发送多少个 TCP 的 keepalive 探测包）。乘以 tcp_keepalive_probes 就得到对于从开始探测以来没有响应的连接杀除的时间。默认值为 75 秒，也就是没有活动的连接将在大约 11 分钟以后将被丢弃。(对于普通应用来说, 这个值有一些偏大, 可以根据需要改小. 特别是 web 类服务器需要改小该值, 15 是个比较合适的值) 

**修改方法：**

一、 命令修改：（暂时生效, 重新启动服务器后, 会还原成默认值）   
sysctl -w net.ipv4.tcp_keepalive_time=600     
sysctl -w net.ipv4.tcp_keepalive_probes=2   
sysctl -w net.ipv4.tcp_keepalive_intvl=2   
  
注意：Linux 的内核参数调整的是否合理要注意观察，看业务高峰时候效果如何。   
  
二、 若做如上修改后，可起作用；则做如下修改以便永久生效。   
vi /etc/sysctl.conf   
  
若配置文件中不存在如下信息，则添加：   
net.ipv4.tcp_keepalive_time = 1800   
net.ipv4.tcp_keepalive_probes = 3   
net.ipv4.tcp_keepalive_intvl = 15   
  
编辑完 /etc/sysctl.conf, 要重启 network 才会生效   
/etc/rc.d/init.d/network restart   
然后，执行 sysctl 命令使修改生效，基本上就算完成了。

问题总结：
-----

       上述操作完事后，常规的问题应该就能解决了，但是在我的场景中未能解决，CLOSE_WAIT 进程未能关闭，因为 netstat 命令中第二列的 REC-Q 也就是接收队列不为 0，有积压，导致 CLOSE_WAIT 未能被自动关闭，经排查，发现该端口对应的进程都成为了僵尸进程导致端口无法释放。

      僵尸进程又分为僵尸进程和孤儿进程，区别是通过 ps 命令查看僵尸进程的 PPID 是否为空，不为空是僵尸进程，为空是孤儿进程。如果是僵尸进程，处理方式参照我之前的博客 [https://blog.csdn.net/microGP/article/details/81509335](https://blog.csdn.net/microGP/article/details/81509335)；如果是孤儿进程，一般情况下操作系统在一定时间内自己就回收了，稍等即可