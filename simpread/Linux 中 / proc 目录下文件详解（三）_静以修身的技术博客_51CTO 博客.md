> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/qinyinbolan/976096)

> Linux 中 / proc 目录下文件详解（三），Linux 中 / proc 目录下文件详解（三）/proc/net 子目录 & nbsp; 此目录下的文件描述或修改了联网代码的行为。

Linux 中 / proc 目录下文件详解（三）

/proc/net 子目录   
此目录下的文件描述或修改了联网代码的行为。可以通过使用 arp,netstat,route 和 ipfwadm 命令设置或查询这些特殊文件中的许多文件。  
示例：  
[root@localhost /]# ls /proc/net  
anycast6       ip_conntrack         mcfilter6  rt6_stats     tcp  
arp            ip_conntrack_expect  netlink    rt_acct       tcp6  
dev            ip_mr_cache          netstat    rt_cache      udp  
dev_mcast      ip_mr_vif            packet     snmp          udp6  
dev_snmp6      ip_tables_matches    psched     snmp6         unix  
if_inet6       ip_tables_names      raw        sockstat      wireless  
igmp           ip_tables_targets    raw6       sockstat6  
igmp6          ipv6_route           route      softnet_stat  
ip6_flowlabel  mcfilter             rpc        stat   
--------------------------------------------------------------------------------  
以下摘要介绍此目录下文件的功能：  
arp  
转储每个网络接口的 arp 表中 dev 包的统计  
dev  
来自网络设备的统计  
dev_mcast  
列出二层（数据链路层）多播组  
igmp  
加入的 IGMP 多播组  
netlink  
netlink 套接口的信息  
netstat  
网络流量的多种统计。第一行是信息头，带有每个变量的名称。接下来的一行保存相应变量的值  
raw  
原始套接口的套接口表  
route  
静态路由表  
rpc  
包含 RPC 信息的目录  
rt_cache  
路由缓冲  
snmp  
snmp agent 的 ip/icmp/tcp/udp 协议统计；各行交替给出字段名和值  
sockstat  
列出使用的 tcp/udp/raw/pac/syc_cookies 的数量  
tcp  
TCP 连接的套接口  
udp  
UDP 连接的套接口表  
unix  
UNIX 域套接口的套接口表   
--------------------------------------------------------------------------------  
示例：[root@localhost /]# cat /proc/net/route  
Iface   Destination     Gateway         Flags   RefCnt  Use     Metric  Mask   MTU      Window  IRTT  
eth0    0035C2DA        00000000        0001    0       0       0       80FFFFF0  
eth0    0000FEA9        00000000        0001    0       0       0       0000FFF0  
eth0    00000000        0135C2DA        0003    0       0       0       00000000   
--------------------------------------------------------------------------------  
[root@localhost /]# cat /proc/net/tcp  
  sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode  
   0: 00000000:8000 00000000:0000 0A 00000000:00000000 00:00000000 00000000    29        0 9525 1 0dde7500 3000 0 0 2 -1  
   1: 00000000:006F 00000000:0000 0A 00000000:00000000 00:00000000 00000000 0        0 9484 1 0dde79e0 3000 0 0 2 -1  
   2: 0100007F:0277 00000000:0000 0A 00000000:00000000 00:00000000 00000000 0        0 10049 1 0a8e3a00 3000 0 0 2 -1  
   3: 0100007F:14D7 00000000:0000 0A 00000000:00000000 00:00000000 00000000    99        0 9847 1 0dde7020 3000 0 0 2 -1  
   4: 0100007F:0019 00000000:0000 0A 00000000:00000000 00:00000000 00000000 0        0 10286 1 0a8e3520 3000 0 0 2 -1   
--------------------------------------------------------------------------------  
[root@localhost /]# cat /proc/net/arp  
IP address       HW type     Flags       HW address            Mask     Device  
218.194.53.1     0x1         0x2         00:0D:BC:78:07:3F     *        eth0   
--------------------------------------------------------------------------------  
[root@localhost /]# cat /proc/net/udp  
  sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode  
   0: 00000000:8000 00000000:0000 07 00000000:00000000 00:00000000 00000000    29        0 9520 2 0b4ef7c0  
