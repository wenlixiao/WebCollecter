> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/cyent/2309744?abTest=51cto)

> 最详细的 CentOS 6 与 CentOS 7 的对比（包括常见设置、服务管理、性能测试），网络上有许多 CentOS6 与 7 的对比文章，但是都比较浅，对工作帮助并不大本文由小慢哥花费大量时间精心整理而成，确保内......

    本主题将从 3 个角度进行对比

    1. 常见设置（CentOS 6 vs CentOS 7）

    2. 服务管理（Sysvinit vs Upstart vs Systemd）

    3. 性能测试（cpu/mem/io/oltp）

* * *

### **环境说明**

硬件

*   服务器: Dell PowerEdge R620
    
*   CPU: E5-2620 v2 @ 2.10GHz * 2
    
*   MEM: 8G DDR3 1333 MHz * 4
    
*   DISK: 300G SSD * 1
    
*   BIOS: 默认
    

系统

*   CentOS 6: CentOS 6.10 (2.6.32-754.el6.x86_64)
    
*   CentOS 7: CentOS 7.5 (3.10.0-862.el7.x86_64)
    

* * *

### **一. 常见设置**

### **1. 字符集**

CentOS 6

*   方法: /etc/sysconfig/i18n
    

CentOS 7

*   方法 1: localectl set-locale.utf8
    
