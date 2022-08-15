> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/DswCnblog/p/5780389.html)

Linux 系统上的 / proc 目录是一种文件系统，即 proc 文件系统。与其它常见的文件系统不同的是，/proc 是一种伪文件系统（也即虚拟文件系统），存储的是当前内核运行状态的一系列特殊文件，用户可以通过这些文件查看有关系统硬件及当前正在运行进程的信息，甚至可以通过更改其中某些文件来改变内核的运行状态。   
基于 / proc 文件系统如上所述的特殊性，其内的文件也常被称作虚拟文件，并具有一些独特的特点。例如，其中有些文件虽然使用查看命令查看时会返回大量信息，但文件本身的大小却会显示为 0 字节。此外，这些特殊文件中大多数文件的时间及日期属性通常为当前系统时间和日期，这跟它们随时会被刷新（存储于 RAM 中）有关。   
为了查看及使用上的方便，这些文件通常会按照相关性进行分类存储于不同的目录甚至子目录中，如 / proc/scsi 目录中存储的就是当前系统上所有 SCSI 设备的相关信息，/proc/N 中存储的则是系统当前正在运行的进程的相关信息，其中 N 为正在运行的进程（可以想象得到，在某进程结束后其相关目录则会消失）。   
大多数虚拟文件可以使用文件查看命令如 cat、more 或者 less 进行查看，有些文件信息表述的内容可以一目了然，但也有文件的信息却不怎么具有可读性。不过，这些可读性较差的文件在使用一些命令如 apm、free、lspci 或 top 查看时却可以有着不错的表现。  
一、进程目录中的常见文件介绍   
/proc 目录中包含许多以数字命名的子目录，这些数字表示系统当前正在运行进程的进程号，里面包含对应进程相关的多个信息文件。   

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# ll /proc<br>total 0<br>dr-xr-xr-x&nbsp;&nbsp;5 root&nbsp; &nbsp;&nbsp; &nbsp;root&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;0 Feb&nbsp;&nbsp;8 17:08 1<br>dr-xr-xr-x&nbsp;&nbsp;5 root&nbsp; &nbsp;&nbsp; &nbsp;root&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;0 Feb&nbsp;&nbsp;8 17:08 10<br>dr-xr-xr-x&nbsp;&nbsp;5 root&nbsp; &nbsp;&nbsp; &nbsp;root&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;0 Feb&nbsp;&nbsp;8 17:08 11<br>dr-xr-xr-x&nbsp;&nbsp;5 root&nbsp; &nbsp;&nbsp; &nbsp;root&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;0 Feb&nbsp;&nbsp;8 17:08 1156<br>dr-xr-xr-x&nbsp;&nbsp;5 root&nbsp; &nbsp;&nbsp; &nbsp;root&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;0 Feb&nbsp;&nbsp;8 17:08 139<br>dr-xr-xr-x&nbsp;&nbsp;5 root&nbsp; &nbsp;&nbsp; &nbsp;root&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;0 Feb&nbsp;&nbsp;8 17:08 140<br>dr-xr-xr-x&nbsp;&nbsp;5 root&nbsp; &nbsp;&nbsp; &nbsp;root&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;0 Feb&nbsp;&nbsp;8 17:08 141<br>dr-xr-xr-x&nbsp;&nbsp;5 root&nbsp; &nbsp;&nbsp; &nbsp;root&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;0 Feb&nbsp;&nbsp;8 17:09 1417<br>dr-xr-xr-x&nbsp;&nbsp;5 root&nbsp; &nbsp;&nbsp; &nbsp;root&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;0 Feb&nbsp;&nbsp;8 17:09 1418</td></tr></tbody></table>

  
上面列出的是 / proc 目录中一些进程相关的目录，每个目录中是当程本身相关信息的文件。下面是作者系统（RHEL5.3）上运行的一个 PID 为 2674 的进程 saslauthd 的相关文件，其中有些文件是每个进程都会具有的，后文会对这些常见文件做出说明。 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# ll /proc/2674<br>total 0<br>dr-xr-xr-x 2 root root 0 Feb&nbsp;&nbsp;8 17:15 attr<br>-r-------- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 auxv<br>-r--r--r-- 1 root root 0 Feb&nbsp;&nbsp;8 17:09 cmdline<br>-rw-r--r-- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 coredump_filter<br>-r--r--r-- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 cpuset<br>lrwxrwxrwx 1 root root 0 Feb&nbsp;&nbsp;8 17:14 cwd -&gt; /var/run/saslauthd<br>-r-------- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 environ<br>lrwxrwxrwx 1 root root 0 Feb&nbsp;&nbsp;8 17:09 exe -&gt; /usr/sbin/saslauthd<br>dr-x------ 2 root root 0 Feb&nbsp;&nbsp;8 17:15 fd<br>-r-------- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 limits<br>-rw-r--r-- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 loginuid<br>-r--r--r-- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 maps<br>-rw------- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 mem<br>-r--r--r-- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 mounts<br>-r-------- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 mountstats<br>-rw-r--r-- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 oom_adj<br>-r--r--r-- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 oom_score<br>lrwxrwxrwx 1 root root 0 Feb&nbsp;&nbsp;8 17:14 root -&gt; /<br>-r--r--r-- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 schedstat<br>-r-------- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 smaps<br>-r--r--r-- 1 root root 0 Feb&nbsp;&nbsp;8 17:09 stat<br>-r--r--r-- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 statm<br>-r--r--r-- 1 root root 0 Feb&nbsp;&nbsp;8 17:10 status<br>dr-xr-xr-x 3 root root 0 Feb&nbsp;&nbsp;8 17:15 task<br>-r--r--r-- 1 root root 0 Feb&nbsp;&nbsp;8 17:14 wchan</td></tr></tbody></table>

  
1.1、cmdline — 启动当前进程的完整命令，但僵尸进程目录中的此文件不包含任何信息； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/2674/cmdline&nbsp;<br>/usr/sbin/saslauthd</td></tr></tbody></table>

