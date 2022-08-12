> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.junmajinlong.com](https://www.junmajinlong.com/linux/systemd/systemd_timesyncd/)

> 回到 Linux 基础系列文章大纲回到 Systemd 系列文章大纲回到 Shell 系列文章大纲 使用 systemd timesyncd 做时间同步 CentOS 8 中已经移除了 ntp 和 ntpdate，它们也没有集成在基础包中。

* * *

**[回到 Linux 基础系列文章大纲](https://www.junmajinlong.com/linux/index)**  
**[回到 Systemd 系列文章大纲](https://www.junmajinlong.com/linux/index#systemd)**  
**[回到 Shell 系列文章大纲](https://www.junmajinlong.com/shell/index)**

* * *

CentOS 8 中已经移除了 ntp 和 ntpdate，它们也没有集成在基础包中。

CentOS 8 使用 chronyd 作为时间服务器，但如果只是简单做时间同步，可直接使用 systemd.timesyncd 组件。

timesyncd 虽然没有 chronyd 更健壮，但胜在简单方便，只需配置一项配置文件并执行一个命令启动便可定时同步。

```
$ vim /etc/systemd/timesyncd.conf
[Time]
NTP=ntp1.aliyun.com ntp2.aliyun.com
# 以下四项均可省略
FallbackNTP=1.cn.pool.ntp.org 2.cn.pool.ntp.org
RootDistanceMaxSec=5
PollIntervalMinSec=32
PollIntervalMaxSec=2048
```

其它常用的网络时间服务器：

```
cn.pool.ntp.org
1.cn.pool.ntp.org
2.cn.pool.ntp.org
3.cn.pool.ntp.org
0.cn.pool.ntp.org

ntp1.aliyun.com
ntp2.aliyun.com
ntp3.aliyun.com
ntp4.aliyun.com
ntp5.aliyun.com
ntp6.aliyun.com
ntp7.aliyun.com
```

配置好 timesyncd.conf 后，启动 systemd timesyncd 时间同步服务：

```
$ timedatectl set-ntp true
```

查看同步状态：

```
$ timedatectl status
               Local time: Sat 2020-07-04 20:01:41 CST
           Universal time: Sat 2020-07-04 12:01:41 UTC
                 RTC time: Sat 2020-07-04 20:01:40
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: yes
              NTP service: inactive
          RTC in local TZ: no
# 或者
$ timedatectl show 
Timezone=Asia/Shanghai
LocalRTC=no
CanNTP=yes
NTP=no
NTPSynchronized=yes
TimeUSec=Sat 2020-07-04 20:01:41 CST
RTCTimeUSec=Sun 2020-07-05 04:01:40 CST
```

版权声明: 本博客所有文章除特别声明外，均采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可协议。转载请注明来自 [骏马金龙](https://www.junmajinlong.com/)！

* * *