105: 00000000:14E9 00000000:0000 07 00000000:00000000 00:00000000 00000000    99        0 10284 2 0b4ef040  
111: 00000000:006F 00000000:0000 07 00000000:00000000 00:00000000 00000000 0        0 9483 2 0b4efcc0  
116: 00000000:02F4 00000000:0000 07 00000000:00000000 00:00000000 00000000 0        0 9511 2 0b4efa40  
119: 00000000:0277 00000000:0000 07 00000000:00000000 00:00000000 00000000 0        0 10050 2 0b4ef2c0  
--------------------------------------------------------------------------------  
/proc/scsi 子目录  
此目录下包含一个列出了所有检测到的 SCSI 设备的文件，并且为每种控制器驱动程序提供一个目录，在这个目录下又为已安装的此种控制器的每个实例提供一个子目录。  
示例：  
由于本人的机器没有 SCSI 设备，顾暂时无法提供示例。   
--------------------------------------------------------------------------------  
/proc/sys 子目录  
在此目录下有许多子目录。此目录中的许多项都可以用来调整系统的性能。这个目录包含信息太多，无法介绍全部。只在示例中展示目录下的一些文件。  
示例：[root@localhost /]# ls /proc/sys  
debug  dev  fs  kernel  net  proc  sunrpc  vm   
--------------------------------------------------------------------------------  
[root@localhost ~]# ls /proc/sys/fs  
aio-max-nr   dentry-state       file-nr      lease-break-time  overflowgid  
aio-nr       dir-notify-enable  inode-nr     leases-enable     overflowuid  
binfmt_misc  file-max           inode-state  mqueue            quota   
--------------------------------------------------------------------------------  
[root@localhost ~]# ls /proc/sys/kernel  
acct                   hotplug      panic                   sem  
cad_pid                modprobe     panic_on_oops           shmall  
cap-bound              msgmax       pid_max                 shmmax  
core_pattern           msgmnb       print-fatal-signals     shmmni  
core_uses_pid          msgmni       printk                  sysrq  
ctrl-alt-del           ngroups_max  printk_ratelimit        tainted  
domainname             osrelease    printk_ratelimit_burst  threads-max  
exec-shield            ostype       pty                     vdso  
exec-shield-randomize  overflowgid  random                  version  
hostname               overflowuid  real-root-dev   
--------------------------------------------------------------------------------  
[root@localhost ~]# ls /proc/sys/net  
core  ethernet  ipv4  ipv6  unix   
--------------------------------------------------------------------------------  
[root@localhost sys]# ls /proc/sys/vm  
block_dump                 laptop_mode            nr_pdflush_threads  
dirty_background_ratio     legacy_va_layout       overcommit_memory  
dirty_expire_centisecs     lower_zone_protection  overcommit_ratio  
dirty_ratio                max_map_count          page-cluster  
dirty_writeback_centisecs  min_free_kbytes        swappiness  
hugetlb_shm_group          nr_hugepages           vfs_cache_pressure   
--------------------------------------------------------------------------------  
[root@localhost sys]# ls /proc/sys/net/ipv4  
conf                               tcp_fack  
icmp_echo_ignore_all               tcp_fin_timeout  
icmp_echo_ignore_broadcasts        tcp_frto  
icmp_ignore_bogus_error_responses  tcp_keepalive_intvl  
icmp_ratelimit                     tcp_keepalive_probes  
icmp_ratemask                      tcp_keepalive_time  
igmp_max_memberships               tcp_low_latency  
igmp_max_msf                       tcp_max_orphans  
inet_peer_gc_maxtime               tcp_max_syn_backlog  
inet_peer_gc_mintime               tcp_max_tw_buckets  
inet_peer_maxttl                   tcp_mem  
inet_peer_minttl                   tcp_moderate_rcvbuf  
inet_peer_threshold                tcp_no_metrics_save  
ip_autoconfig                      tcp_orphan_retries  
ip_conntrack_max                   tcp_reordering  
ip_default_ttl                     tcp_retrans_collapse  
ip_dynaddr                         tcp_retries1  
ip_forward                         tcp_retries2  
ipfrag_high_thresh                 tcp_rfc1337  
ipfrag_low_thresh                  tcp_rmem  
ipfrag_secret_interval             tcp_sack  
ipfrag_time                        tcp_stdurg  
ip_local_port_range                tcp_synack_retries  
ip_nonlocal_bind                   tcp_syncookies  
ip_no_pmtu_disc                    tcp_syn_retries  
neigh                              tcp_timestamps  
netfilter                          tcp_tso_win_divisor  
route                              tcp_tw_recycle  
tcp_abort_on_overflow              tcp_tw_reuse  
tcp_adv_win_scale                  tcp_vegas_alpha  
tcp_app_win                        tcp_vegas_beta  
tcp_bic                            tcp_vegas_cong_avoid  
tcp_bic_fast_convergence           tcp_vegas_gamma  
tcp_bic_low_window                 tcp_westwood  
tcp_dsack                          tcp_window_scaling  
tcp_ecn                            tcp_wmem   
--------------------------------------------------------------------------------  
[root@localhost sys]# cat /proc/sys/kernel/shmall  
2097152   
--------------------------------------------------------------------------------  
[root@localhost sys]# cat /proc/sys/kernel/osrelease  
2.6.9-1.667  
--------------------------------------------------------------------------------  
总结：/proc 文件系统包含了大量的有关当前系统状态的信息。proc 的手册页中也有对这些文件的解释文档。把文件和分析这些文件的工具产生的输出进行比较能够更加清晰地了解这些文件。  
 [本文出自 51CTO.COM 技术博客](http://qq164587043.blog.51cto.com/261469/48913)

**本文来自 ChinaUnix 博客，如果查看原文请点：** [http://blog.chinaunix.net/u/31/showart_602241.html](http://blog.chinaunix.net/u/31/showart_602241.html)

举报文章

请选择举报类型

内容侵权 涉嫌营销 内容抄袭 违法信息 其他

上传截图

格式支持 JPEG/PNG/JPG，图片不超过 1.9M

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

已经收到您得举报信息，我们会尽快审核

相关文章
----