1.2、cwd — 指向当前进程运行目录的一个符号链接；   
1.3、environ — 当前进程的环境变量列表，彼此间用空字符（NULL）隔开；变量用大写字母表示，其值用小写字母表示； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/2674/environ&nbsp;<br>TERM=linuxauthd</td></tr></tbody></table>

  
1.4、exe — 指向启动当前进程的可执行文件（完整路径）的符号链接，通过 / proc/N/exe 可以启动当前进程的一个拷贝；   
1.5、fd — 这是个目录，包含当前进程打开的每一个文件的文件描述符（file descriptor），这些文件描述符是指向实际文件的一个符号链接； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# ll /proc/2674/fd<br>total 0<br>lrwx------ 1 root root 64 Feb&nbsp;&nbsp;8 17:17 0 -&gt; /dev/null<br>lrwx------ 1 root root 64 Feb&nbsp;&nbsp;8 17:17 1 -&gt; /dev/null<br>lrwx------ 1 root root 64 Feb&nbsp;&nbsp;8 17:17 2 -&gt; /dev/null<br>lrwx------ 1 root root 64 Feb&nbsp;&nbsp;8 17:17 3 -&gt; socket:[7990]<br>lrwx------ 1 root root 64 Feb&nbsp;&nbsp;8 17:17 4 -&gt; /var/run/saslauthd/saslauthd.pid<br>lrwx------ 1 root root 64 Feb&nbsp;&nbsp;8 17:17 5 -&gt; socket:[7991]<br>lrwx------ 1 root root 64 Feb&nbsp;&nbsp;8 17:17 6 -&gt; /var/run/saslauthd/mux.accept</td></tr></tbody></table>

  
1.6、limits — 当前进程所使用的每一个受限资源的软限制、硬限制和管理单元；此文件仅可由实际启动当前进程的 UID 用户读取；（2.6.24 以后的内核版本支持此功能）；   
1.7、maps — 当前进程关联到的每个可执行文件和库文件在内存中的映射区域及其访问权限所组成的列表； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# cat /proc/2674/maps&nbsp;<br>00110000-00239000 r-xp 00000000 08:02 130647&nbsp; &nbsp;&nbsp;&nbsp;/lib/libcrypto.so.0.9.8e<br>00239000-0024c000 rwxp 00129000 08:02 130647&nbsp; &nbsp;&nbsp;&nbsp;/lib/libcrypto.so.0.9.8e<br>0024c000-00250000 rwxp 0024c000 00:00 0&nbsp;<br>00250000-00252000 r-xp 00000000 08:02 130462&nbsp; &nbsp;&nbsp;&nbsp;/lib/libdl-2.5.so<br>00252000-00253000 r-xp 00001000 08:02 130462&nbsp; &nbsp;&nbsp;&nbsp;/lib/libdl-2.5.so</td></tr></tbody></table>

  
1.8、mem — 当前进程所占用的内存空间，由 open、read 和 lseek 等系统调用使用，不能被用户读取；   
1.9、root — 指向当前进程运行根目录的符号链接；在 Unix 和 Linux 系统上，通常采用 chroot 命令使每个进程运行于独立的根目录；   
1.10、stat — 当前进程的状态信息，包含一系统格式化后的数据列，可读性差，通常由 ps 命令使用；   
1.11、statm — 当前进程占用内存的状态信息，通常以 “页面”（page）表示；   
1.12、status — 与 stat 所提供信息类似，但可读性较好，如下所示，每行表示一个属性信息；其详细介绍请参见 proc 的 man 手册页； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/2674/status&nbsp;<br>Name:&nbsp; &nbsp;saslauthd<br>State:&nbsp;&nbsp;S (sleeping)<br>SleepAVG:&nbsp; &nbsp;&nbsp; &nbsp; 0%<br>Tgid:&nbsp; &nbsp;2674<br>Pid:&nbsp; &nbsp; 2674<br>PPid:&nbsp; &nbsp;1<br>TracerPid:&nbsp; &nbsp;&nbsp; &nbsp;0<br>Uid:&nbsp; &nbsp; 0&nbsp; &nbsp;&nbsp; &nbsp; 0&nbsp; &nbsp;&nbsp; &nbsp; 0&nbsp; &nbsp;&nbsp; &nbsp; 0<br>Gid:&nbsp; &nbsp; 0&nbsp; &nbsp;&nbsp; &nbsp; 0&nbsp; &nbsp;&nbsp; &nbsp; 0&nbsp; &nbsp;&nbsp; &nbsp; 0<br>FDSize: 32<br>Groups:<br>VmPeak:&nbsp; &nbsp;&nbsp;&nbsp;5576 kB<br>VmSize:&nbsp; &nbsp;&nbsp;&nbsp;5572 kB<br>VmLck:&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;0 kB<br>VmHWM:&nbsp; &nbsp;&nbsp; &nbsp; 696 kB<br>VmRSS:&nbsp; &nbsp;&nbsp; &nbsp; 696 kB<br>…………</td></tr></tbody></table>

  
1.13、task — 目录文件，包含由当前进程所运行的每一个线程的相关信息，每个线程的相关信息文件均保存在一个由线程号（tid）命名的目录中，这类似于其内容类似于每个进程目录中的内容；（内核 2.6 版本以后支持此功能）   
二、/proc 目录下常见的文件介绍   
2.1、/proc/apm   
高级电源管理（APM）版本信息及电池相关状态信息，通常由 apm 命令使用；   
2.2、/proc/buddyinfo   
用于诊断内存碎片问题的相关信息文件；   
2.3、/proc/cmdline   
在启动时传递至内核的相关参数信息，这些信息通常由 lilo 或 grub 等启动管理工具进行传递； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/cmdline&nbsp;<br>ro root=/dev/VolGroup00/LogVol00 rhgb quiet</td></tr></tbody></table>

  
2.4、/proc/cpuinfo   
处理器的相关信息的文件；   
2.5、/proc/crypto   
系统上已安装的内核使用的密码算法及每个算法的详细信息列表； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/crypto&nbsp;<br>name&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;: crc32c<br>driver&nbsp; &nbsp;&nbsp; &nbsp; : crc32c-generic<br>module&nbsp; &nbsp;&nbsp; &nbsp; : kernel<br>priority&nbsp; &nbsp;&nbsp;&nbsp;: 0<br>type&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;: digest<br>blocksize&nbsp; &nbsp; : 32<br>digestsize&nbsp; &nbsp;: 4<br>…………</td></tr></tbody></table>

  
2.6、/proc/devices   
系统已经加载的所有块设备和字符设备的信息，包含主设备号和设备组（与主设备号对应的设备类型）名； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/devices&nbsp;<br>Character devices:<br>&nbsp;&nbsp;1 mem<br>&nbsp;&nbsp;4 /dev/vc/0<br>&nbsp;&nbsp;4 tty<br>&nbsp;&nbsp;4 ttyS<br>&nbsp;&nbsp;…………<br>Block devices:<br>&nbsp;&nbsp;1 ramdisk<br>&nbsp;&nbsp;2 fd<br>&nbsp;&nbsp;8 sd<br>&nbsp;&nbsp;…………</td></tr></tbody></table>

  
2.7、/proc/diskstats   
每块磁盘设备的磁盘 I/O 统计信息列表；（内核 2.5.69 以后的版本支持此功能）   
2.8、/proc/dma   
每个正在使用且注册的 ISA DMA 通道的信息列表； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/dma<br>2: floppy<br>4: cascade</td></tr></tbody></table>

  
2.9、/proc/execdomains   
内核当前支持的执行域（每种操作系统独特 “个性”）信息列表； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/execdomains&nbsp;<br>0-0&nbsp; &nbsp;&nbsp;&nbsp;Linux&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; [kernel]</td></tr></tbody></table>

  
2.10、/proc/fb   
帧缓冲设备列表文件，包含帧缓冲设备的设备号和相关驱动信息；   
2.11、/proc/filesystems   
当前被内核支持的文件系统类型列表文件，被标示为 nodev 的文件系统表示不需要块设备的支持；通常 mount 一个设备时，如果没有指定文件系统类型将通过此文件来决定其所需文件系统的类型； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/filesystems&nbsp;<br>nodev&nbsp; &nbsp;sysfs<br>nodev&nbsp; &nbsp;rootfs<br>nodev&nbsp; &nbsp;proc<br>&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;iso9660<br>&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;ext3<br>…………<br>…………</td></tr></tbody></table>

  
2.12、/proc/interrupts   
X86 或 X86_64 体系架构系统上每个 IRQ 相关的中断号列表；多路处理器平台上每个 CPU 对于每个 I/O 设备均有自己的中断号； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/interrupts&nbsp;<br>&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;CPU0&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;<br>&nbsp;&nbsp;0:&nbsp; &nbsp; 1305421&nbsp; &nbsp; IO-APIC-edge&nbsp;&nbsp;timer<br>&nbsp;&nbsp;1:&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;61&nbsp; &nbsp; IO-APIC-edge&nbsp;&nbsp;i8042<br>185:&nbsp; &nbsp;&nbsp; &nbsp; 1068&nbsp; &nbsp;IO-APIC-level&nbsp;&nbsp;eth0<br>…………</td></tr></tbody></table>

  
2.13、/proc/iomem   
每个物理设备上的记忆体（RAM 或者 ROM）在系统内存中的映射信息； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/iomem&nbsp;<br>00000000-0009f7ff : System RAM<br>0009f800-0009ffff : reserved<br>000a0000-000bffff : Video RAM area<br>000c0000-000c7fff : Video ROM<br>&nbsp;&nbsp;…………</td></tr></tbody></table>

  
2.14、/proc/ioports   
当前正在使用且已经注册过的与物理设备进行通讯的输入 - 输出端口范围信息列表；如下面所示，第一列表示注册的 I/O 端口范围，其后表示相关的设备； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# less /proc/ioports&nbsp;<br>0000-001f : dma1<br>0020-0021 : pic1<br>0040-0043 : timer0<br>0050-0053 : timer1<br>0060-006f : keyboard<br>…………</td></tr></tbody></table>

  
2.15、/proc/kallsyms   
模块管理工具用来动态链接或绑定可装载模块的符号定义，由内核输出；（内核 2.5.71 以后的版本支持此功能）；通常这个文件中的信息量相当大； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/kallsyms&nbsp;<br>c04011f0 T _stext<br>c04011f0 t run_init_process<br>c04011f0 T stext<br>&nbsp;&nbsp;…………</td></tr></tbody></table>

  
2.16、/proc/kcore   
系统使用的物理内存，以 ELF 核心文件（core file）格式存储，其文件大小为已使用的物理内存（RAM）加上 4KB；这个文件用来检查内核数据结构的当前状态，因此，通常由 GBD 通常调试工具使用，但不能使用文件查看命令打开此文件；   
2.17、/proc/kmsg   
此文件用来保存由内核输出的信息，通常由 / sbin/klogd 或 / bin/dmsg 等程序使用，不要试图使用查看命令打开此文件；   
2.18、/proc/loadavg   
保存关于 CPU 和磁盘 I/O 的负载平均值，其前三列分别表示每 1 秒钟、每 5 秒钟及每 15 秒的负载平均值，类似于 uptime 命令输出的相关信息；第四列是由斜线隔开的两个数值，前者表示当前正由内核调度的实体（进程和线程）的数目，后者表示系统当前存活的内核调度实体的数目；第五列表示此文件被查看前最近一个由内核创建的进程的 PID； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/loadavg&nbsp;<br>0.45 0.12 0.04 4/125 5549<br>[root@rhel5 ~]# uptime<br>06:00:54 up&nbsp;&nbsp;1:06,&nbsp;&nbsp;3 users,&nbsp;&nbsp;load average: 0.45, 0.12, 0.04</td></tr></tbody></table>

  
2.19、/proc/locks   
保存当前由内核锁定的文件的相关信息，包含内核内部的调试数据；每个锁定占据一行，且具有一个惟一的编号；如下输出信息中每行的第二列表示当前锁定使用的锁定类别，POSIX 表示目前较新类型的文件锁，由 lockf 系统调用产生，FLOCK 是传统的 UNIX 文件锁，由 flock 系统调用产生；第三列也通常由两种类型，ADVISORY 表示不允许其他用户锁定此文件，但允许读取，MANDATORY 表示此文件锁定期间不允许其他用户任何形式的访问； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/locks&nbsp;<br>1: POSIX&nbsp;&nbsp;ADVISORY&nbsp;&nbsp;WRITE 4904 fd:00:4325393 0 EOF<br>2: POSIX&nbsp;&nbsp;ADVISORY&nbsp;&nbsp;WRITE 4550 fd:00:2066539 0 EOF<br>3: FLOCK&nbsp;&nbsp;ADVISORY&nbsp;&nbsp;WRITE 4497 fd:00:2066533 0 EOF</td></tr></tbody></table>

  
2.20、/proc/mdstat   
保存 RAID 相关的多块磁盘的当前状态信息，在没有使用 RAID 机器上，其显示为如下状态： 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# less /proc/mdstat&nbsp;<br>Personalities :&nbsp;<br>unused devices: &lt;none&gt;</td></tr></tbody></table>

  
2.21、/proc/meminfo   
系统中关于当前内存的利用状况等的信息，常由 free 命令使用；可以使用文件查看命令直接读取此文件，其内容显示为两列，前者为统计属性，后者为对应的值； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# less /proc/meminfo&nbsp;<br>MemTotal:&nbsp; &nbsp;&nbsp; &nbsp; 515492 kB<br>MemFree:&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; 8452 kB<br>Buffers:&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;19724 kB<br>Cached:&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;376400 kB<br>SwapCached:&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; 4 kB<br>…………</td></tr></tbody></table>

  
2.22、/proc/mounts   
在内核 2.4.29 版本以前，此文件的内容为系统当前挂载的所有文件系统，在 2.4.19 以后的内核中引进了每个进程使用独立挂载名称空间的方式，此文件则随之变成了指向 / proc/self/mounts（每个进程自身挂载名称空间中的所有挂载点列表）文件的符号链接；/proc/self 是一个独特的目录，后文中会对此目录进行介绍； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# ll /proc |grep mounts<br>lrwxrwxrwx&nbsp;&nbsp;1 root&nbsp; &nbsp;&nbsp; &nbsp;root&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; 11 Feb&nbsp;&nbsp;8 06:43 mounts -&gt; self/mounts</td></tr></tbody></table>

  
如下所示，其中第一列表示挂载的设备，第二列表示在当前目录树中的挂载点，第三点表示当前文件系统的类型，第四列表示挂载属性（ro 或者 rw），第五列和第六列用来匹配 / etc/mtab 文件中的转储（dump）属性； 

