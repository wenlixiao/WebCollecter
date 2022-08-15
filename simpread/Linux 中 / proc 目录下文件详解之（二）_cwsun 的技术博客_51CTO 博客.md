> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/cw000/566954)

> Linux 中 / proc 目录下文件详解之（二），Linux 中 / proc 目录下文件详解之（二）Linux 中 / proc 目录下文件详解（二）[/ 声明：可以自由转载本文，但请务必保留本文的完整性。

Linux 中 / proc 目录下文件详解（二）[/  
声明：可以自由转载本文，但请务必保留本文的完整性。  
作者：张子坚  
email:zhangzijian@163.com  
说明：本文所涉及示例均在 fedora core3 下得到。  
-----------------------------------------------------------------------------------------------------  
/proc/mdstat 文件

这个文件包含了由 md 设备驱动程序控制的 RAID 设备信息。

示例：

[root@localhost ~]# cat /proc/mdstat  
Personalities :  
unused devices: <none>

--------------------------------------------------------------------------------

/proc/meminfo 文件

这个文件给出了内存状态的信息。它显示出系统中空闲内存，已用物理内存和交换内存的总量。它还显示出内核使用的共享内存和缓冲区总量。这些信息的格式和 free 命令显示的结果类似。

示例：

[root@localhost ~]# cat /proc/meminfo  
MemTotal:       223812 kB  
MemFree:          3764 kB  
Buffers:          9148 kB  
Cached:          92112 kB  
SwapCached:        364 kB  
Active:         183640 kB  
Inactive:        17196 kB  
HighTotal:           0 kB  
HighFree:            0 kB  
LowTotal:       223812 kB  
LowFree:          3764 kB  
SwapTotal:      626524 kB  
SwapFree:       620328 kB  
Dirty:              12 kB  
Writeback:           0 kB  
Mapped:         142880 kB  
Slab:            12668 kB  
Committed_AS:   376732 kB  
PageTables:       2336 kB  
VmallocTotal:  3907576 kB  
VmallocUsed:      2968 kB  
VmallocChunk:  3904224 kB  
HugePages_Total:     0  
HugePages_Free:      0  
Hugepagesize:     4096 kB

--------------------------------------------------------------------------------

/proc/misc 文件

这个文件报告用内核函数 misc_register 登记的设备驱动程序。

示例：

[root@localhost ~]# cat /proc/misc  
63 device-mapper  
175 agpgart  
135 rtc

--------------------------------------------------------------------------------

/proc/modules 文件

这个文件给出可加载内核模块的信息。lsmod 程序用这些信息显示有关模块的名称，大小，使用数目方面的信息。

示例：

[root@localhost /]# cat /proc/modules  
md5 4033 1 - Live 0x10a7f000  
ipv6 232577 8 - Live 0x10b0c000  
parport_pc 24705 1 - Live 0x10a8b000  
lp 11565 0 - Live 0x10a7b000  
parport 41737 2 parport_pc,lp, Live 0x10a55000  
autofs4 24005 0 - Live 0x10a74000  
i2c_dev 10433 0 - Live 0x109d2000  
i2c_core 22081 1 i2c_dev, Live 0x10a6d000  
sunrpc 160421 1 - Live 0x10a9d000  
ipt_REJECT 6465 1 - Live 0x109da000  
ipt_state 1857 5 - Live 0x109eb000  
ip_conntrack 40693 1 ipt_state, Live 0x10a62000  
iptable_filter 2753 1 - Live 0x10896000  
ip_tables 16193 3 ipt_REJECT,ipt_state,iptable_filter, Live 0x109ed000  
dm_mod 54741 0 - Live 0x109f8000  
button 6481 0 - Live 0x10905000  
battery 8517 0 - Live 0x109d6000  
ac 4805 0 - Live 0x10908000  
uhci_hcd 31449 0 - Live 0x109dd000  
ehci_hcd 31557 0 - Live 0x10949000  
snd_via82xx 27237 2 - Live 0x10953000  
snd_ac97_codec 64401 1 snd_via82xx, Live 0x10912000  
snd_pcm_oss 47609 0 - Live 0x1093c000  
snd_mixer_oss 17217 2 snd_pcm_oss, Live 0x1090c000  
snd_pcm 97993 2 snd_via82xx,snd_pcm_oss, Live 0x10923000  
snd_timer 29765 1 snd_pcm, Live 0x108ec000  
snd_page_alloc 9673 2 snd_via82xx,snd_pcm, Live 0x108bd000  
gameport 4801 1 snd_via82xx, Live 0x108a6000  
snd_mpu401_uart 8769 1 snd_via82xx, Live 0x108b9000  
snd_rawmidi 26725 1 snd_mpu401_uart, Live 0x108e4000  
snd_seq_device 8137 1 snd_rawmidi, Live 0x1083b000  
snd 54053 11 snd_via82xx,snd_ac97_codec,snd_pcm_oss,snd_mixer_oss,snd_pcm,snd_timer,snd_mpu401_uart,snd_rawmidi,snd_seq_device, Live 0x108f6000  
soundcore 9889 2 snd, Live 0x1089b000  
via_rhine 23497 0 - Live 0x1089f000  
mii 4673 1 via_rhine, Live 0x10893000  
floppy 58609 0 - Live 0x108a9000  
ext3 116809 1 - Live 0x10875000  
jbd 74969 1 ext3, Live 0x10861000

  
lsmod 命令显示结果如下：

[root@localhost /]# lsmod  
Module                  Size  Used by  
md5                     4033  1  
ipv6                  232577  8  
parport_pc             24705  1  
lp                     11565  0  
parport                41737  2 parport_pc,lp  
autofs4                24005  0  
i2c_dev                10433  0  
i2c_core               22081  1 i2c_dev  
sunrpc                160421  1  
ipt_REJECT              6465  1  
ipt_state               1857  5  
ip_conntrack           40693  1 ipt_state  
iptable_filter          2753  1  
ip_tables              16193  3 ipt_REJECT,ipt_state,iptable_filter  
dm_mod                 54741  0  
button                  6481  0  
battery                 8517  0  
ac                      4805  0  
uhci_hcd               31449  0  
ehci_hcd               31557  0  
snd_via82xx            27237  2  
snd_ac97_codec         64401  1 snd_via82xx  
snd_pcm_oss            47609  0  
snd_mixer_oss          17217  2 snd_pcm_oss  
snd_pcm                97993  2 snd_via82xx,snd_pcm_oss  
snd_timer              29765  1 snd_pcm  
snd_page_alloc          9673  2 snd_via82xx,snd_pcm  
gameport                4801  1 snd_via82xx  
snd_mpu401_uart         8769  1 snd_via82xx  
snd_rawmidi            26725  1 snd_mpu401_uart  
snd_seq_device          8137  1 snd_rawmidi  
snd                    54053  11 snd_via82xx,snd_ac97_codec,snd_pcm_oss,snd_mixer_oss,snd_pcm,snd_timer,snd_mpu401_uart,snd_rawmidi,snd_seq_device  
soundcore               9889  2 snd  
via_rhine              23497  0  
mii                     4673  1 via_rhine  
floppy                 58609  0  
ext3                  116809  1  
jbd                    74969  1 ext3

--------------------------------------------------------------------------------

/proc/mounts 文件

这个文件以 / etc/mtab 文件的格式给出当前系统所安装的文件系统信息。这个文件也能反映出任何手工安装从而在 / etc/mtab 文件中没有包含的文件系统。

示例：

[root@localhost /]# cat /proc/mounts  
rootfs / rootfs rw 0 0  
/proc /proc proc rw,nodiratime 0 0  
none /dev tmpfs rw 0 0  
/dev/root / ext3 rw 0 0  
none /dev tmpfs rw 0 0  
none /selinux selinuxfs rw 0 0  
/proc /proc proc rw,nodiratime 0 0  
/proc/bus/usb /proc/bus/usb usbfs rw 0 0  
/sys /sys sysfs rw 0 0  
none /dev/pts devpts rw 0 0  
none /dev/shm tmpfs rw 0 0  
none /proc/sys/fs/binfmt_misc binfmt_misc rw 0 0  
sunrpc /var/lib/nfs/rpc_pipefs rpc_pipefs rw 0 0

--------------------------------------------------------------------------------

/proc/pci 文件

这个文件给出 PCI 设备的信息。用它可以方便地诊断 PCI 问题。你可以从这个文件中检索到的信息包括诸如 IDE 接口或 USB 控制器这样的设备，总线，设备和功能编号，设备延迟以及 IRQ 编号。

示例：

[root@localhost /]# cat /proc/pci  
PCI devices found:  
  Bus  0, device   0, function  0:  
    Class 0600: PCI device 1106:3116 (rev 0).  
      Master Capable.  Latency=8.  
      Prefetchable 32 bit memory at 0xe0000000 [0xe7ffffff].  
  Bus  0, device   1, function  0:  
    Class 0604: PCI device 1106:b091 (rev 0).  
      Master Capable.  No bursts.  Min Gnt=12.  
  Bus  0, device  16, function  2:  
    Class 0c03: PCI device 1106:3038 (rev 12![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==).  
      IRQ 5.  
      Master Capable.  Latency=32.  
      I/O at 0xec00 [0xec1f].  
  Bus  0, device  16, function  1:  
    Class 0c03: PCI device 1106:3038 (rev 12![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==).  
      IRQ 3.  
      Master Capable.  Latency=32.  
      I/O at 0xe800 [0xe81f].  
  Bus  0, device  16, function  0:  
    Class 0c03: PCI device 1106:3038 (rev 12![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==).

      IRQ 11.  
      Master Capable.  Latency=32.  
      I/O at 0xe400 [0xe41f].  
  Bus  0, device  16, function  3:  
    Class 0c03: PCI device 1106:3104 (rev 130).  
      IRQ 10.  
      Master Capable.  Latency=32.  
      Non-prefetchable 32 bit memory at 0xdfffff00 [0xdfffffff].  
  Bus  0, device  17, function  0:  
    Class 0601: PCI device 1106:3177 (rev 0).  
  Bus  0, device  17, function  1:  
    Class 0101: PCI device 1106:0571 (rev 6).  
      IRQ 255.  
      Master Capable.  Latency=32.  
      I/O at 0xfc00 [0xfc0f].  
  Bus  0, device  17, function  5:  
    Class 0401: PCI device 1106:3059 (rev 80).  
      IRQ 5.  
      I/O at 0xe000 [0xe0ff].  
  Bus  0, device  18, function  0:  
    Class 0200: PCI device 1106:3065 (rev 116).  
      IRQ 11.  
      Master Capable.  Latency=32.  Min Gnt=3.Max Lat=8.  
      I/O at 0xdc00 [0xdcff].  
      Non-prefetchable 32 bit memory at 0xdffffe00 [0xdffffeff].  
  Bus  1, device   0, function  0:  
    Class 0300: PCI device 5333:8d04 (rev 0).  
      IRQ 11.  
      Master Capable.  Latency=32.  Min Gnt=4.Max Lat=255.  
      Non-prefetchable 32 bit memory at 0xdfe80000 [0xdfefffff].  
      Prefetchable 32 bit memory at 0xd0000000 [0xd7ffffff].

--------------------------------------------------------------------------------

/proc/stat 文件

这个文件包含的信息有 CPU 利用率，磁盘，内存页，内存对换，全部中断，接触开关以及赏赐自举时间（自 1970 年 1 月 1 日起的秒数）。

示例：

[root@localhost /]# cat /proc/stat  
cpu  31994 3898 7161 381600 15254 451 0  
cpu0 31994 3898 7161 381600 15254 451 0  
intr 4615930 4404290 3364 0 0 12 0 7 0 2 0 0 12618 112114 0 44142 39381  
ctxt 1310498  
btime 1148891913  
processes 4249  
procs_running 4  
procs_blocked 0

--------------------------------------------------------------------------------

  
/proc/uptime 文件

这个文件给出自从上次系统自举以来的秒数，以及其中有多少秒处于空闲。这主要供 uptime 程序使用。比较这两个数字能够告诉你长期来看 CPU 周期浪费的比例。

示例：

[root@localhost /]# cat /proc/uptime  
4477.04 4021.10

--------------------------------------------------------------------------------

/proc/version 文件

这个文件只有一行内容，说明正在运行的内核版本。可以用标准的编程方法进行分析获得所需的系统信息。

  
示例：

[root@localhost /]# cat /proc/version  
Linux version 2.6.9-1.667 ( [bhcompile@tweety.build.redhat.com](mailto:bhcompile@tweety.build.redhat.com)) (gcc version 3.4.2 20041017 (Red Hat 3.4.2-6.fc3)) #1 Tue Nov 2 14:41:25 EST 2004