> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/qinyinbolan/976101)

> Linux 中 / proc 目录下文件详解（一）， Linux 中 / proc 目录下文件详解（一）/proc 文件系统下的多种文件提供的系统信息不是针对某个特定进程的，而是能够在整个系统范围的上下文中使用。

 Linux 中 / proc 目录下文件详解（一）

**/proc** 文件系统下的多种文件提供的系统信息不是针对某个特定进程的，而是能够在整个系统范围的上下文中使用。可以使用的文件随系统配置的变化而变化。命令 procinfo 能够显示基于其中某些文件的多种系统信息。以下详细描述 / proc 下的文件。   
--------------------------------------------------------------------------------  
**/proc/cmdline 文件**  
这个文件给出了内核启动的命令行。它和用于进程的 cmdline 项非常相似。  
**示例：**   
[root@localhost proc]# cat cmdline  
ro root=LABEL=/ rhgb quiet  
--------------------------------------------------------------------------------  
**/proc/cpuinfo 文件**  
这个文件提供了有关系统 CPU 的多种信息。这些信息是从内核里对 CPU 的测试代码中得到的。文件列出了 CPU 的普通型号（386，486，586，686 等），以及能得到的更多特定信息（制造商，型号和版本）。文件还包含了以 bogomips 表示的处理器速度，而且如果检测到 CPU 的多种特性或者 bug，文件还会包含相应的标志。这个文件的格式为：文件由多行构成，每行包括一个域名称，一个冒号和一个值。  
**示例：**   
[root@localhost proc]# cat cpuinfo  
processor : 0  
vendor_id : AuthenticAMD  
cpu family : 6  
model : 8  
model name : AMD Athlon(tm) XP 1800+  
stepping : 1  
cpu MHz : 1530.165  
cache size : 256 KB  
fdiv_bug : no  
hlt_bug : no  
f00f_bug : no  
coma_bug : no  
fpu : yes  
fpu_exception : yes  
cpuid level : 1  
wp : yes  
flags : fpu vme de pse tsc msr pae mce cx8 apic mtrr pge mca cmov pat pse36 mmx fxsr sse syscall mmxext 3dnowext 3dnow  
bogomips : 2998.27  
--------------------------------------------------------------------------------  
**/proc/devices 文件**  
这个文件列出字符和块设备的主设备号，以及分配到这些设备号的设备名称。  
**示例：**   
[root@localhost /]# cat /proc/devices  
Character devices:  
1 mem  
4 /dev/vc/0  
4 tty  
4 ttyS  
5 /dev/tty  
5 /dev/console  
5 /dev/ptmx  
6 lp  
7 vcs  
10 misc  
13 input  
14 sound  
29 fb  
36 netlink  
116 alsa  
128 ptm  
136 pts  
180 usb  
Block devices:  
1 ramdisk  
2 fd  
3 ide0  
9 md  
22 ide1  
253 device-mapper  
254 mdp   
--------------------------------------------------------------------------------  
/proc/dma 文件   
这个文件列出由驱动程序保留的 DMA 通道和保留它们的驱动程序名称。casade 项供用于把次 DMA 控制器从主控制器分出的 DMA 行所使用；这一行不能用于其它用途。  
示例：  
[root@localhost ~]# cat /proc/dma  
4: cascade  
--------------------------------------------------------------------------------  
/proc/filesystems 文件  
这个文件列出可供使用的文件系统类型，一种类型一行。虽然它们通常是编入内核的文件系统类型，但该文件还可以包含可加载的内核模块加入的其它文件系统类型。  
示例：  
[root@localhost proc]# cat /proc/filesystems   
nodev sysfs  
nodev rootfs  
nodev bdev  
nodev proc  
nodev sockfs  
nodev binfmt_misc  
nodev usbfs  
nodev usbdevfs  
nodev futexfs  
nodev tmpfs  
nodev pipefs  
nodev eventpollfs  
nodev devpts  
ext2  
nodev ramfs  
nodev hugetlbfs  
iso9660  
nodev mqueue  
nodev selinuxfs  
ext3  
nodev rpc_pipefs  
nodev autofs   
--------------------------------------------------------------------------------  
/proc/interrupts 文件  
这个文件的每一行都有一个保留的中断。每行中的域有：中断号，本行中断的发生次数，可能带有一个加号的域（SA_INTERRUPT 标志设置），以及登记这个中断的驱动程序的名字。可以在安装新硬件前，像查看 / proc/dma 和 / proc/ioports 一样用 cat 命令手工查看手头的这个文件。这几个文件列出了当前投入使用的资源（但是不包括那些没有加载驱动程序的硬件所使用的资源）。  
示例：  
[root@localhost SPECS]# cat /proc/interrupts  
CPU0  
0: 7039406 XT-PIC timer  
1: 6533 XT-PIC i8042  
2: 0 XT-PIC cascade  
3: 0 XT-PIC uhci_hcd  
5: 108 XT-PIC VIA8233, uhci_hcd  
8: 1 XT-PIC rtc  
9: 0 XT-PIC acpi  
10: 0 XT-PIC ehci_hcd  
11: 17412 XT-PIC uhci_hcd, eth0  
12: 140314 XT-PIC i8042  
14: 37897 XT-PIC ide0  
15: 60813 XT-PIC ide1  
NMI: 0  
ERR: 1   
--------------------------------------------------------------------------------  
/proc/ioports 文件  
这个文件列出了诸如磁盘驱动器，以太网卡和声卡设备等多种设备驱动程序登记的许多 I/O 端口范围。  
示例：  
[root@localhost SPECS]# cat /proc/ioports  
0000-001f : dma1  
0020-0021 : pic1  
0040-0043 : timer0  
0050-0053 : timer1  
0060-006f : keyboard  
0070-0077 : rtc  
0080-008f : dma page reg  
00a0-00a1 : pic2  
00c0-00df : dma2  
00f0-00ff : fpu  
0170-0177 : ide1  
01f0-01f7 : ide0  
0376-0376 : ide1  
0378-037a : parport0  
037b-037f : parport0  
03c0-03df : vga+  
03f6-03f6 : ide0  
03f8-03ff : serial  
0800-0803 : PM1a_EVT_BLK  
0804-0805 : PM1a_CNT_BLK  
0808-080b : PM_TMR  
0810-0815 : ACPI CPU throttle  
0820-0823 : GPE0_BLK  
0cf8-0cff : PCI conf1  
dc00-dcff : 0000:00:12.0  
dc00-dcff : via-rhine  
e000-e0ff : 0000:00:11.5  
e000-e0ff : VIA8233  
e400-e41f : 0000:00:10.0  
e400-e41f : uhci_hcd  
e800-e81f : 0000:00:10.1  
e800-e81f : uhci_hcd  
ec00-ec1f : 0000:00:10.2  
ec00-ec1f : uhci_hcd  
fc00-fc0f : 0000:00:11.1  
fc00-fc07 : ide0  
fc08-fc0f : ide1   
--------------------------------------------------------------------------------  
/proc/kcore 文件  
这个文件是系统的物理内存以 core 文件格式保存的文件。例如，GDB 能用它考察内核的数据结构。它不是纯文本，而是 / proc 目录下为数不多的几个二进制格式的项之一。  
示例：  
暂无   
--------------------------------------------------------------------------------  
/proc/kmsg 文件  
这个文件用于检索用 printk 生成的内核消息。任何时刻只能有一个具有超级用户权限的进程可以读取这个文件。也可以用系统调用 syslog 检索这些消息。通常使用工具 dmesg 或守护进程 klogd 检索这些消息。  
示例：  
暂无   
--------------------------------------------------------------------------------  
/proc/ksyms 文件  
这个文件列出了已经登记的内核符号；这些符号给出了变量或函数的地址。每行给出一个符号的地址，符号名称以及登记这个符号的模块。程序 ksyms,insmod 和 kmod 使用这个文件。它还列出了正在运行的任务数，总任务数和最后分配的 PID。  
示例：  
暂无   
--------------------------------------------------------------------------------  
/proc/loadavg 文件  
这个文件给出以几个不同的时间间隔计算的系统平均负载，这就如同 uptime 命令显示的结果那样。前三个数字是平均负载。这是通过计算过去 1 分钟，5 分钟，15 分钟里运行队列中的平均任务数得到的。随后是正在运行的任务数和总任务数。最后是上次使用的进程号。  
示例：  
[root@localhost ~]# cat /proc/loadavg  
0.11 0.16 0.14 3/126 3912   
--------------------------------------------------------------------------------  
/proc/locks 文件  
这个文件包含在打开的文件上的加锁信息。文件中的每一行描述了特定文件和文档上的加锁信息以及对文件施加的锁的类型。内核也可以需要时对文件施加强制性锁。  
示例：  
[root@localhost redhat]# cat /proc/locks  
1: POSIX ADVISORY READ 3822 03:0a:1067117 0 EOF  
2: POSIX ADVISORY READ 3822 03:0a:1067138 0 EOF  
3: POSIX ADVISORY WRITE 3326 03:0a:2326540 0 EOF  
4: POSIX ADVISORY WRITE 2639 03:0a:2966595 0 EOF  
5: FLOCK ADVISORY WRITE 2591 03:0a:2966586 0 EOF  
6: POSIX ADVISORY WRITE 2540 03:0a:2966578 0 EOF  
7: POSIX ADVISORY WRITE 2530 03:0a:2966579 0 EOF  
8: POSIX ADVISORY WRITE 2402 03:0a:2966563 0 EOF  
9: POSIX ADVISORY WRITE 2371 03:0a:2966561 0 EOF   
--------------------------------------------------------------------------------  
 [本文出自 51CTO.COM 技术博客](http://qq164587043.blog.51cto.com/261469/48911)

**本文来自 ChinaUnix 博客，如果查看原文请点：** [http://blog.chinaunix.net/u/31/showart_602243.html](http://blog.chinaunix.net/u/31/showart_602243.html)

举报文章

请选择举报类型

内容侵权 涉嫌营销 内容抄袭 违法信息 其他

上传截图

格式支持 JPEG/PNG/JPG，图片不超过 1.9M

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

已经收到您得举报信息，我们会尽快审核

相关文章
----