<table cellspacing="0"><tbody><tr><td><br>[root@rhel5 ~]# more /proc/mounts&nbsp;<br>rootfs / rootfs rw 0 0<br>/dev/root / ext3 rw,data=ordered 0 0<br>/dev /dev tmpfs rw 0 0<br>/proc /proc proc rw 0 0<br>/sys /sys sysfs rw 0 0<br>/proc/bus/usb /proc/bus/usb usbfs rw 0 0<br>…………</td></tr></tbody></table>

  
2.23、/proc/modules   
当前装入内核的所有模块名称列表，可以由 lsmod 命令使用，也可以直接查看；如下所示，其中第一列表示模块名，第二列表示此模块占用内存空间大小，第三列表示此模块有多少实例被装入，第四列表示此模块依赖于其它哪些模块，第五列表示此模块的装载状态（Live：已经装入；Loading：正在装入；Unloading：正在卸载），第六列表示此模块在内核内存（kernel memory）中的偏移量； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/modules&nbsp;<br>autofs4 24517 2 - Live 0xe09f7000<br>hidp 23105 2 - Live 0xe0a06000<br>rfcomm 42457 0 - Live 0xe0ab3000<br>l2cap 29505 10 hidp,rfcomm, Live 0xe0aaa000<br>…………</td></tr></tbody></table>

  
2.24、/proc/partitions   
块设备每个分区的主设备号（major）和次设备号（minor）等信息，同时包括每个分区所包含的块（block）数目（如下面输出中第三列所示）； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/partitions&nbsp;<br>major minor&nbsp;&nbsp;#blocks&nbsp;&nbsp;name<br>&nbsp; &nbsp;8&nbsp; &nbsp;&nbsp;&nbsp;0&nbsp; &nbsp;20971520 sda<br>&nbsp; &nbsp;8&nbsp; &nbsp;&nbsp;&nbsp;1&nbsp; &nbsp;&nbsp;&nbsp;104391 sda1<br>&nbsp; &nbsp;8&nbsp; &nbsp;&nbsp;&nbsp;2&nbsp; &nbsp; 6907950 sda2<br>&nbsp; &nbsp;8&nbsp; &nbsp;&nbsp;&nbsp;3&nbsp; &nbsp; 5630782 sda3<br>&nbsp; &nbsp;8&nbsp; &nbsp;&nbsp;&nbsp;4&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; 1 sda4<br>&nbsp; &nbsp;8&nbsp; &nbsp;&nbsp;&nbsp;5&nbsp; &nbsp; 3582463 sda5</td></tr></tbody></table>

  
2.25、/proc/pci   
内核初始化时发现的所有 PCI 设备及其配置信息列表，其配置信息多为某 PCI 设备相关 IRQ 信息，可读性不高，可以用 “/sbin/lspci –vb” 命令获得较易理解的相关信息；在 2.6 内核以后，此文件已为 / proc/bus/pci 目录及其下的文件代替；   
2.26、/proc/slabinfo   
在内核中频繁使用的对象（如 inode、dentry 等）都有自己的 cache，即 slab pool，而 / proc/slabinfo 文件列出了这些对象相关 slap 的信息；详情可以参见内核文档中 slapinfo 的手册页； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/slabinfo&nbsp;<br>slabinfo - version: 2.1<br># name&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&lt;active_objs&gt; &lt;num_objs&gt; &lt;objsize&gt; &lt;objperslab&gt; &lt;pagesperslab&gt; : tunables &lt;limit&gt; &lt;batchcount&gt; &lt;sharedfactor&gt; : slabdata &lt;ac<br>tive_slabs&gt; &lt;num_slabs&gt; &lt;sharedavail&gt;<br>rpc_buffers&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;8&nbsp; &nbsp;&nbsp; &nbsp;8&nbsp; &nbsp;2048&nbsp; &nbsp; 2&nbsp; &nbsp; 1 : tunables&nbsp; &nbsp;24&nbsp; &nbsp;12&nbsp; &nbsp; 8 : slabdata&nbsp; &nbsp;&nbsp; &nbsp;4&nbsp; &nbsp;&nbsp; &nbsp;4&nbsp; &nbsp;&nbsp; &nbsp;0<br>rpc_tasks&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;8&nbsp; &nbsp;&nbsp;&nbsp;20&nbsp; &nbsp; 192&nbsp; &nbsp;20&nbsp; &nbsp; 1 : tunables&nbsp;&nbsp;120&nbsp; &nbsp;60&nbsp; &nbsp; 8 : slabdata&nbsp; &nbsp;&nbsp; &nbsp;1&nbsp; &nbsp;&nbsp; &nbsp;1&nbsp; &nbsp;&nbsp; &nbsp;0<br>rpc_inode_cache&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;6&nbsp; &nbsp;&nbsp; &nbsp;9&nbsp; &nbsp; 448&nbsp; &nbsp; 9&nbsp; &nbsp; 1 : tunables&nbsp; &nbsp;54&nbsp; &nbsp;27&nbsp; &nbsp; 8 : slabdata&nbsp; &nbsp;&nbsp; &nbsp;1&nbsp; &nbsp;&nbsp; &nbsp;1&nbsp; &nbsp;&nbsp; &nbsp;0<br>…………<br>…………<br>…………</td></tr></tbody></table>

  
2.27、/proc/stat   
实时追踪自系统上次启动以来的多种统计信息；如下所示，其中，   
“cpu” 行后的八个值分别表示以 1/100（jiffies）秒为单位的统计值（包括系统运行于用户模式、低优先级用户模式，运系统模式、空闲模式、I/O 等待模式的时间等）；   
“intr” 行给出中断的信息，第一个为自系统启动以来，发生的所有的中断的次数；然后每个数对应一个特定的中断自系统启动以来所发生的次数；   
“ctxt” 给出了自系统启动以来 CPU 发生的上下文交换的次数。   
“btime” 给出了从系统启动到现在为止的时间，单位为秒；   
“processes (total_forks) 自系统启动以来所创建的任务的个数目；   
“procs_running”：当前运行队列的任务的数目；   
“procs_blocked”：当前被阻塞的任务的数目； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/stat<br>cpu&nbsp;&nbsp;2751 26 5771 266413 2555 99 411 0<br>cpu0 2751 26 5771 266413 2555 99 411 0<br>intr 2810179 2780489 67 0 3 3 0 5 0 1 0 0 0 1707 0 0 9620 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0&nbsp;<br>0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0&nbsp;<br>0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 5504 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 12781 0 0 0<br>0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0<br>ctxt 427300<br>btime 1234084100<br>processes 3491<br>procs_running 1<br>procs_blocked 0</td></tr></tbody></table>

  
2.28、/proc/swaps   
当前系统上的交换分区及其空间利用信息，如果有多个交换分区的话，则会每个交换分区的信息分别存储于 / proc/swap 目录中的单独文件中，而其优先级数字越低，被使用到的可能性越大；下面是作者系统中只有一个交换分区时的输出信息； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/swaps&nbsp;<br>Filename&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;Type&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;Size&nbsp; &nbsp; Used&nbsp; &nbsp; Priority<br>/dev/sda8&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; partition&nbsp; &nbsp;&nbsp; &nbsp; 642560&nbsp;&nbsp;0&nbsp; &nbsp;&nbsp; &nbsp; -1</td></tr></tbody></table>

  
2.29、/proc/uptime   
系统上次启动以来的运行时间，如下所示，其第一个数字表示系统运行时间，第二个数字表示系统空闲时间，单位是秒； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/uptime&nbsp;<br>3809.86 3714.13</td></tr></tbody></table>

  
2.30、/proc/version   
当前系统运行的内核版本号，在作者的 RHEL5.3 上还会显示系统安装的 gcc 版本，如下所示； 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/version&nbsp;<br>Linux version 2.6.18-128.el5 (<a href="mailto:mockbuild@hs20-bc1-5.build.redhat.com" rel="noopener">mockbuild@hs20-bc1-5.build.redhat.com</a>) (gcc version 4.1.2 20080704 (Red Hat 4.1.2-44)) #1 SMP Wed Dec 17 11:42:39 EST 2008</td></tr></tbody></table>

  
2.31、/proc/vmstat   
当前系统虚拟内存的多种统计数据，信息量可能会比较大，这因系统而有所不同，可读性较好；下面为作者机器上输出信息的一个片段；（2.6 以后的内核支持此文件） 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/vmstat&nbsp;<br>nr_anon_pages 22270<br>nr_mapped 8542<br>nr_file_pages 47706<br>nr_slab 4720<br>nr_page_table_pages 897<br>nr_dirty 21<br>nr_writeback 0<br>…………</td></tr></tbody></table>

  
2.32、/proc/zoneinfo   
内存区域（zone）的详细信息列表，信息量较大，下面列出的是一个输出片段： 

