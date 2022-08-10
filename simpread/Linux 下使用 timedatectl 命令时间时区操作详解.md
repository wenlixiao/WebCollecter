> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/zhi-leaf/p/6282301.html)

　　timedatectl 命令对于 RHEL / CentOS 7 和基于 Fedora 21 + 的分布式系统来说，是一个新工具，它作为 systemd 系统和服务管理器的一部分，代替旧的传统的用在基于 Linux 分布式系统的 sysvinit 守护进程的 date 命令。

　　timedatectl 命令可以查询和更改系统时钟和设置，你可以使用此命令来设置或更改当前的日期，时间和时区，或实现与远程 NTP 服务器的自动系统时钟同步。

　　在本教程中，我要讲的是，如何在你的 Linux 系统上，通过使用来自于终端使用 timedatectl 命令的 NTP，设置 date、time、timezone 和 synchronize time 来管理时间。让你的 Linux 服务器或系统保持正确的时间是一个很好的实践，它有以下优点：

　　1）维护及时操作的系统任务，因为在 Linux 中的大多数任务都是由时间来控制的。

　　2）记录事件和系统上其它信息等的正确时间。

**如何查找和设置 Linux 本地时区**

1、要显示系统的当前时间和日期，使用命令行中的 timedatectl 命令，如下：

```
# timedatectl status

```

![](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113125840760-1412416063.gif)

　　在上面的示例中，RTC time 就是硬件时钟的时间。

2、Linux 系统上的 time 总是通过系统上的 timezone 设置的，要查看当前时区，按如下做：

```
# timedatectl 
OR
# timedatectl | grep Time

```

![](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113130255760-443096740.gif)

3、要查看所有可用的时区，运行以下命令：

```
# timedatectl list-timezones

```

![](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113130353681-1064142284.gif)

4、要根据地理位置找到本地的时区，运行以下命令：

```
# timedatectl list-timezones | egrep -o ‘’Asia/B.*”
# timedatectl list-timezones | egrep -o “Europe/L.*”
# timedatectl list-timezones | egrep -o “America/N.*”

```

![](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113130554931-355176850.gif)

5、要在 Linux 中设置本地时区，使用 set-timezone 开关，如下所示。

```
# timedatectl set-timezone "Asia/Kolkata"

```

　　中国上海的时区：

```
# timedatectl set-timezone "Asia/Shanghai"

```

![](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113130723338-1706546881.gif)

 　　推荐使用和设置协调世界时，即 UTC。

```
# timedatectl set-timezone UTC

```

![](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113130915228-1721727285.gif)

 　　你需要输入正确命名的时区，否者在你改变时区的时候，可能会发生错误。在下面的例子中，由于 “Asia/Kalkata” 这个时区是不正确的，因此导致了错误。

![](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113131009728-29191656.gif)

**如何在 Linux 中设置时间和日期**

　　你可以使用 timedatectl 命令，设置系统上的日期和时间，如下所示：

6、设置 Linux 中的时间。只设置时间的话，我们可以使用 set-time 开关以及 HH：MM：SS（小时，分，秒）的时间格式。

```
# timedatectl set-time 15:58:30

```

![](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113131240260-1561897221.gif)

 7、在 Linux 中设置日期。只设置日期的话，我们可以使用 set-time 开关以及 YY：MM：DD（年，月，日）的日期格式。

```
# timedatectl set-time 20151120

```

![](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113131335525-1442987888.gif)

8、设置日期和时间：

```
# timedatectl set-time '16:10:40 2015-11-20'

```

![](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113131704369-1771813681.gif)

**如何在 Linux 中查找和设置硬件时钟**

9、要设置硬件时钟以协调世界时，UTC，可以使用 set-local-rtc boolean-value 选项，如下所示：

　　首先确定你的硬件时钟是否设置为本地时区：

```
# timedatectl | grep local

```

　　将你的硬件时钟设置为本地时区：

```
# timedatectl set-local-rtc 1

```

![](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113132023681-700633329.gif)

　　将你的硬件时钟设置为协调世界时（UTC）：

```
# timedatectl set-local-rtc 0

```

![](https://images2015.cnblogs.com/blog/1031555/201701/1031555-20170113131936135-1877678773.gif)

**将 Linux 系统时钟同步到远程 NTP 服务器**

　　NTP 即 Network Time Protocol（网络时间协议），是一个互联网协议，用于同步计算机之间的系统时钟。timedatectl 实用程序可以自动同步你的 Linux 系统时钟到使用 NTP 的远程服务器。

　　注意，你必须在系统上安装 NTP 以实现与 NTP 服务器的自动时间同步。

　　要开始自动时间同步到远程 NTP 服务器，在终端键入以下命令。

```
# timedatectl set-ntp true

```

　　要禁用 NTP 时间同步，在终端键入以下命令。

```
# timedatectl set-ntp false

```

原文链接：http://www.codeceo.com/article/linux-timedatectl-set-time.html

英文原文：[How to Set Time, Timezone and Synchronize System Clock Using timedatectl Command](http://www.tecmint.com/set-time-timezone-and-synchronize-time-using-timedatectl-command/)