> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [aoarasi.com](https://aoarasi.com/archives/033)

![](https://aoarasi.com/upload/2022/05/image-a4b02fc13ac64c1c92bb00fe394db62a.png)

端口转发服务是由一组端口转发规则定义的策略。一个端口转发服务可以应用到一个或更多的服务器。然后服务器的入站网络访问就根据端口转发服务所定义的策略进行管理。可以根据需要指定一个或多个 CIDR 来过滤源 lP 地址，以允许来自特定 IP 地址的请求被转发。

这里介绍一下使用 Windows 服务器实现端口转发的功能，以一台 Windows 服务器转发另一台 Linux 服务器为例。

首先要找到 Windows 云服务器的内网 IP，不是你的公网 IP，使用命令 ipconfig 即可找到。

对于 WindowsServer 2008 以下版本的系统，需要安装 IPV6 才行，如果是 Windows Server 2008 或者以上的系统则默认已经支持。

之后，使用 Portproxy 模式下的 Netsh 命令即能实现 Windows 系统中的端口转发，转发命令如下，执行命令前，请使用管理员权限。

解释一下这其中的参数意义

*   1.listenaddress – 等待连接的本地 ip 地址
    
*   2.listenport – 本地监听的 TCP 端口（待转发）
    
*   3.connectaddress – 被转发端口的本地或者远程主机的 ip 地址
    
*   4.connectport – 被转发的端口
    

> 使用 netsh interface portproxy add v4tov6/v6tov4/v6tov6 选项，可以在 IPv4 和 IPv6 地址之间创建端口转发规则。

这里举个例子，服务器内网 IP 是 172.16.0.4，需要将 3340 端口转发到 Linux 服务器 104.104.104.104 的 9999 端口，那么命令如下：

如果没有任何回显，说明执行成功。

![](http://images.aoarasi.com/blog7c3fccb9-677c-4a00-9373-64abfcd66356.png)

注意转发前需要确认端口是否占用：

![](http://images.aoarasi.com/bloge23961b0-5ae4-4497-b34b-a61a74dfa947.png)

如果开启了防火墙，需要使用以下命令放行转发的端口：

![](http://images.aoarasi.com/blog575f93f9-9493-4ce2-9151-d65a57605fe0.png)

下面的命令是用来展示系统中的所有转发规则：

![](http://images.aoarasi.com/blogb1541e5b-178a-449f-b29e-225714fce719.png)

删除刚才创建的那个转发的命令：

删除之后再次查看，发现刚刚的记录已经没有了。

![](http://images.aoarasi.com/blog79f164a9-2fc1-4e1c-aa69-79aa7ec386dc.png)

若要进行 tcp 端口重定向：

前面的命令默认是转发 tcp 端口，如果要转发 udp 端口，如将本机的 53 端口转发至 192.168.100.100 的 53 端口：

注意：连接时请确保防火墙（Windows 防火墙或者其他的第三方防护软件）允许外部连接到一个全新的端口，如果不允许，那么只能自行添加一个新的 Windows 防火墙规则。

该命令的常用参数如下：