<table cellspacing="0"><tbody><tr><td>[root@rhel5 ~]# more /proc/zoneinfo&nbsp;<br>Node 0, zone&nbsp; &nbsp;&nbsp; &nbsp;DMA<br>&nbsp;&nbsp;pages free&nbsp; &nbsp;&nbsp;&nbsp;1208<br>&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;min&nbsp; &nbsp;&nbsp; &nbsp;28<br>&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;low&nbsp; &nbsp;&nbsp; &nbsp;35<br>&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;high&nbsp; &nbsp;&nbsp;&nbsp;42<br>&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;active&nbsp; &nbsp;439<br>&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;inactive 1139<br>&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;scanned&nbsp;&nbsp;0 (a: 7 i: 30)<br>&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;spanned&nbsp;&nbsp;4096<br>&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;present&nbsp;&nbsp;4096<br>&nbsp; &nbsp; nr_anon_pages 192<br>&nbsp; &nbsp; nr_mapped&nbsp; &nbsp; 141<br>&nbsp; &nbsp; nr_file_pages 1385<br>&nbsp; &nbsp; nr_slab&nbsp; &nbsp;&nbsp; &nbsp;253<br>&nbsp; &nbsp; nr_page_table_pages 2<br>&nbsp; &nbsp; nr_dirty&nbsp; &nbsp;&nbsp;&nbsp;523<br>&nbsp; &nbsp; nr_writeback 0<br>&nbsp; &nbsp; nr_unstable&nbsp;&nbsp;0<br>&nbsp; &nbsp; nr_bounce&nbsp; &nbsp; 0<br>&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;protection: (0, 0, 296, 296)<br>&nbsp;&nbsp;pagesets<br>&nbsp;&nbsp;all_unreclaimable: 0<br>&nbsp;&nbsp;prev_priority:&nbsp; &nbsp;&nbsp;&nbsp;12<br>&nbsp;&nbsp;start_pfn:&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;0<br>…………</td></tr></tbody></table>

  
三、/proc/sys 目录详解   
与 / proc 下其它文件的 “只读” 属性不同的是，管理员可对 / proc/sys 子目录中的许多文件内容进行修改以更改内核的运行特性，事先可以使用 “ls -l” 命令查看某文件是否 “可写入”。写入操作通常使用类似于“echo  DATA > /path/to/your/filename” 的格式进行。需要注意的是，即使文件可写，其一般也不可以使用编辑器进行编辑。   
3.1、/proc/sys/debug 子目录   
此目录通常是一空目录；   
3.2、/proc/sys/dev 子目录 

为系统上特殊设备提供参数信息文件的目录，其不同设备的信息文件分别存储于不同的子目录中，如大多数系统上都会具有的 / proc/sys/dev/cdrom 和 / proc/sys/dev/raid（如果内核编译时开启了支持 raid 的功能） 目录，其内存储的通常是系统上 cdrom 和 raid 的相关参数信息文件。