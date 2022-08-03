> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/hguisu/article/details/102620787)

之前文章[《Linux 服务器性能评估与优化 (一)》](https://blog.csdn.net/hguisu/article/details/39373311)太长，阅读不方便，因此拆分成系列博文：

[《](https://blog.csdn.net/hguisu/article/details/39373311)[Linux 服务器性能评估与优化 (一)--CPU](https://blog.csdn.net/hguisu/article/details/39373311)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/39373311)[Linux 服务器性能评估与优化 (二)-- 内存](https://blog.csdn.net/hguisu/article/details/102620787)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/102620903)[Linux 服务器性能评估与优化 (三)-- 磁盘 i/o](https://blog.csdn.net/hguisu/article/details/102620903)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/102620981)[Linux 服务器性能评估与优化 (四)-- 网络](https://blog.csdn.net/hguisu/article/details/39373311)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/39249775)[Linux 服务器性能评估与优化（五)-- 内核参数](https://blog.csdn.net/hguisu/article/details/39249775)[》](https://blog.csdn.net/hguisu/article/details/39373311)

我们通过 top 或者 ps -aux 查看应用实际占用的[内存](https://so.csdn.net/so/search?q=%E5%86%85%E5%AD%98&spm=1001.2101.3001.7020)和虚拟内存：

![](https://img-blog.csdnimg.cn/20210821164756687.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hndWlzdQ==,size_16,color_FFFFFF,t_70)

RSS（ Resident Set Size ）常驻内存集合大小，表示相应进程在 RAM 中占用了多少内存，并不包含在 SWAP 中占用的[虚拟内存](https://so.csdn.net/so/search?q=%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98&spm=1001.2101.3001.7020)。即使是在内存中的使用了共享库的内存大小也一并计算在内，包含了完整的在 stack 和 heap 中的内存。

VSZ （Virtual Memory Size)，表明是虚拟内存大小，表明了该进程可以访问的所有内存，包括被交换的内存和共享库内存。

1、虚拟内存简介
--------

       虚拟内存是使用磁盘作为 RAM 的扩充使得可用内存的有效大小得到相应增加。 内核会将当前内存中未被使用的块的内容写入硬盘以此来腾出内存空间。 当上面的内容再次需要被用到时，它们将被重新读入内存。这些对用户完全透明；在 linux 下运行的程序只会看到有大量内存空间可用而不会去管它们的一部分时不时的从硬盘读取。 当然， 硬盘的读写操作速度比内存慢上千倍， 所以这个程序的运行速度也不会很快。 这个被用作虚拟内存的硬盘空间  
呗称作交换空间（swap space） 。

     （电脑中所运行的程序均需经由内存执行，若执行的程序很大或很多，则会导致内存消耗殆尽。为解决该问题，Windows 中运用了虚拟内存技术，即匀出一部分硬盘空间来充当内存使用。当内存耗尽时，电脑就会自动调用硬盘来充当内存，以缓解内存的紧张。是计算机系统内存管理的一种技术。它使得应用程序认为它拥有连续的可用的内存，而实际上，它常是被分隔成多个物理内存碎片，还有部分暂存储于外部磁盘存储器上，在需要时进行数据交换。）

### 1.1 linux 虚拟内存页

   对 Linux 系统而言，虚拟内存就是 swap 分区。Linux 虚拟内存被分成页，在 X86 架构下的每个虚拟内存页大小为 4KB。  
#/usr/bin/time -v date  
 ...  
Page size (bytes): 4096  
     当内核由内存向磁盘读写数据时，是以页为单位。内核会把内存页写入 swap 空间和文件系统。  
盘读写数据时，是以页为单位。内核会把内存页写入 swap 空间和文件系统。

      每一个进程启动时都会向系统申请虚拟内存（VSZ），内核同意或者拒就请求。当程序真正用到内存时，系统就它映射到物理内存。RSS 表示程序所占的物理内存的大小。用 ps 命令我们可以看到进程占用的 VSZ 和 RSS。

![](https://img-blog.csdnimg.cn/20210821164754162.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hndWlzdQ==,size_16,color_FFFFFF,t_70)

### 1.2 内核内存分页:Memory Paging

       内存分页是一个常见的动作， 不能和内存交换相混淆。 内存分页是内核会定期将内存中的数据同步到硬盘的过程。 随着时间的增长， 应用会不断耗尽内存， 有时候内核必须扫描内存并回收被分配给其他应用的空闲页面。

       同时内核也要负责回收不用的内存，将他们分**给其他**需要的进程。页帧回收算法（PFRA）算法（Page Frame reclaim algorithm）负责回收空闲的内存。算法根据内存页的类型来决定要释放的内存页。有下列 4 种类型：

1． 不可被回收（Unreclaimable） –locked，kernel，reserved 被锁、内核、保留页；  
2． 可交换（Swappable） –无名内存页；  
3． 可同步 (Syncable) –通过硬盘文件备份的内存页；  
4． 可丢弃 (Discardable) –静态页和废弃页。

除了第一种（Unreclaimable）之外其余的都可以被 PFRA 进行回收。与 PFRA 相关是内核进程是 kswapd。

### 1.3 kswapd 负责执行页面回收 PFRA

kswapd 守护进程负责确保内存保持可用空闲空间。它监测内核中的 pages_high 和 pages_low 标记，如果空闲内存空间值小于 pages_low 值，kswwapd 进程开始扫描并尝试每次回收 32 个 free pages 页面，如此重复直至空闲内存页 free pages 空间大于 pages_high 值。  
kswapd 进程执行如下操作：  
1、如果页面未改变，它将该页面放入空闲队列 free list。  
2、如果页面发生改变且可备份回文件系统的，它将页面内容写回磁盘。  
3、如果页面发生改变且未被文件系统回写（无名页：没有任何磁盘上的备份） ，它将页面内容写入 swap 分区。

![](https://img-blog.csdnimg.cn/20191107140537569.png)

在回收内存过程中还有两个重要的方法：  
1、LMR（Low on memory reclaiming）。  
2、OOM killer(Out of Memory Killer) 机制。

> **LMR：**当分配内存失败的时候 LMR 将会其作用，失败的原因是 kswapd 不能提供足够的空闲内存，这个时候 LMR 会每次释放 1024 个垃圾页知道内存分配成功。

> **OOM killer：**当 LMR 不能快速释放内存的时候，**OOM killer** 就开始其作用，**OOM killer** 会采用一个选择算法来决定杀死某些进程。当选定进程时，就会发送信号 SIGKILL，这就会使内存立即被释放。**OOM killer** 选择进程的方法如下：  
> 1.   进程占用大量的内存；  
> 2.   进程只会损失少量工作；  
> 3.   进程具有低的静态优先级；  
> 4.   进程不属于 root 用户。
> 
> 简单说，Linux 内核 OOM killer 机制监控那些占用内存过大，尤其是瞬间占用内存很快的进程，然后防止内存耗尽而自动把该进程杀掉。
> 
> 内核 OOM killer 机如何选择进程参考内核源代码 linux/mm/oom_kill.c，当系统内存不足的时候，out_of_memory() 被触发，然后调用 select_bad_process() 选择一个”bad” 进程杀掉。linux 内核判断和选择一个”bad 进程是通过调用 oom_badness() 方法，挑选的算法如上。

如果系统出现 OOM Killer，可以查看系统日志：grep  Kill /var/log/messages  可以看到类似日志：

Nov  27 13:29:54 kernel: Out of memory: Kill process 12246 (java) score 77 or sacrifice child

这种情况，一般是 java 进程触发了 oom killer，既 java 进程要申请的内存大于了系统可用的物理内存大小。/proc/sys/vm/min_free_kbytes 参数来控制，当系统可用内存（不包含 buffer 和 cache）小于这个值的时候，系统会启动内核线程 kswapd 来对内存进行回收。而还是触发了 oom killer，则表明内存真的不够用了或者在内存回收前或者回收中直接触发了 oom killer。

### 1.4、pdflush 内核分页

内核 pdflush 守护进程负责同步所有与文件系统相关的页面至磁盘，换句话说，就是当一个文件在内存中发生改变，pdflush 守护进程就将其回写进磁盘。

# ps -ef | grep pdflush

root 28 3 0 23:01 ? 00:00:00 [pdflush]

root 29 3 0 23:01 ? 00:00:00 [pdflush]

每当内存中脏页面（dirty page）数量超过 10% 的时候，pdflush 守护进程就会开始将这些页面备份写回硬盘。这个动作与内核的 vm.dirty_background_ratio 参数值有关。  
# sysctl -n vm.dirty_background_ratio 10

多数情况下 pdflush 守护进程与 PFRA 之间是相互独立的。除了常规的回收动作之外，当内核调用 LMR 算法时，LMR 就触发 pdflush 将脏页面写回硬盘。

**2、利用 free 指令监控内存**
--------------------

free 是监控 linux 内存使用状况最常用的指令，看下面的一个输出：

[root@webserver ~]# free  -m

               total         used       free     shared    buffers     cached

Mem:       8111       7185        926          0       243           6299

-/+ buffers/cache:     643       7468

Swap:       8189          0         8189

        一般有这样一个经验公式：应用程序可用内存 / 系统物理内存 > 70% 时，表示系统内存资源非常充足，不影响系统性能，应用程序可用内存 / 系统物理内存 < 20% 时，表示系统内存资源紧缺，需要增加系统内存，20%< 应用程序可用内存 / 系统物理内存 < 70% 时，表示系统内存资源基本能满足应用需求，暂时不影响系统性能。

       对于应用程序来说，buffers/cached 是等于可用的，因为 buffer/cached 是为了提高文件读取的性能，当应用程序需在用到内存的时候，buffer/cached 会很快地被回收。所以从应用程序的角度来说，可用内存 = 系统 free memory+buffers+cached.

       可以执行：echo 1 > /proc/sys/vm/drop_caches: 表示清除 page cache。

       了解具体 buffers/cache，参考另外一篇文章[《Linux 了解内存使用》](https://guisu.blog.csdn.net/article/details/7403855)

    ** Buffers 和 Cached 的区别：**

     buffers 是为块设备设计的缓冲。比如磁盘读写，把分散的写操作集中进行，减少磁盘 I/O，从而提高系统性能。比如入 U 盘里 cp 一个文件，但是 U 盘读写指示灯未闪动，过了一会儿才闪动。卸载时会清空缓冲，所以有时卸载一个设备需要等待几秒。

    cached 是缓存读取过的内容，下次再读时，如果在缓存中命中，则直接从缓存读取，否则读取磁盘。由于缓存空间有限，过一段时间以后没用的缓存会被移动到 swap 里面，所以有时看到物理内存还有很多，swap 就被利用了。

**3、利用 vmstat 命令监控内存**
----------------------

vmstat 命令除了报告 CPU 的情况外还能查看虚拟内存的使用情况，vmstat 输出的以下区域与虚拟内存有关

[root@node1 ~]# vmstat 2 3

procs -----------memory----------  ---swap--  -----io---- --system--  -----cpu------

 r  b   swpd   free      buff  cache   si   so    bi    bo       in     cs     us sy  id  wa st

 0  0    0    162240   8304  67032   0    0    13    21   1007   23     0  1  98   0  0

 0  0    0    162240   8304  67032   0    0     1     0     1010   20     0  1  100 0  0

 0  0    0    162240   8304  67032   0    0     1     1     1009   18     0  1  99   0  0

 **memory**

         swpd 列表示切换到内存交换区的内存数量（以 k 为单位）。如果 swpd 的值不为 0，或者比较大，只要 si、so 的值长期为 0，这种情况下一般不用担心，不会影响系统性能。

         free 列表示当前空闲的物理内存数量（以 k 为单位）

         buff 列表示 buffers cache 的内存数量，一般对块设备的读写才需要缓冲。

         cache 列表示 page cached 的内存数量，一般作为文件系统 cached，频繁访问的文件都会被 cached，如果 cache 值较大，说明 cached 的文件数较多，如果此时 IO 中 bi 比较小，说明文件系统效率比较好。

**swap**

si 列表示由磁盘调入内存，也就是内存进入内存交换区的数量。

so 列表示由内存调入磁盘，也就是内存交换区进入内存的数量。

一般情况下，si、so 的值都为 0，如果 si、so 的值长期不为 0，则表示系统内存不足。需要增加系统内存。

例如大规模 IO 写入：

vmstat  3  
procs memory swap io system cpu  
 r  b swpd    free          buff       cache  si   so bi     bo    in    cs  us sy id wa  
3 2 809192 261556 79760 886880 416 0 8244 751 426 863 17 3 6 75  
0 3 809188 194916 79820 952900 307 0 21745 1005 1189 2590 34 6 12 48  
0 3 809188 162212 79840 988920 95 0 12107 0 1801 2633 2 2 3 94  
1 3 809268 88756 79924 1061424 260 28 18377 113 1142 1694 3 5 3 88  
1 2 826284 17608 71240 1144180 100 6140 25839 16380 1528 1179 19 9 12 61  
2 1 854780 17688 34140 1208980 1 9535 25557 30967 1764 2238 43 13 16 28  
0 8 867528 17588 32332 1226392 31 4384 16524 27808 1490 1634 41 10 7 43  
4 2 877372 17596 32372 1227532 213 3281 10912 3337 678 932 33 7 3 57  
1 2 885980 17800 32408 1239160 204 2892 12347 12681 1033 982 40 12 2 46  
5 2 900472 17980 32440 1253884 24 4851 17521 4856 934 1730 48 12 13 26  
1 1 904404 17620 32492 1258928 15 1316 7647 15804 919 978 49 9 17 25  
4 1 911192 17944 32540 1266724 37 2263 12907 3547 834 1421 47 14 20 20  
1 1 919292 17876 31824 1275832 1 2745 16327 2747 617 1421 52 11 23 14  
5 0 925216 17812 25008 1289320 12 1975 12760 3181 772 1254 50 10 21 19  
0 5 932860 17736 21760 1300280 8 2556 15469 3873 825 1258 49 13 24 15  
通过以上数据可得出以下结果：

*   大量的数据从磁盘读入内存（bi） ，因 cache 值在不断增长
*   尽管数据正不断消耗内存，空闲空间仍保持在 17M 左右
*   buff 值正不断减少，说明 kswapd 正不断回收内存
*   swpd 值不断增大，说明 kswapd 正将脏页面内容写入交换空间（so）

总结下来:

1. 当一个系统有越少的页错误 (所需数据不在内存, 需要从硬盘读入), 就会有越好的响应时间. 因为内存比硬盘快得多.

2. Free memory 数量低是一个好的征兆, 因为证明了 cache 在起作用, 除非同时存在大量的 bi 和 so.

3. 如果一个系统有持续的 si 和 so, 就说明系统的内存是一个瓶颈.

**4、TOP 命令 按内存占用排序和按 CPU 占用排序**
-------------------------------

1：在命令行提示符执行 top 命令

2：输入大写 P，则结果按 CPU 占用降序排序。输入大写 M，结果按内存占用降序排序。（注：大写 P 可以在 capslock 状态输入 p，或者按 Shift+p）

**小结：**虚拟内存的性能监测包括以下步骤：

1.  当系统利用内存缓存超过磁盘缓存，系统反应速度更快
2.  除在有大量持续的交换空间和磁盘读入动作情况下外， 空闲内存空间很少说明 cache 得到了有效的利用
3.  如果系统报告有持续的交换空间使用，说明内存不足