*   方法 2: /etc/locale.conf` 中的 LANG=
    

### **2. 主机名**

CentOS 6

*   在线生效: hostname
    
*   重启生效: /etc/sysconfig/network 中的 HOSTNAME=
    

CentOS 7

*   在线 + 重启生效: hostnamectl set-hostname
    

### **3. 时区**

CentOS 6

*   方法: ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    

CentOS 7

*   方法 1: 同 CentOS 6
    
*   方法 2: timedatectl set-timezone Asia/Shanghai
    

### **4. 时间同步**

CentOS 6

*   逐步: ntpd 或 ntpdate
    
*   直接: ntpdate -b（通常加到 crontab）
    

CentOS 7

*   方法 1: systemctl start chronyd
    
*   方法 2: timedatectl set-ntp yes（同 systemctl start chronyd）
    

> 可以通过 timedatectl | grep "NTP synchronized" 判断当前时间是否已同步
> 
> 不建议用 ntpd 和 ntpdate，redhat 强烈推荐 chrony，可用于网络不稳定的环境 chrony.conf 关键参数
> 
> makestep 1.0 -1 [ ntpd 和 chronyd 区别](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-configuring_ntp_using_the_chrony_suite#sect-differences_between_ntpd_and_chronyd)

### **5. 手动更改时间**

CentOS 6

*   方法: date -s "2018-07-08 11:11:11"
    

CentOS 7

*   方法 1: 同 CentOS 6
    
*   方法 2: timedatectl set-time "2018-07-08 11:11:12"（前提是 timedatectl set-ntp false）
    

### **6. 单用户修改密码**

CentOS 6: `grub`界面键入 `e`，在 `kernel`行最后加 `1`，键入 `b`启动进入单用户模式，之后输入 `passwd`修改密码

CentOS 7: `grub`界面键入 `e`，在 `linux16`行上将 `ro`改为 `rw`，并在当前行最后加 `init=/bin/sh`，键入 `ctrl-x`进入，之后输入 `passwd`修改密码

*   如果有开启 selinux，则需要在修改密码后，重启前，执行 touch/.autorelabel
    
*   passwd 执行后，最好执行 sync，防止强制重启导致修改密码没有落地
    

### **7. grub 添加参数**

CentOS 6:

*   /boot/grub/grub.conf 的 kernel 中加入需要添加的参数
    

CentOS 7:

*   步骤 1：/etc/default/grub 的 GRUBCMDLINELINUX 中加入需要添加的参数
    
*   步骤 2：grub2-mkconfig -o /boot/grub2/grub.cfg
    

### **8. 查看开机记录**

CentOS 6: last

CentOS 7: journalctl --list-boots 或 last

### **9. 修改启动内核**

1. 查看当前启动内核

*   CentOS 6: cat /boot/grub/grub.conf 中的 default
    
*   CentOS 7: grub2-editenv list
    

2. 查看有哪些内核

*   CentOS 6: cat /boot/grub/grub.conf | sed -n '/^title/s/^title //p'
    
*   CentOS 7: cat /boot/grub2/grub.cfg | grep '^menuentry' | awk -F"'"'{print $2}'
    

3. 设置启动内核

*   CentOS 6:
    

*   修改 / boot/grub/grub.conf 中的 default
    

*   CentOS 7:
    

*   步骤 1：确保 / etc/default/grub 中的 GRUB_DEFAULT 为 saved
    
*   步骤 2：grub2-set-default 'CentOS Linux (3.10.0-693.el7.x86_64) 7 (Core)'
    

### **10. rc.local**

执行顺序

*   CentOS 6: 串行的最后一个执行
    
*   CentOS 7: 和其他服务并行执行
    

可执行权限

*   CentOS 6: 默认有可执行权限
    
*   CentOS 7: 默认没有可执行权限（官方不推荐使用 rc.local），需要自行增加（chmod +x /etc/rc.d/rc.local）
    

CentOS 7 的注意事项

*   rc.local 由 rc-local.service 执行，由于 systemd 服务是并行执行，仅能保证在 network 之后启动，因此建议 rc.local 里增加 sleep 10 来尽可能在最后执行
    
*   需要在 rc.local 的最后一行增加 exit 0，否则可能导致已启动的进程被关闭（echo 'exit 0' >> /etc/rc.d/rc.local）
    
*   建议尽量使用 systemd 来配置服务，不要使用 rc.local
    

### **11. limit 配置**

CentOS 6:

*   全局设置: 没有全局设置的方法（/etc/security/limits.conf 仅针对使用 pam 的进程，且有加载 pamlimits.so 的模块，因为 limits.conf 是 pamlimits.so 的配置文件）
    
*   服务设置: 只能在服务启动前设置 ulimit，才能在启动后看到效果
    

CentOS 7:

*   全局设置: /etc/systemd/system.conf 里 DefaultLimitNOFILE=65535
    
*   服务设置: [Service] 里增加 LimitNOFILE=65535
    

### **12. yum 仅使用 ipv4**

CentOS 6: yum 没有自带方法

CentOS 7: yum.conf 里增加 ip_resolve=4

### **13. 彻底禁用 ipv6**

CentOS 6 和 CentOS 7 相同

*   在 grub 上增加 ipv6.disable=1
    

查看是否彻底关闭

*   sysctl -a | grep -i ipv6 如果没有任何输出，则表示彻底关闭
    

### **14. 防火墙**

CentOS 6

*   默认开启 iptables 服务，只不过默认没有条目
    

CentOS 7

*   默认安装并开启 firewalld 服务
    
*   默认不安装 iptables 服务（yum install iptables-services）
    

### **15. NetworkManager**

CentOS 6: 默认未安装

CentOS 7: 默认安装并启动

### **16. 网卡名**

CentOS 6:

*   系统安装完，默认是 em1 开始，这其实是在装机完成时在 udev 里做的绑定
    
*   把 / etc/udev/rules.d/70-persistent-net.rules 内容清空，则恢复成 eth0 开始编号
    

CentOS 7:

*   不再通过 udev 绑定网卡名，默认是 em1 开始，有的是 eno、enp、ens 等名字
    
*   如果想恢复 eth0，则 / etc/default/grub 里增加 net.ifnames=0 biosdevname=0
    
*   如果想让 CentOS 6 的网卡名不受 udev 影响，达到 CentOS 7 的效果，则删除 3 个文件即可
    

网卡名规则

*   eno：主板板载网卡
    
*   enp：独立网卡（PCI 网卡）
    
*   ens：热插拔网卡（usb 之类）
    
*   参考：https://www.cnblogs.com/chia/p/7379775.html
    

### **17. CPU 频率（performance）**

CentOS 6

*   始终：2.1GHz
    

![](http://s2.51cto.com/images/20181027/1540646072275086.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

CentOS 7:

*   空闲：1.2GHz
    

![](http://s2.51cto.com/images/20181027/1540646086440688.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

*   sysbench 1 线程压测：一个物理 cpu 所有核的频率瞬间增长，其中最高打到 2.6GHz
    

![](http://s2.51cto.com/images/20181027/1540646096430434.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

*   sysbench 42 线程压测：所有 cpu 所有核的频率全部达到 2.4GHz
    

![](http://s2.51cto.com/images/20181027/1540646104983709.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

*   若要和 6 一样保持频率，则在 / etc/default/grub 里增加 intel_pstate=disable（不建议，因为性能没有任何提升，还在某些情况下降）
    

* * *

**二. 服务管理  
**
--------------

### **1. sysvinit、upstart、systemd 简介**

### ![](http://s2.51cto.com/images/20181027/1540644393219221.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)  
**2. sysvinit、upstart、systemd 常用命令**

![](http://s2.51cto.com/images/20181027/1540644408523462.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### **3. runlevel 运行级别**

![](http://s2.51cto.com/images/20181027/1540644438661048.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

 **4. 日志查询**

CentOS 6: 手工在 / var/log/messages、/var/log/dmesg、/var/log/secure 中 grep，麻烦且效率低

CentOS 7: 统一使用 journalctl，可以使用多个因素匹配，比如时间段、服务名、日志级别等等。另外，systemd 日志默认经过压缩，是二进制文件，无法直接查看

![](http://s2.51cto.com/images/20181027/1540644467428287.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)**5. 实现守护进程**

CentOS 6

*   sysvinit 需要自行实现，如:
    

*   - nohup &
    
*   - screen
    
*   - supervisor
    

*   upstart 和 systemd 类似，将程序运行在前台即可
    

CentOS 7

*   由 systemd 启动，将程序运行在前台即可
    

### **6. sysvinit、upstart、systemd 例子**

sysvinit

upstart

systemd

### **7. PID 管理**

sysvinit: 需要生成 PID 文件，用于后期关闭、重启等使用

upstart: 无需 PID 文件，upstart 会记录主进程 ID，子进程 ID 没有记录

systemd: 无需 PID 文件，所有进程 ID 由 cgroup 统一接管

### **8. 内置的资源限制**

CentOS 6: 除了 ulimit，没有其他限制进程资源的简便方法

CentOS 7: 除了 ulimit，还支持部分 cgroup 限制，可对进程做内存限制和 cpu 资源限制等

另外，CentOS 7 可以通过 systemd-cgtop 命令查看 cgroup 里的性能数据

### **9. 服务异常自动重启**

upstart

systemd

上面 2 种方式均表示，无限次自动重启，每次重启前等待 5 秒

### **10. 写日志方式**

CentOS 6: 自行输出到文件中，或通过 syslog 记录（如 logger 命令）

CentOS 7: 只要程序由 systemd 启动，只需将输出日志到标准输出或标准错误

*   建议 centos7 只将应用程序的一些元信息输出到标准输出或标准错误，比如启动成功、启动失败等等
    
*   不建议将业务日志输出到 journal。因为 journal 中所有日志都存在一个文件中，会导致 2 个问题：
    

*   1. 如果没有做日志持久化，则默认存在内存中，会导致最多一半的内存被占用
    
*   2. 存储量很大，会导致查询其他日志很耗时
    

*   解决办法：输出到 syslog，[Service] 支持 StandardOutput=syslog
    

### **11. 指定每条日志级别**

CentOS 6: 通过 syslog 将不同级别的日志输出到不同文件

CentOS 7: 只需在输出的每一行开头加 <日志级别>，比如

### **12. systemd 日志永久保存**

systemd 日志默认保存在内存中，因此当服务器重启后，就无法通过 journalctl 来查看之前的日志，解决方法:

* * *

**三. 性能对比**
-----------

### **1. CPU 测试**

工具: 通过 sysbench 对 cpu 进行压力测试

参数设置

*   素数: 10000
    
*   测试时间: 900 秒
    
*   线程数: 1、6、12、18、24、30、36、42
    

分别测试使用睿频和不实用睿频

**> 图 1: cpu 测试 - 每秒 events**

![](https://fzxiaomange.com/img/el67/sysbench-cpu-event-ps.png)

如何看图：越高越好

此图结论：

*   cpu 性能基本一致
    
*   CentOS 7 固定频率（不使用睿频），并没有提升性能，因此没有关闭睿频的必要
    

**> 图 2: cpu 测试 - event 数量标准差**

![](https://fzxiaomange.com/img/el67/sysbench-cpu-event-stdev.png)

如何看图：越少越好

此图结论：

*   通过标准差可以看出在稳定性方面，CentOS 7 要稳定很多（包括不使用睿频）
    

### **2. 内存测试**

工具: 通过 sysbench 对内存进行压力测试

参数设置

*   读写方式: 随机
    
*   测试时间: 900 秒
    
*   分别测试读和写
    
*   块大小: 4K、16K、2M
    
*   线程数: 1、12、24、36、48
    

**> 图 1: 内存测试 - 速率**

![](https://fzxiaomange.com/img/el67/sysbench-mem-speed.png)

如何看图：越高越好

此图结论：

*   CentOS 6 和 CentOS 7 性能一致
    

**> 图 2: 内存测试 - event 数量标准差**

![](https://fzxiaomange.com/img/el67/sysbench-mem-event-stdev.png)

如何看图：越少越好

此图结论：

*   通过标准差可以看出在稳定性方面，CentOS 7 要稳定很多
    

### **3. IO 测试**

工具: 通过 fio 对 io 进行压力测试

参数设置

*   ioengine: libaio
    
*   iodepth: 16
    
*   测试时间: 900 秒
    
*   文件大小: 100G
    
*   运行方式: 线程
    
*   缓存方式: 无缓存（non-buffered I/O）
    
*   读写方式: 随机读写
    
*   块大小: 分别测试 4K 和 16K
    
*   线程数: 1、12、24、36、48
    

**> 图 1: io 测试 - iops**

![](https://fzxiaomange.com/img/el67/fio-iops.png)

如何看图：越高越好

此图结论：

*   CentOS 6（默认 ext4）不如 CentOS 7（默认 xfs）
    
*   CentOS 6（默认 ext4）不如 CentOS 6（xfs）
    

**> 图 2: io 测试 - 读写平均延时**

![](https://fzxiaomange.com/img/el67/fio-rw-avg.png)

如何看图：越少越好

此图结论：

*   CentOS 7 的写延时和 CentOS 6（默认 ext4）接近
    
*   CentOS 7 的读延时比 CentOS 6（默认 ext4）好很多
    

### **4. OLTP 测试**

工具: 通过 tpcc-mysql 对整机性能进行测试

参数设置

*   文件系统: 均为 xfs
    
*   mysql 版本: mysql-8.0.12
    
*   tpccload:
    

*   warehouse: 100
    
*   sql: createtable.sql、addfkeyidx.sql
    
*   运行时长: 没有限制，跑完将近 1 小时
    

*   tpcc_start:
    

*   warehouse: 100
    
*   warmup: 300 秒
    
*   运行时长: 1800 秒
    

*   线程数: 16、32、64、128、256、512、1024
    

**> 图: oltp 测试 - tpmc**

![](https://fzxiaomange.com/img/el67/tpcc-mysql.png)

如何看图：越高越好

此图结论：

*   CentOS 7 比 CentOS 6（默认 ext4）高
    

### **5. 总结**

![](http://s2.51cto.com/images/20181027/1540644775401573.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**总结：7 个测试结果中，只有 2 项是基本一致，其余 5 项均是 CentOS 7 胜利，因此基本可以得出结论，CentOS 7 性能比 CentOS 6 更好！**

* * *

微信: 小慢哥 Linux 运维  

每周一文，轻松学 Linux 运维

![](https://fzxiaomange.com/weixin.jpg)

个人网站 [fzxiaomange.com](https://fzxiaomange.com/)

个人邮箱 [cyent@163.com](mailto:cyent@163.com)

举报文章

请选择举报类型

内容侵权 涉嫌营销 内容抄袭 违法信息 其他

上传截图

格式支持 JPEG/PNG/JPG，图片不超过 1.9M

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

已经收到您得举报信息，我们会尽快审核

相关文章
----