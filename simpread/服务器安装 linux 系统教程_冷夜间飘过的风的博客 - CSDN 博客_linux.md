> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_15057857/article/details/89841590)

**一、linux 镜像的刻录**

1. 首先打开电脑上面任意浏览器（IE、Microsoft Edge、chrome、Firefox）, 输入网址   [https://www.centos.org/](https://www.centos.org/) 我们可以看到如下界面选择立即获取 centos 下载最新的安装镜像，复制下载链接（[http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso)）打开迅雷，等待一刻钟（等待时间依个人实际网速决定）。![](https://img-blog.csdnimg.cn/20190505111448120.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

2. 下载专用的 linux 镜像刻录工具地址：[https://www.pendrivelinux.com/](https://www.pendrivelinux.com/)

![](https://img-blog.csdnimg.cn/20190505112015518.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

完成后双击刚刚下载的可执行文件，出现下图所示的界面

![](https://img-blog.csdnimg.cn/20190505113108162.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

选择我同意，进入如下界面，现在的服务器基本上都是支持 UEFI 启动但是也不乏可能出现不支持的情况，所以我们在选择镜录的时候选择 cnetos installer，这里需要注意一点买一个好点的 U 盘因为刻录的时候会复制 centos 镜像至 U 盘，而镜像的大小为 4GB+，所以 U 盘的文件系统需要为 NTFS 格式箭头所示的地方√上。

![](https://img-blog.csdnimg.cn/20190505115657445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

准备就绪点击 create，我使用的为 usb3.0U 盘所以基本上等待 5 分钟即可完成。如下图，至此我们就完成了镜像的刻录！！

![](https://img-blog.csdnimg.cn/20190505115456226.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

**二、系统的安装**

1、我们的演示环境为一台 dell 服务器，开机进入系统初始化界面，按 F11 进入启动管理如下图：

![](https://img-blog.csdnimg.cn/20190505131302437.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

如下图我们进入的为戴尔启动管理（dell boot manager）设置启动模式为 bios，图中红框所示，不同型号的服务器可能设置方法有差异，但只要记住两点第一 boot manager 这个是开机时键盘要选择的看好是哪个按钮，第二启动模式（boot mode）设置为 bios。

![](https://img-blog.csdnimg.cn/20190505131529837.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

再之后选择启动的硬盘为插入的 U 盘，这里我使用的为闪迪的 U 盘可以看到图中所示为 sandisk（闪迪）。↓键选择后按 enter 键，即可从 U 盘引导。

![](https://img-blog.csdnimg.cn/20190505131951473.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

下面即可进入 centos 的安装界面这里就不过多的介绍了，看截图

![](https://img-blog.csdnimg.cn/201905051326570.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20190505133903773.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/2019050513431330.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20190505134603123.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20190505134705180.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

期望容量这里改为 100GB，如下图

![](https://img-blog.csdnimg.cn/20190505134746913.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20190505135443300.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

/ 分区为系统分区承载系统文件这里我们一般配置为 100GB，/boot 分区为系统引导分区这里我们配置为 1GB，swap 为交换分区这里我们配置为物理内存的两倍（如果服务器插上了一条 8G 的内存条，这里我们配置就为 16Gb），至此系统分区完成。然后开始系统的安装，这里我们设置 root 的密码，可为 123456 后续再自行更改，设置好后就是漫长的等待。

![](https://img-blog.csdnimg.cn/20190505135952331.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20190505140638958.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

至此安装完成，可以重启。

2. 配置 ip

开机进入系统输入用户名密码，然后进入图中所示的界面输入如下图所示命令，我这里虚拟机环境，如果是真机进入系统网络配置目录，会看到, ifcfg-ens1、ifcfg-ens2、ifcfg-ens3、ifcfg-ens4，我们需要配置的为服务器的第一个网口，所以编辑 ifcfg-ens1

![](https://img-blog.csdnimg.cn/20190505141133415.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

![](https://img-blog.csdnimg.cn/20190505142206513.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzE1MDU3ODU3,size_16,color_FFFFFF,t_70)

如图根剧实际情况配置 IP、MASK、GATEWAY、DNS，如果不知道怎么配置可自行百度。下面再说明一下：

```
TYPE=Ethernet
BOOTPROTO=dhcp
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=eno16777736
UUID=ae05ccde-6a29-4332-b486-f3042da73ac0
DEVICE=eno16777736
ONBOOT=no
```

这里说一下需要修改的位置:

```
#修改
BOOTPROTO=static #这里讲dhcp换成ststic
ONBOOT=yes #将no换成yes
#新增
IPADDR=192.168.85.100 #静态IP
GATEWAY=192.168.85.2 #默认网关
NETMASK=255.255.255.0 #子网掩码
```

保存退出后, 重启网络服务:

```
# service network restart
Restarting network (via systemctl):                        [  确定  ]
```

查看当前 ip:

```
# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eno16777736: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:e7:b8:77 brd ff:ff:ff:ff:ff:ff
    inet 192.168.85.100/24 brd 192.168.85.255 scope global eno16777736
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fee7:b877/64 scope link 
       valid_lft forever preferred_lft forever
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
    link/ether 52:54:00:b9:8f:6c brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 500
    link/ether 52:54:00:b9:8f:6c brd ff:ff:ff:ff:ff:ff
```

配置好后测试一下 ping baidu.com 是否可接受到回包，如果可以，到这里服务器的基本配置已经完成，可以上架了。