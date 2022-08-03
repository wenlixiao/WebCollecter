> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/hguisu/article/details/102620903)

之前文章[《Linux 服务器性能评估与优化 (一)》](https://blog.csdn.net/hguisu/article/details/39373311)太长，阅读不方便，因此拆分成系列博文：

[《](https://blog.csdn.net/hguisu/article/details/39373311)[Linux 服务器性能评估与优化 (一)--CPU](https://blog.csdn.net/hguisu/article/details/39373311)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/39373311)[Linux 服务器性能评估与优化 (二)-- 内存](https://blog.csdn.net/hguisu/article/details/102620787)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/102620903)[Linux 服务器性能评估与优化 (三)-- 磁盘 i/o](https://blog.csdn.net/hguisu/article/details/102620903)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/102620981)[Linux 服务器性能评估与优化 (四)-- 网络](https://blog.csdn.net/hguisu/article/details/39373311)[》](https://blog.csdn.net/hguisu/article/details/39373311)

[《](https://blog.csdn.net/hguisu/article/details/39249775)[Linux 服务器性能评估与优化（五)-- 内核参数](https://blog.csdn.net/hguisu/article/details/39249775)[》](https://blog.csdn.net/hguisu/article/details/39373311)

前言、磁盘基础知识
---------

彻底了解磁盘的基础知识：

1、硬件层磁盘工作原理：[《磁盘读写原理》](https://guisu.blog.csdn.net/article/details/7408047)[https://guisu.blog.csdn.net/article/details/7408047](https://guisu.blog.csdn.net/article/details/7408047)

2、软件层文件系统详解：[《linux 文件系统详解 》https://guisu.blog.csdn.net/article/details/7401963](https://guisu.blog.csdn.net/article/details/7401963)

### 1、按存储的设备分类

磁盘可以是支持华存储的设备，根据存储介质的不同，常见磁盘可以分为机械磁盘，固态磁盘

**1）. 机械磁盘：**也成为硬盘驱动器 (Hard Disk Driver)，通常缩写为 HDD，机械磁盘主要由盘片和读写磁头组成，数据就存储在盘片的环状磁道中，在读写数据前，需要移动读写磁头，定位到数据所在的磁道，然后才能访问数据。  
      如果 I/O 请求刚好连续，就不需要磁道寻址，自然可以获得最佳性能，这其实就是我们熟悉的，连续 I/O 工作原理，与会对应的就是随机 I/O，它需要不停的移动磁头，来定位数据位置，所以读写速度就比较慢。

>        机械磁盘转速 (Rotational Speed)：是普通硬盘内电机主轴的旋转速度，也就是硬盘盘片在一分钟内所能完成的最大转数。转速的快慢是标示硬盘档次的重要参数之一，它是决定硬盘内部传输率的关键因素之一，在很大程度上直接影响到硬盘的速度。硬盘的转速越快，硬盘寻找文件的速度也就越快，相对的硬盘的传输速度也就得到了提高。
> 
>        硬盘转速以每分钟多少转来表示，单位表示为 RPM，RPM 是 Revolutions per minute 的缩写，是转 / 每分钟。RPM 值越大，内部传输率就越快，访问时间就越短，硬盘的整体性能也就越好。硬盘的主轴马达带动盘片高速旋转，产生浮力使磁头飘浮在盘片上方。要将所要存取资料的[扇区](https://baike.baidu.com/item/%E6%89%87%E5%8C%BA)带到磁头下方，转速越快，则等待时间也就越短。因此转速在很大程度上决定了硬盘的速度。
> 
>         家用的普通硬盘的转速一般有 5400rpm、7200rpm 几种，高转速硬盘也是现在台式机用户的首选；而对于笔记本用户，5400rpm、7200rpm 的[笔记本硬盘](https://baike.baidu.com/item/%E7%AC%94%E8%AE%B0%E6%9C%AC%E7%A1%AC%E7%9B%98)，在市场中都可见到，但装配 7200rpm 的笔记本价格要高出 300 元左右；服务器用户对硬盘性能要求最高，**服务器中使用的 SCSI 硬盘转速基本都采用 10000rpm，甚至还有 15000rpm 的，性能要超出家用产品很多。**

**2）. 固态硬盘 (Solid State Disk)，**通常缩写为 SSD，由固态电子元器件组成，固态磁盘不需要磁道寻址，所以不管是连续 I/O，还是随机 I/O 的性能，都比机械磁盘要好得多.

**两者比较：**无论机械盘还是固态磁盘，相同磁盘的随机 I/O 都要比连续 I/O 慢很多

    对机械磁盘来说，由于随机 I/O 需要更多的磁头寻道和盘片旋转，它的性能自然比连续 I/O 慢  
    对于固态磁盘，虽然它的随机性能比机械硬盘好很多，但同样存在 “先擦除再写入” 的限制，随机读写会导致大量的垃圾回收，导致随机 I/O 的性能比连续 I/O 还是差了很多  
     连续 I/O 还可以通过预读取的方式，来减少 I/O 请求的次数，这也是其性能优异的一个原因，很多性能优化的方案，都会从这个角度出发，来优化 I/O 性能

机械磁盘和固态硬盘还有一个最小的读写单位

    机械磁盘的最小读写单位是扇区，一般为 512 字节  
    固态硬盘的最小读写单位是页，通常大小是 4KB，8KB 等

由于每次读写 512 字节这么小的单位效率很低，所以文件系统会把连续的扇区，组成逻辑块，然后以逻辑快作为最小单位来管理数据，常见的逻辑块大小是 4KB，也就是连续 8 个扇区，或者一个独立的页，都可以组成一个逻辑块。

**查看 linux 磁盘是 ssd 还是 SATA 盘：**

命令：`cat /sys/block/sda/queue/rotational  #`sba 是你的磁盘名称，可以通过 df 命令查看磁盘

> 结果：
> 
> 返回 0：SSD 盘
> 
> 返回 1：SATA 盘

### 2、按存储的接口分类

除了按照存储介质分类，另一种常见的分类方法，是按照接口来分类，比如把硬盘分为

    IDE(Integrated Drive Electronics)，  
    SCSI(Small Computer System Interface)，  
    SAS(Serial Attached SCSI)，  
    SATA(Serial ATA)，  
    FC(Fibre Channel)

不同的接口，往往分配不同的设备名称：比如 IDE 设备会分配一个 hd 前缀的设备名，SCSI 和 SATA 设备会分配一个 sd 前缀的设备名，或者是多块同类型的磁盘，就会按照 a，b，长凳的字母顺序来编号  
除了磁盘本身的分类外，当把磁盘介入服务器后，按照不同的使用方法，又可以把他们划分为多种不同的架构  
最简单的，就是直接作为独立磁盘设备来使用，这些磁盘往往还会根据需要，划分为不同的逻辑分区，每个分区再用数字编号，比如  
/dev/sda，还可以分成两个分区  
/dev/sda1/，/dev/sda2

### 3、磁盘阵列

另一个比较常用的架构，是把多块磁盘组合成一个逻辑磁盘，构成冗余独立磁盘阵列，也就是 RAID(Redundant Array of Independent Disks)，从而可以提高数据访问性能，并且增强数据存储的可靠性。

根据容量，性能和可靠性的不同，RAID 一般分为多个级别，RAIDO，RAID1，RAID5，RAID10 等。RAID0 有最优的读写性能，但不能提供数据冗余的功能，而其他级别的 RAID，在提供数据冗余的基础上，对读写性能也有一定程度的优化。  
![](https://img-blog.csdnimg.cn/20191107153407337.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ndWlzdS5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70)

**一、磁盘存储基础**
------------

        磁盘 IO 子系统是 linux 系统里最慢的部分，这是由于其与 CPU 相比相去甚远，且依赖于物理式工作（转动和检索） 。如果将读取磁盘和读取[内存](https://so.csdn.net/so/search?q=%E5%86%85%E5%AD%98&spm=1001.2101.3001.7020)所花费的时间转换为分秒，那么他们之间的差距是 7 天和 7 分钟，所以 linux 内核尽量少的进行磁盘操作是至关重要的。以下部分将描述下内核处理数据从磁盘到内存和从内存到磁盘的不同方式。

### **1、数据读写——内存页面**

linux 内核将磁盘 IO 分为页面进行操作， 大多数 linux 系统中默认页面大小为 4K， 即以 4K 为单位进行磁盘和内存间的读写操作。我们可以使用 time 命令来查找页面大小：  
# /usr/bin/time -v date  
......  
Page size (bytes): 4096

### **2、缺页中断**

 **缺页中断分为主缺页中断 MPF（Major Page Faults）和缺页中断 MnPF（Minor PageFaults）**  
        linux 和大多数 UNIX 系统一样，使用虚拟内存层来映射物理地址空间，这种映射在某种意义上是说使得原来物理内存不能容下的程序也可以通过内存和硬盘之间的不断交换（把暂时不用的内存页交换到硬盘，把需要的内存页从硬盘读到内存）来赢得更多的内存，看起来就像物理内存被扩大了一样。事实上这个过程对程序是完全透明的，程序完全不用理会自己哪一部分、什么时候被交换进内存，一切都有内核的虚拟内存管理来完成。

      当一个进程开始运行，内核仅仅映射其需要的那部分，内核首先会搜索 CPU 缓存和物理内，如果数据已经在内存里就忽略，如果数据不在内存里就引起一个缺页中断（Page Fault），然后从硬盘读取缺页，并把缺页缓存到物理内存里。

主缺页中断（Major Page Fault）：要从磁盘读取数据而产生的中断是主缺页中断；

次缺页中断（Minor Page Fault）：数据已经被读入内存并被缓存起来，从内存缓存区中而不是直接从硬盘中读取数据而产生的中断是次缺页中断。

通过 time 命令显示了当一个程序启动的时候产生了多少 MPF 和 MnPF，在第一次启动的时候产生了很多 MPF：  
# /usr/bin/time -v evolution  
......  
Major (requiring I/O) page faults: 163  
Minor (reclaiming a frame) page faults: 5918  
......  
第二次启动的时候 MPF 消失因为程序已经在内存中：  
# /usr/bin/time -v evolution  
......  
Major (requiring I/O) page faults: 0  
Minor (reclaiming a frame) page faults: 5581  
......

### **3、文件缓冲区 File Buffer Cache**

从**文件缓冲区**（也叫**文件缓存区** File Buffer Cache）读取页比从硬盘读取页要快得多，所以 Linux 内核希望能尽可能产生次缺页中断（从文件缓冲区读），并且能尽可能避免主缺页中断（从硬盘读）.

随着系统不断地 IO 操作, 这样随着次缺页中断的增多，文件缓冲区也逐步增大，直至内存空闲空间不足 linux 才开始回收释放一些不用的页。我们运行 Linux 一段时间后会发现虽然系统上运行的程序不多，但是可用内存总是很少，这样给大家造成了 Linux 对内存管理很低效的假象，事实上 Linux 把那些暂时不用的物理内存高效的利用起来做预存（内存缓存区）呢。

我们在 centos 系统查看物理内存和文件缓存区的情况：

![](https://img-blog.csdnimg.cn/20191107164634608.png)

详细信息可以查看 / proc/meminfo 文件：  
![](https://img-blog.csdnimg.cn/20191107164718723.png)  
......  
以上显示系统拥有差不多 8G 内存， 当前有 120M 可用内存（free）， 120M 的 buffer 供应磁盘写操作缓存， 2.9GB 的 cache 由磁盘读入内存缓存区。可见 Linux 真的用了很多物理内存做 Cache，而且这个缓存区还可以不断增长。

如果我们在 ubuntu 查看：

可以看到 available 字段：

![](https://img-blog.csdnimg.cn/20191107165205133.png)

可用的 available memory=free +buffers+cached  
free 命令输出的内存状态，可以通过两个角度来查看：一个是从内核的角度来看，一个是从应用层的角度来看的。

了解具体信息：[《Linux 了解内存使用》](https://guisu.blog.csdn.net/article/details/7403855#t11)[https://guisu.blog.csdn.net/article/details/7403855#t11](https://guisu.blog.csdn.net/article/details/7403855#t11)

### **4、内存页面分类**

[《](https://blog.csdn.net/hguisu/article/details/39373311)[Linux 服务器性能评估与优化 (二)-- 内存](https://blog.csdn.net/hguisu/article/details/102620787)[》已经提到内存分页类型，重新说明：](https://blog.csdn.net/hguisu/article/details/39373311)

在 linux 内核中有 3 种内核页：  
**1）、Read pages 只读页（或代码页）**——通过主缺页中断从硬盘读取的只读页面，这些页面存在于缓冲区中包括不可改变的静态文件，二进制可执行文件和库文件。内核需要它们的时候把它们读到内存中。当内存不足的时候，内核就释放它们到空闲列表，当程序再次需要它们的时候需要通过**缺页中断**再次读到内存。

**2）Dirty pages 脏页面**——在内存中被内核修改的数据页，比如文本文件等。它们将被 pdflush 守护进程回写至磁盘， 当内存不够用时，kswapd 进程也会和 pdflush 一道进行回写磁盘以释放更多内存空间。

**3）Anonymous pages 无名页面（匿名页）**——它们属于某个进程，但是没有任何文件或后端存储与之关联，它们不能被回写进磁盘，当内存不够用时 kswapd 守护进程会将它们写入交换区直至 RAM  
释放出来。

### **5、数据页面磁盘回写**

  
应用可能直接调用 fsync() 或 sync() 系统调用将脏页面回写入磁盘， 这些系统调用会直接请求至 I/O 调度器。如果一个应用不调用它们，则 pdflush 守护进程会时不时地进行页面回写操作。

二、磁盘 i/o 性能指标
-------------

### 2.1、性能指标

**使用率：**是指磁盘处理 I/O 的时间百分比，过高的使用率 (比如超过 80%)，通产意味着磁盘 I/O 存在性能瓶颈  
**饱和度：**是指磁盘处理 I/O 的繁忙程度，过高的饱和度，意味着磁盘存在严重的性能瓶颈，当饱和度为 100% 时，磁盘无法接受新的 I/O 请求  
**IOPS**(Input/Output Per Second)，是指每秒的 I/O 请求数  
**吞吐量：**是指每秒的 I/O 请求大小，即每秒磁盘 I/O 的流量：磁盘写入加上读出的数据的大小。  
**响应时间：**是指 I/O 请求从发出到收到响应的间隔时间

注意，使用率只考虑有没有 I/O，而不考虑 I/O 的大小，换句话说，当使用率是 100% 的时候，磁盘依然有可能接受新的 I/O 请求  
不要孤立的去比较某一个非指标，而要结合读写比列，I/O 类型 (随机还是连续)，以及 I/O 大小综合分析  
比如，在数据库，大量小文件等这些类随机读写比较多的场景中，IOPS 更能反映系统的整体性能  
在多媒体等顺序读写较多的场景中，吞吐量才更能反映系统的整体性能

我们在位应用程序的服务器选型时，要限度磁盘的 I/O 性能进行基准测试，以便可以准确评估，磁盘性能是否可以满足应用程序的需求，可选的性能测试工具是 fio，来测试磁盘的 IOPS，吞吐量以及响应时间等核心指标，在基准测试时，一定要注意跟进应用程序的 I/O 特点，来具体评估指标

需要测试出，不同 I/O 大小 (一般是 512B 到 1MB 中间的某若干值) 分别在随机读，顺序读，随机写，顺序写等各种场景下的性能情况  
用性能工具得到这些指标，可以作为后续分析应用程序性能的依据，一旦发生性能问题，可以把他们作为磁盘性能的极限值，进而评估磁盘 I/O 的使用情况  
 

### **2.2、IOPS 计算**

每一次向磁盘的 IO 请求都会花费一定时间，这主要是因为磁盘必须旋转，磁头必须检索。磁盘的旋转通常被称为 “旋转延迟 (rotational delay)” （RD） ，磁头的移动杯称为 “磁盘检索 (disk seek)” （DS） 。因此每次 IO 请求的时间由 DS 和 RD 计算得来，磁盘的 RD 由驱动气的转速所固定，一 RD 被为磁盘一转的一半**，计算一块 10000 转磁盘的 RD 如下**：  
1. 算出每转的秒数：60 秒 / 10000 转 = 0.006 秒 / 转  
2. 转换为毫秒： 0.006*1000 毫秒 = 6 毫秒  
3. 算出 RD 值： 6/2 = 3 毫秒  
4. 加上平均检索时间：3+3 = 6 毫秒  
5. 加上内部转移等待时间： 6+2 = 8 毫秒  
6. 算出一秒的 IO 数：1000 毫秒 / 8 毫秒 = 125 次 / 秒（IOPS）  
在一块万转硬盘上应用每请求一次 IO 需要 8 毫秒，每秒可提供 120 到 150 次操作。

IOPS 可细分为如下几个指标：

1.  Toatal IOPS，混合读写和顺序随机 I/O 负载情况下的磁盘 IOPS，这个与实际 I/O 情况最为相符，大多数应用关注此指标。
2.  Random Read IOPS，100% 随机读负载情况下的 IOPS。
3.  Random Write IOPS，100% 随机写负载情况下的 IOPS。
4.  Sequential Read IOPS，100% 顺序读负载情况下的 IOPS。
5.  Sequential Write IOPS，100% 顺序写负载情况下的 IOPS。

### 2.3 IOPS 与吞吐量的关系

吞吐量＝ IOPS* 平均 I/O SIZE。从公式可以看出： I/O SIZE 越大，IOPS 越高，那么每秒 I/O 的吞吐量就越高。因此，我们会认为 IOPS 和吞吐量的数值越高越好。实际上，对于一个磁盘来讲，这两个参数均有其最大值，而且这两个参数也存在着一定的关系。  
 

**三、顺序 IO 和 随机 IO**
-------------------

IO 可分为顺序 IO 和 随机 IO 两种，性能监测前需要弄清楚系统偏向顺序 IO 的应用还是随机 IO 应用。

**顺序 IO** ：同时顺序请求大块数据,   假设我们已经找到了大块数据的第一块数据，并且其他所需的数据就在这一块数据后边，那么就不需要重新寻址，可以依次拿到我们所需的数据，**这个就叫顺序 IO。**  
 

顺序 IO, 比如数据库执行大量的查询、流媒体服务等，顺序 IO 可以同时很快的移动大量数据。可以这样来评估 IOPS 的性能，用每秒读写 IO 字节数除以每秒读写 IOPS 数，rkB/s 除以 r/s，wkB/s 除以 w/s. 下面显示的是连续 2 秒的 IO 情况，可见每次 IO 写的数据是增加的（45060.00 / 99.00 = 455.15 KB per IO，54272.00 / 112.00 = 484.57 KB per IO）。相对随机 IO 而言，顺序 IO 更应该重视每次 IO 的吞吐能力（KB per IO）：

```
$ iostat -kx 1
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.00    0.00    2.50   25.25    0.00   72.25
 
Device:  rrqm/s   wrqm/s   r/s   w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await  svctm  %util
sdb       24.00 19995.00 29.00 99.00  4228.00 45060.00   770.12    45.01  539.65   7.80  99.80
 
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.00    0.00    1.00   30.67    0.00   68.33
 
Device:  rrqm/s   wrqm/s   r/s   w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await  svctm  %util
sdb        3.00 12235.00  3.00 112.00   768.00 54272.00   957.22   144.85  576.44   8.70 100.10
```

**随机 IO：** 是指随机请求数据。 假设我们所需要的数据是随机分散在磁盘的不同页的不同扇区中的，那么找到相应的数据需要等到磁臂（寻址作用）旋转到指定的页，然后盘片寻找到对应的扇区，才能找到我们所需要的一块数据，一次进行此过程直到找完所有数据，这个就是随机 IO，读取数据速度较慢。

随机 IO 速度不依赖于数据的大小和排列，依赖于磁盘的每秒能 IO 的次数，比如 Web 服务、Mail 服务等每次请求的数据都很小，随机 IO 每秒同时会有更多的请求数产生，所以磁盘的每秒能 IO 多少次是关键。

```
$ iostat -kx 1
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.75    0.00    0.75    0.25    0.00   97.26
 
Device:  rrqm/s   wrqm/s   r/s   w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await  svctm  %util
sdb        0.00    52.00  0.00 57.00     0.00   436.00    15.30     0.03    0.54   0.23   1.30
 
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.75    0.00    0.75    0.25    0.00   97.24
 
Device:  rrqm/s   wrqm/s   r/s   w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await  svctm  %util
sdb        0.00    56.44  0.00 66.34     0.00   491.09    14.81     0.04    0.54   0.19   1.29
```

按照上面的公式得出：436.00 / 57.00 = 7.65 KB per IO，491.09 / 66.34 = 7.40 KB per IO. 与顺序 IO 比较发现，随机 IO 的 KB per IO 小到可以忽略不计，可见对于随机 IO 而言重要的是每秒能 IOPS 的次数，而不是每次 IO 的吞吐能力（KB per IO）。

三、监测磁盘 I/O
----------

在一定条件下系统会出现 I/O 瓶颈，可由很多监测工具监测到，如 top，vmstat，iostat，sar 等。这些工具输入的信息大致一样，也有不同之处，下面将讨论出现 I/O 瓶颈的情况。

### 1、利用 iostat 评估磁盘性能

[root@webserver ~]#   iostat -d 2 3

Linux 2.6.9-42.ELsmp (webserver)        12/01/2008      _i686_  (8 CPU)

Device:         tps  Blk_read/s   Blk_wrtn/s  Blk_read      Blk_wrtn

sda               1.87         2.58       114.12        6479462     286537372

Device:         tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn

sda               0.00         0.00         0.00              0                0

Device:         tps   Blk_read/s   Blk_wrtn/s   Blk_read    Blk_wrtn

sda               1.00         0.00        12.00             0                24

对上面每项的输出解释如下：

tps 平均每秒钟的 IO 请求次数

Blk_read/s 表示每秒读取的数据块数。

Blk_wrtn/s 表示每秒写入的数据块数。

Blk_read 表示读取的所有块数。

Blk_wrtn 表示写入的所有块数。

>  可以通过 Blk_read/s 和 Blk_wrtn/s 的值对磁盘的读写性能有一个基本的了解，如果 Blk_wrtn/s 值很大，表示磁盘的写操作很频繁，可以考虑优化磁盘或者优化程序，如果 Blk_read/s 值很大，表示磁盘直接读取操作很多，可以将读取的数据放入内存中进行操作。
> 
>  对于这两个选项的值没有一个固定的大小，根据系统应用的不同，会有不同的值，但是有一个规则还是可以遵循的：长期的、超大的数据读写，肯定是不正常的，这种情况一定会影响系统性能。

### **2、利用 sar 评估磁盘性能**

         通过 “sar –d” 组合，可以对系统的磁盘 IO 做一个基本的统计，请看下面的一个输出：

[root@webserver ~]# sar -d 2 3

Linux 2.6.9-42.ELsmp (webserver)        11/30/2008      _i686_  (8 CPU)

11:09:33 PM  DEV     tps   rd_sec/s   wr_sec/s  avgrq-sz  avgqu-sz  await  svctm   %util

11:09:35 PM dev8-0  0.00  0.00            0.00        0.00          0.00         0.00   0.00     0.00

11:09:35 PM  DEV     tps  rd_sec/s    wr_sec/s  avgrq-sz  avgqu-sz await   svctm   %util

11:09:37 PM dev8-0  1.00  0.00         12.00        12.00         0.00        0.00    0.00     0.00

11:09:37 PM   DEV    tps    rd_sec/s  wr_sec/s   avgrq-sz  avgqu-sz await  svctm   %util

11:09:39 PM dev8-0  1.99   0.00         47.76         24.00       0.00        0.50    0.25     0.05

Average:  DEV          tps    rd_sec/s   wr_sec/s  avgrq-sz  avgqu-sz   await  svctm   %util

Average:  dev8-0      1.00   0.00          19.97         20.00       0.00         0.33    0.17     0.02

      需要关注的几个参数含义：

     await 表示平均每次设备 I/O 操作的等待时间（以毫秒为单位）。

     svctm 表示平均每次设备 I/O 操作的服务时间（以毫秒为单位）。

     %util 表示一秒中有百分之几的时间用于 I/O 操作。

对以磁盘 IO 性能，一般有如下评判标准：

     正常情况下 svctm 应该是小于 await 值的，而 svctm 的大小和磁盘性能有关，CPU、内存的负荷也会对 svctm 值造成影响，过多的请求也会间接的导致 svctm 值的增加。

     await 值的大小一般取决与 svctm 的值和 I/O 队列长度以及 I/O 请求模式，如果 svctm 的值与 await 很接近，表示几乎没有 I/O 等待，磁盘性能很好，如果 await 的值远高于 svctm 的值，则表示 I/O 队列等待太长，系统上运行的应用程序将变慢，此时可以通过更换更快的硬盘来解决问题。

     %util 项的值也是衡量磁盘 I/O 的一个重要指标，如果 %util 接近 100%，表示磁盘产生的 I/O 请求太多，I/O 系统已经满负荷的在工作，该磁盘可能存在瓶颈。长期下去，势必影响系统的性能，可以通过优化程序或者通过更换更高、更快的磁盘来解决此问题。

如下图，可以看到磁盘 I/O 系统已经满负荷的在工作：

![](https://img-blog.csdnimg.cn/20191025171344601.png)

### 3、利用 iotop 评估磁盘性能

Linux 下的 IO 统计工具如 iostat, nmon 等大多数是只能统计到 per 设备的读写情况, 如果你想知道每个进程是如何使用 IO 的就比较麻烦.

iotop 是一个用来监视磁盘 I/O 使用状况的 top 类工具。iotop 具有与 top 相似的 UI，其中包括 PID、用户、I/O、进程等相关信息。

简单安装：yum -y install iotop

iotop [OPTIONS]：

--version #显示版本号  
-h, --help #显示帮助信息  
-o, --only #显示进程或者线程实际上正在做的 I/O，而不是全部的，可以随时切换按 o  
-b, --batch #运行在非交互式的模式  
-n NUM, --iter=NUM #在非交互式模式下，设置显示的次数，  
-d SEC, --delay=SEC #设置显示的间隔秒数，支持非整数值  
-p PID, --pid=PID #只显示指定 PID 的信息  
-u USER, --user=USER #显示指定的用户的进程的信息  
-P, --processes #只显示进程，一般为显示所有的线程  
-a, --accumulated #显示从 iotop 启动后每个线程完成了的 IO 总数  
-k, --kilobytes #以千字节显示  
-t, --time #在每一行前添加一个当前的时间  
-q, --quiet #suppress some lines of header (implies --batch). This option can be specified up to three times to remove header lines.  
-q column names are only printed on the first iteration,  
-qq column names are never printed,  
-qqq the I/O summary is never printed.

![](https://img-blog.csdnimg.cn/2019102517141081.png)