> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Michaelwubo/article/details/110823366)

### 一、IP 隧道技术

IP 隧道技术：是路由器把一种网络层协议封装到另一个协议中以跨过网络传送到另一个路由器的处理过程。IP 隧道（IP tunneling）是将一个 IP 报文封装在另一个 IP 报文的技术，这可以使得目标为一个 IP 地址的数据报文能被封装和转发到另一个 IP 地址。IP 隧道技术亦称为 IP 封装技术（IP encapsulation）。IP 隧道主要用于移动主机和虚拟私有网络（Virtual Private Network），在其中隧道都是静态建立的，隧道一端有一个 IP 地址，另一端也有唯一的 IP 地址。移动 IPv4 主要有三种隧道技术，它们分别是：IP in IP、最小封装以及通用路由封装。更多信息可以参看百度百科：[IP 隧道](http://baike.baidu.com/view/467497.htm) 和 [隧道技术](http://baike.baidu.com/view/626368.htm) 。

Linux 系统内核实现的 IP 隧道技术主要有三种（PPP、PPTP 和 L2TP 等协议或软件不是基于内核模块的）：ipip、gre、sit 。这三种隧道技术都需要内核模块 tunnel4.ko 的支持。

*   ipip 需要内核模块 ipip.ko ，该方式最为简单！但是你不能通过 IP-in-IP 隧道转发广播或者 IPv6 数据包。你只是连接了两个一般情况下无法直接通讯的 IPv4 网络而已。至于兼容性，这部分代码已经有很长一段历史了，它的兼容性可以上溯到 1.3 版的内核。据网上查到信息，Linux 的 IP-in-IP 隧道不能与其他操作系统或路由器互相通讯。它很简单，也很有效。
*   GRE 需要内核模块 ip_gre.ko ，GRE 是最初由 CISCO 开发出来的隧道协议，能够做一些 IP-in-IP 隧道做不到的事情。比如，你可以使用 GRE 隧道传输多播数据包和 IPv6 数据包。
*   sit 他的作用是连接 ipv4 与 ipv6 的网络。个人感觉不如 gre 使用广泛 

三个模块的信息如下：

1.  # sit 模块
    
    ```
    [root@localhost ~]# modinfo sit
    filename:       /lib/modules/3.10.0-1127.el7.x86_64/kernel/net/ipv6/sit.ko.xz
    alias:          netdev-sit0
    alias:          rtnl-link-sit
    license:        GPL
    retpoline:      Y
    rhelversion:    7.8
    srcversion:     8FEAE2838076CA07D989A03
    depends:        ip_tunnel,tunnel4
    intree:         Y
    vermagic:       3.10.0-1127.el7.x86_64 SMP mod_unload modversions 
    signer:         CentOS Linux kernel signing key
    sig_key:        69:0E:8A:48:2F:E7:6B:FB:F2:31:D8:60:F0:C6:62:D8:F1:17:3D:57
    sig_hashalgo:   sha256
    parm:           log_ecn_error:Log packets received with corrupted ECN (bool)
    ```
    
2.  # ipip 模块
    
    ```
    [root@localhost ~]# modinfo ipip
    filename:       /lib/modules/3.10.0-1127.el7.x86_64/kernel/net/ipv4/ipip.ko.xz
    alias:          netdev-tunl0
    alias:          rtnl-link-ipip
    license:        GPL
    retpoline:      Y
    rhelversion:    7.8
    srcversion:     8032CC3EDB2F63D42025A07
    depends:        ip_tunnel,tunnel4
    intree:         Y
    vermagic:       3.10.0-1127.el7.x86_64 SMP mod_unload modversions 
    signer:         CentOS Linux kernel signing key
    sig_key:        69:0E:8A:48:2F:E7:6B:FB:F2:31:D8:60:F0:C6:62:D8:F1:17:3D:57
    sig_hashalgo:   sha256
    parm:           log_ecn_error:Log packets received with corrupted ECN (bool)
    ```
    
3.  # ip_gre 模块
    
    ```
    [root@localhost ~]# modinfo ip_gre
    filename:       /lib/modules/3.10.0-1127.el7.x86_64/kernel/net/ipv4/ip_gre.ko.xz
    alias:          netdev-gretap0
    alias:          netdev-gre0
    alias:          rtnl-link-gretap
    alias:          rtnl-link-gre
    license:        GPL
    retpoline:      Y
    rhelversion:    7.8
    srcversion:     8D93B95BDB2B52FA2B08958
    depends:        ip_tunnel,gre
    intree:         Y
    vermagic:       3.10.0-1127.el7.x86_64 SMP mod_unload modversions 
    signer:         CentOS Linux kernel signing key
    sig_key:        69:0E:8A:48:2F:E7:6B:FB:F2:31:D8:60:F0:C6:62:D8:F1:17:3D:57
    sig_hashalgo:   sha256
    parm:           log_ecn_error:Log packets received with corrupted ECN (bool)
    ```
    

实验环境
----

ip tunnel 配置 Vmware 添加 2 个网卡，一个是桥接，一个是私有

hostA：

![](https://img-blog.csdnimg.cn/2020120815005992.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01pY2hhZWx3dWJv,size_16,color_FFFFFF,t_70)

```
[root@localhost ~]# ip address add 10.10.1.20/24 dev ens33
[root@localhost ~]# ip address add 192.168.1.2/24 dev ens37
[root@localhost ~]# ip link set dev ens33 up
[root@localhost ~]# ip link set dev ens37 up
[root@localhost ~]# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:f9:78:c3 brd ff:ff:ff:ff:ff:ff
    inet 10.10.1.20/24 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fef9:78c3/64 scope link 
       valid_lft forever preferred_lft forever
3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:f9:78:cd brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.2/24 scope global ens37
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fef9:78cd/64 scope link 
       valid_lft forever preferred_lft forever
```

hostB： 

![](https://img-blog.csdnimg.cn/2020120815012116.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01pY2hhZWx3dWJv,size_16,color_FFFFFF,t_70)

```
[root@localhost ~]# ip address add 10.10.1.21/24 dev ens38
[root@localhost ~]# ip address add 192.168.2.2/24 dev ens39
[root@localhost ~]# ip link set dev ens33 up
[root@localhost ~]# ip link set dev ens37 up
[root@localhost ~]# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
8: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:d0:47:fb brd ff:ff:ff:ff:ff:ff
    inet 10.10.1.21/24 scope global ens38
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fed0:47fb/64 scope link 
       valid_lft forever preferred_lft forever
9: ens39: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:d0:47:05 brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.2/24 scope global ens39
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fed0:4705/64 scope link 
       valid_lft forever preferred_lft forever
```

实际应用中，以上提到的三种 ip tunnel 技术和 VPN 技术一样，多用于跨公网的网络中（当然跨网段的内网环境也适用）。如上所示，我搞了两台虚机，其中 ens33 和 ens38 网段就类似于常见的公网网络。 ens37 和 ens39 网络为各自的私网 。最终实现效果是实现两台主机的 ens37 和 ens39 网络可以互通。这里测试中使用的是基于 gre 模式进行的实现，如果使用 ipip、sit，只需要把 modprobe 后面的模块换掉，把 ip tunnel 命令中 mode 后面的字符替换掉即可。

```
[root@localhost ~]# ping 10.10.1.21
PING 10.10.1.21 (10.10.1.21) 56(84) bytes of data.
64 bytes from 10.10.1.21: icmp_seq=1 ttl=64 time=0.809 ms
^C
--- 10.10.1.21 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.809/0.809/0.809/0.000 ms
[root@localhost ~]# ping 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=0.015 ms
^C
--- 192.168.1.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.015/0.015/0.015/0.000 ms
[root@localhost ~]# ping 192.168.2.2
connect: Network is unreachable
```

实验目的
----

在 host A 和 host B 之间建里 gre 隧道，另外在机器 A 上面配置 192.168.1.2， 在机器 B 上面配置 192.168.2.2 ，然后在 A 上面能够 ping -I 192.168.1.2 192.168.2.2 能够通

实验步骤
----

### 1、在 host A（10.10.1.20）上面操作

```
[root@localhost ~]# ip tunnel add ipiptun mode gre remote 10.10.1.21  local 10.10.1.20 ttl 64 dev ens33
[root@localhost ~]# ip addr add dev ipiptun 192.168.1.2/24 peer 192.168.2.2/24
[root@localhost ~]# ip link set dev ipiptun up
[root@localhost ~]# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:f9:78:c3 brd ff:ff:ff:ff:ff:ff
    inet 10.10.1.20/24 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fef9:78c3/64 scope link 
       valid_lft forever preferred_lft forever
3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:f9:78:cd brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.2/24 scope global ens37
       valid_lft forever preferred_lft forever
    inet 192.168.13.131/24 brd 192.168.13.255 scope global dynamic ens37
       valid_lft 1344sec preferred_lft 1344sec
    inet6 fe80::20c:29ff:fef9:78cd/64 scope link 
       valid_lft forever preferred_lft forever
8: gre0@NONE: <NOARP> mtu 1476 qdisc noop state DOWN group default qlen 1000
    link/gre 0.0.0.0 brd 0.0.0.0
9: gretap0@NONE: <BROADCAST,MULTICAST> mtu 1462 qdisc noop state DOWN group default qlen 1000
    link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
10: ipiptun@ens33: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1476 qdisc noqueue state UNKNOWN group default qlen 1000
    link/gre 10.10.1.20 peer 10.10.1.21
    inet 192.168.1.2 peer 192.168.2.2/24 scope global ipiptun
       valid_lft forever preferred_lft forever
    inet6 fe80::5efe:a0a:114/64 scope link 
       valid_lft forever preferred_lft forever
```

```
[root@localhost ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.10.1.0       0.0.0.0         255.255.255.0   U     0      0        0 ens33
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 ens37
192.168.2.0     0.0.0.0         255.255.255.0   U     0      0        0 ipiptun
```

如果没有路由信息需要手动填写，也是 ping 不通的

```
[root@localhost ~]# ip route del 192.168.2.0/24 dev ipiptun
[root@localhost ~]# ping -I 192.168.1.2 192.168.2.2
PING 192.168.2.2 (192.168.2.2) from 192.168.1.2 : 56(84) bytes of data.
ping: sendmsg: Network is unreachable
ping: sendmsg: Network is unreachable
ping: sendmsg: Network is unreachable
```

 添加路由

```
[root@localhost ~]# ip route add 192.168.2.0/24 dev ipiptun  #路由ip和ipiptun同网段可不写路由ip ，此时就是0.0.0.0,网关可以是任意主机任意ip只要能ping通就可以。
[root@localhost ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.10.1.0       0.0.0.0         255.255.255.0   U     0      0        0 ens33
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 ens37
192.168.2.0     0.0.0.0         255.255.255.0   U     0      0        0 ipiptun
```

防火墙相关，两边互 ping 发现可以 ping ，即为实验成功。这里需要注意 iptables 项，执行 iptables -F 是必须的，不然两边不通。如果在需要开启防火墙的情况下，也可以执行如下步骤：

```
[root@localhost ~]# iptables -F
[root@localhost ~]# iptables -I INPUT -p gre -j ACCEPT
或
[root@localhost ~]# firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT 0 -i ens33 -p gre -j ACCEPT （rhel7下默认使有的firewalld）
```

### 2、在 host B（10.10.1.21）上面操作

```
[root@localhost ~]# ip tunnel add ipiptun mode gre remote 10.10.1.20  local 10.10.1.21 ttl 64 dev ens38
[root@localhost ~]# ip addr add dev ipiptun 192.168.2.2/24 peer 192.168.1.2/24
[root@localhost ~]# ip link set dev ipiptun up
[root@localhost ~]# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
8: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:d0:47:fb brd ff:ff:ff:ff:ff:ff
    inet 10.10.1.21/24 scope global ens38
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fed0:47fb/64 scope link 
       valid_lft forever preferred_lft forever
9: ens39: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:d0:47:05 brd ff:ff:ff:ff:ff:ff
    inet 192.168.2.2/24 scope global ens39
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fed0:4705/64 scope link 
       valid_lft forever preferred_lft forever
10: gre0@NONE: <NOARP> mtu 1476 qdisc noop state DOWN group default qlen 1000
    link/gre 0.0.0.0 brd 0.0.0.0
11: gretap0@NONE: <BROADCAST,MULTICAST> mtu 1462 qdisc noop state DOWN group default qlen 1000
    link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
12: ipiptun@ens38: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1476 qdisc noqueue state UNKNOWN group default qlen 1000
    link/gre 10.10.1.21 peer 10.10.1.20
    inet 192.168.2.2 peer 192.168.1.2/24 scope global ipiptun
       valid_lft forever preferred_lft forever
    inet6 fe80::5efe:a0a:115/64 scope link 
       valid_lft forever preferred_lft forever
```

```
[root@localhost ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.10.1.0       0.0.0.0         255.255.255.0   U     0      0        0 ens38
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 ipiptun
192.168.2.0     0.0.0.0         255.255.255.0   U     0      0        0 ens39
```

 如果没有路由信息需要手动填写

```
[root@localhost ~]# ip route del 192.168.1.0/24 dev ipiptun
[root@localhost ~]# ping -I 10.10.1.21 192.168.1.2
PING 192.168.1.2 (192.168.1.2) from 10.10.1.21 : 56(84) bytes of data.
ping: sendmsg: Network is unreachable
ping: sendmsg: Network is unreachable
^C
--- 192.168.1.2 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 999ms
```

 添加路由

```
[root@localhost ~]# ip route add 192.168.1.0/24 dev ipiptun  #路由ip和ipiptun同网段可不写路由ip ，此时就是0.0.0.0,网关可以是任意主机任意ip只要能ping通就可以。
[root@localhost ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.10.1.0       0.0.0.0         255.255.255.0   U     0      0        0 ens33
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 ens37
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 ipiptun
```

防火墙相关，两边互 ping 发现可以 ping ，即为实验成功。这里需要注意 iptables 项，执行 iptables -F 是必须的，不然两边不通。如果在需要开启防火墙的情况下，也可以执行如下步骤：

```
[root@localhost ~]# iptables -F
[root@localhost ~]# iptables -I INPUT -p gre -j ACCEPT
或
[root@localhost ~]# firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT 0 -i ens38 -p gre -j ACCEPT （rhel7下默认使有的firewalld）
```

### 实验结果

在 host A（10.10.1.20）上面操作

```
[root@localhost ~]# ping -I 192.168.1.2 192.168.2.2 -c 4
PING 192.168.2.2 (192.168.2.2) from 192.168.1.2 : 56(84) bytes of data.
64 bytes from 192.168.2.2: icmp_seq=1 ttl=64 time=0.411 ms
64 bytes from 192.168.2.2: icmp_seq=2 ttl=64 time=1.07 ms
64 bytes from 192.168.2.2: icmp_seq=3 ttl=64 time=1.03 ms
64 bytes from 192.168.2.2: icmp_seq=4 ttl=64 time=1.02 ms
 
--- 192.168.2.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
```

在 host B（10.10.1.21）上面操作

```
[root@localhost ~]# ping -I 192.168.2.2 192.168.1.2 -c 4
PING 192.168.1.2 (192.168.1.2) from 192.168.2.2 : 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=0.943 ms
 
--- 192.168.1.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.943/0.943/0.943/0.000 m
```

host C:

假如这边还有台主机 host C，host C 主机只有一块网卡，其 IP 为 10.10.1.22，和 host A 主机同在 ens33 网段，可以将 a 主机配置为一个简单的种由器，其可以访问 b 主机的 IP 192.168.2.2 。只需要在 host A 主机中做如下配置即可。

```
 
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -I POSTROUTING -o ipiptun  -j MASQUERADE  #此处注意一定是ipiptun这个网卡，也就是说网络在哪个网卡就写那个网卡,MASQUERADE 比SNAT能自动学习网卡的IP变化
```

注意：该场景下，需要将 C 主机的网关指向 a 主机 。

```
[root@localhost ~]# ip route add 192.168.2.0/24 via 10.10.1.20 dev ens33
 
[root@gitlab ~]# ping 192.168.2.2 -c 4
PING 192.168.2.2 (192.168.2.2) 56(84) bytes of data.
64 bytes from 192.168.2.2: icmp_seq=1 ttl=63 time=1.58 ms
64 bytes from 192.168.2.2: icmp_seq=2 ttl=63 time=1.97 ms
64 bytes from 192.168.2.2: icmp_seq=3 ttl=63 time=2.20 ms
64 bytes from 192.168.2.2: icmp_seq=4 ttl=63 time=1.91 ms
 
--- 192.168.2.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 1.586/1.919/2.203/0.222 ms
 
 
 
[root@gitlab ~]# ip route add 192.168.1.0/24 via 10.10.1.20 dev ens33
[root@gitlab ~]# ping 192.168.1.2 -c 4
PING 192.168.1.2  (192.168.1.2 ) 56(84) bytes of data.
64 bytes from 192.168.1.2 : icmp_seq=1 ttl=63 time=1.58 ms
64 bytes from 192.168.1.2 : icmp_seq=2 ttl=63 time=1.97 ms
 
--- 192.168.1.2  ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 1.586/1.919/2.203/0.222 ms
```

### 还原实验环境

在 host A（10.10.1.20）执行

```
[root@localhost ~]# ip link set dev ipiptun down
[root@localhost ~]# ip tunnel del ipiptun
[root@localhost ~]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:f9:78:c3 brd ff:ff:ff:ff:ff:ff
3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:f9:78:cd brd ff:ff:ff:ff:ff:ff
8: gre0@NONE: <NOARP> mtu 1476 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/gre 0.0.0.0 brd 0.0.0.0
9: gretap0@NONE: <BROADCAST,MULTICAST> mtu 1462 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
```

  
在 host B（10.10.1.21）上面操作

```
[root@localhost ~]# ip link set dev ipiptun down
[root@localhost ~]# ip tunnel del ipiptun
[root@localhost ~]#  ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
8: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:d0:47:fb brd ff:ff:ff:ff:ff:ff
9: ens39: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:d0:47:05 brd ff:ff:ff:ff:ff:ff
10: gre0@NONE: <NOARP> mtu 1476 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/gre 0.0.0.0 brd 0.0.0.0
11: gretap0@NONE: <BROADCAST,MULTICAST> mtu 1462 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
```

最后还有一个 gre0，不能用  
ip link set gre0 down  
ip tunnel del gre0  
上面两个命令删除，否则会报错

```
[root@localhost ~]# ip link set dev gre0 down
[root@localhost ~]# ip tunnel del gre0
delete tunnel "gre0" failed: Operation not permitted
[root@localhost ~]# ip link delete dev gre0
[root@localhost ~]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:f9:78:c3 brd ff:ff:ff:ff:ff:ff
3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:f9:78:cd brd ff:ff:ff:ff:ff:ff
8: gre0@NONE: <NOARP> mtu 1476 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/gre 0.0.0.0 brd 0.0.0.0
9: gretap0@NONE: <BROADCAST,MULTICAST> mtu 1462 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
[root@localhost ~]# 
```

2 台机器一样操作，需要按照下面的命令删除：

```
[root@localhost ~]# lsmod|grep gre
ip_gre                 22749  0 
gre                    13144  1 ip_gre
ip_tunnel              25163  1 ip_gre
[root@localhost ~]# rmmod ip_gre
[root@localhost ~]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:f9:78:c3 brd ff:ff:ff:ff:ff:ff
3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:f9:78:cd brd ff:ff:ff:ff:ff:ff
```

=======================================================================================================

### 需求

有 2 个相互隔离的网络，拓扑见下图。ServerA 想直接访问到 ServerB 连接的私有网络

```
                                                  |
            10.10.1.20             10.10.1.21     |
            +---------+  Public   +---------+     | Private
            | ServerA +-----------+ ServerB +-----+
            +---------+  Network  +---------+     | Network
                                                  |
                                                  | 192.168.1.0/24
```

### 实现

通过 ip tunnel 建立 ipip 隧道，再通过 iptables 进行 nat，便可以实现。  
**Step 1. 建立 ip 隧道**  
ServerA(10.10.1.20),tun0(192.168.1.2) 配置 iptunnel, 并给 tunnel 接口配置上 ip

```
ip tunnel add tun0 mode ipip remote 10.10.1.21 local 10.10.1.20
ifconfig tun0 192.168.2.2 netmask 255.255.255.0
```

ServerB 配置 iptunnel, 并给 tunnel 接口配置上 ip

```
ip tunnel add tun0 mode ipip remote 10.10.1.20 local 10.10.1.21
ifconfig tun0 192.168.2.3 netmask 255.255.255.0
```

隧道配置完成后，请在 ServerA 上 192.168.2.3，看是否可以 ping 通，ping 通则继续，ping 不通需要再看一下上面的命令执行是否有报错

**Step 2. 添加路由和 nat**  
ServerA 上，添加到 192.168.1.0/24 的路由, 意思就是说 ServerA 方位 192.168.1.0 网段的主机通过 ServerB（192.168.2.3） ip 转发出去。此时 ServerB（192.168.2.3）的 IP 就是 ServerA 的网关

```
route add -net 192.168.1.0/24 gw 192.168.2.3

```

ServerB 上，添加 iptables nat，将 ServerA 过了访问 192.168.1.0/24 段的包进行 NAT，并开启 ip foward 功能

```
sysctl -w net.ipv4.ip_forward=1
sed -i '/net.ipv4.ip_forward/ s/0/1/'  /etc/sysctl.conf
```

```
iptables -t nat -A POSTROUTING -s 192.168.2.2 -d 192.168.1.0/24 -j MASQUERADE

```

###  至此，完成了两端的配置，ServerA 可以直接访问 ServerB 所接的私网了

参考

[http://www.opstool.com/article/183](http://www.opstool.com/article/183)

[https://www.cnblogs.com/weifeng1463/p/6805856.html](https://www.cnblogs.com/weifeng1463/p/6805856.html)

[https://docs.oracle.com/cd/E26926_01/html/E25874/gepbe.html](https://docs.oracle.com/cd/E26926_01/html/E25874/gepbe.html)

[https://wiki.linuxfoundation.org/networking/tunneling](https://wiki.linuxfoundation.org/networking/tunneling)

[https://www.cnblogs.com/0pandas0/p/12005218.html](https://www.cnblogs.com/0pandas0/p/12005218.html)

[https://wenku.baidu.com/view/79da02ed172ded630b1cb621.html](https://wenku.baidu.com/view/79da02ed172ded630b1cb621.html)