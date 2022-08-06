> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/u012489236/article/details/119427091)

我们做内核开发的时候，我们经常要去跟踪 linux 内核的函数调用关系，对于我们来说 ftrace 是一个十分好用的工具，值得我们好好学习。ftrace **不只是一个函数跟踪工具**，它的跟踪能力之强大，还能调试和分析诸如延迟、意外代码路径、性能问题等一大堆问题。它也是一种很好的学习工具。本章的主要是学习：

*   ftrace 是什么
*   ftrace 来解决什么问题

1 什么是 ftrace
------------

首先，在学习 ftrace 之前，我们要知道它是什么？根据 [linux ftrace](https://www.kernel.org/doc/Documentation/trace/ftrace.txt) 的详细介绍，ftrace 是一个 linux 内部的一个 trace 工具，用于帮助开发者和系统设计者知道内核当前正在干什么，从而更好的去分析性能问题。

### 1.1 ftrace 的由来

ftrace 是由 Steven Rostedy 和 Ingo Molnar 在内核 2.6.27 版本中引入的，那个时候，systemTap 已经开始崭露头角，其它的 trace 工具包括 LTTng 等已经发展多年，那么为什么人们还需要开发一个 trace 工具呢？

> **SystemTap** 项目是 Linux 社区对 SUN Dtrace 的反应，目标是达到甚至超越 Dtrace 。因此 SystemTap 设计比较复杂，Dtrace 作为 SUN 公司的一个项目开发了多年才最终稳定发布，况且得到了 Solaris 内核中每个子系统开发人员的大力支持。 SystemTap 想要赶超 Dtrace，困难不仅是一样，而且更大，因此她始终处在不断完善自身的状态下，在真正的产品环境，人们依然无法放心的使用她。不当的使用和 SystemTap 自身的不完善都有可能导致系统崩溃。

Ftrace 的设计目标简单，本质上是一种**静态代码插装技术**，**不需要**支持某种编程接口让用户自定义 trace 行为。静态代码插装技术更加可靠，不会因为用户的**不当使用**而导致**内核崩溃**。 ftrace 代码量很小，稳定可靠。同时`Ftrace` 有重大的创新：

*   Ftrace 只需要在函数入口插入一个外部调用：mcount
*   Ftrace 巧妙的拦截了函数返回的地址，从而可以在运行时先跳到一个事先准备好的统一出口，记录各类信息，然后再返回原来的地址
*   Ftrace 在链接完成以后，把所有插入点地址都记录到一张表中，然后默认把所有插入点都替换成为空指令（nop），因此默认情况下 Ftrace 的开销几乎是 0
*   Ftrace 可以在运行时根据需要通过 Sysfs 接口使能和使用，即使在没有第三方工具的情况下也可以方便使用

### 1.2 ftrace 原理

ftrace 的名字由 function trace 而来。function trace 是利用 gcc 编译器在编译时在每个函数的入口地址放置一个 probe 点，这个 probe 点会调用一个 probe 函数（gcc 默认调用名为 mcount 的函数），这样这个 probe 函数会对每个执行的内核函数进行跟踪（其实有少数几个内核函数不会被跟踪），并打印 log 到一个内核中的环形缓存（ring buffer）中，而用户可以通过 debugfs 来访问这个环形缓存中的内容。

各类 [tracer](https://so.csdn.net/so/search?q=tracer&spm=1001.2101.3001.7020) 往 ftrace 主框架注册，不同的 trace 则在不同的 probe 点把信息通过 probe 函数给送到 ring buffer 中，再由暴露在用户态 debufs 实现相关控制。其主要的框架图如下图所示

![](https://img-blog.csdnimg.cn/ce2dc592eef547c988211f01ee01a493.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

其主要由两部分构成

*   **ftrace Framework core:** 其主要包括利用 debugfs 系统在 /debugfs 下建立 tracing 目录，对用户空间输出 trace 信息，并提供了一系列的控制文件
*   **一系列的 tracer：** 每个 tracer 完成不同的功能，ftrace 的 trace 信息保存在 ring buffer(内存缓冲区) 中，它们统一由 framework 管理

对于 ftrace 有两种主要的跟踪机制往缓冲区写数据

*   ** 动态探针：** 可以动态跟踪内核函数的调用栈，包括 function tracr，function graph trace 两个 tracer。其原理是利用 mcount 机制，在内核编译时，在每个函数入口保留数个字节，然后在使用 ftrace 时，将保留的字节替换为需要的指令，比如跳转到需要的执行探测操作的代码。
*   ** 静态探针：** 是在内核代码中调用 ftrace 提供的相应接口实现，称之为静态是因为，是在内核代码中写死的，静态编译到内核代码中的，在内核编译后，就不能再动态修改。在开启 ftrace 相关的内核配置选项后，内核中已经在一些关键的地方设置了静态探测点，需要使用时，即可查看到相应的信息。

> ftrace 利用了 gcc 的 profile 特性，gcc 的 -pg 选项将在每个函数的入口处加入对 mcount 的代码调用。
> 
> ![](https://img-blog.csdnimg.cn/0762d88db197491891a2e0c5efa5f06c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

> 如果 ftrace 编写了自己的 mcount stub 函数，则可借此实现 trace 功能。但是，在每个内核函数入口加入 trace 代码，必然影响内核的性能，为了减小对内核性能的影响，ftrace 支持动态 trace 功能。  
> 当 COFNIG_DYNAMIC_FTRACE 被选中后，内核编译时会调用 recordmcount.pl 脚本，将每个函数的地址写入一个特殊的段：__mcount_loc

2 ftrace 控制机制
-------------

要使用 ftrace，首先就是需要将系统的 debugfs 或者 tracefs 给挂载到某个地方，幸运的是，几乎所有的 Linux 发行版，都开启了 debugfs/tracefs 的支持，所以我们也没必要去重新编译内核了。

在比较老的内核版本，譬如 CentOS 7 的上面，debugfs 通常被挂载到 `/sys/kernel/debug` 上面（debug 目录下面有一个 tracing 目录），而比较新的内核，则是将 tracefs 挂载到 `/sys/kernel/tracing`，无论是什么，我都喜欢将 tracing 目录直接 link 到 `/tracing`。后面都会假设直接进入了 `/tracing` 目录，在讲解 ftrace 的 tracer 之前，我们先来看看 tracing 目录下的文件，它们提供了对 ftrace trace 过程的控制。

![](https://img-blog.csdnimg.cn/5cc2feba14b54352b4234a9c2618d261.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

tracing 目录下的文件分成了下面四类:

1.  提示类：显示当前系统可用的 event，tracer 列表
2.  控制类：控制 ftrace 的跟踪参数
3.  显示类：显示 trace 信息
4.  辅助类：一些不明或者不重要的辅助信息

### 2.1 提示类

<table><thead><tr><th>ftrace 文件</th><th>作用</th></tr></thead><tbody><tr><td>available_events</td><td>可用事件列表，也可查看 events/ 目录</td></tr><tr><td>available_filter_functions</td><td>当前内核导出的可以跟踪的函数</td></tr><tr><td>dyn_ftrace_total_info</td><td>显示 available_filter_functins 中跟中函数的数目</td></tr><tr><td>available_tracers</td><td>可用的 tracer，不同的 tracer 有不同的功能</td></tr><tr><td>events</td><td>\1. 查看可用事件列表以及事件参数 (事件包含的内核上下文信息)<br>cat events/sched/sched_switch/format<br>2. 设置事件的过滤条件<br>echo ‘next_comm ~ “cs”‘ &gt; events/sched/sched_switch/filter</td></tr></tbody></table>

### 2.2 控制类

<table><thead><tr><th>使用 tracer</th><th>ftrace 文件</th><th>作用</th></tr></thead><tbody><tr><td>通用</td><td>tracing_on</td><td>用于控制跟踪打开或停止，0 停止跟踪，1 继续跟踪</td></tr><tr><td>通用</td><td>tracing_cpumask</td><td>设置允许跟踪特定 CPU</td></tr><tr><td>通用</td><td>tracing_max_latency</td><td>记录 Tracer 的最大延时</td></tr><tr><td>通用</td><td>tracing_thresh</td><td>延时记录 Trace 的阈值，当延时超过此值时才开始记录 Trace。单位是 ms，只有非 0 才起作用</td></tr><tr><td>通用</td><td>events</td><td>1. 查看可用事件列表以及事件参数 (事件包含的内核上下文信息)<br>cat events/sched/sched_switch/format<br>2. 设置事件的过滤条件<br>echo ‘next_comm ~ “cs”‘ &gt; events/sched/sched_switch/filter</td></tr><tr><td>通用</td><td>set_event</td><td>设置跟踪的 event 事件，与通过 events 目录内的 filter 文件设置一致</td></tr><tr><td>通用</td><td>current_tracer</td><td>1. 设置或者显示当前使用的跟踪器列表<br>2. 系统缺省为 nop，可以通过写入 nop 重置跟踪器<br>3. 使用 echo 将跟踪器名字写入即可打开<br>echo function_graph &gt; current_tracer</td></tr><tr><td>通用</td><td>buffer_size_kb</td><td>设置单个 CPU 所使用的跟踪缓存的大小<br>如果跟踪太多，旧的信息会被新的跟踪信息覆盖掉<br>不想被覆盖需要先将 current_trace 设置为 nop 才可以</td></tr><tr><td>通用</td><td>buffer_total_size_kb</td><td>显示所有 CPU ring buffer 大小之和</td></tr><tr><td>通用</td><td>trace_options</td><td>trace 过程的复杂控制选项 控制 Trace 打印内容或者操作跟踪器 也可通过 options / 目录设置</td></tr><tr><td>通用</td><td>options/</td><td>显示 trace_option 的设置结果<br>也可以直接设置，作用同 trace_options</td></tr><tr><td>func</td><td>function_profile_enabled</td><td>打开此选项，trace_stat 就会显示 function 的统计信息<br>echo 0/1 &gt; function_profile_enabled</td></tr><tr><td>func</td><td>set_ftrace_pid</td><td>设置跟踪的 pid</td></tr><tr><td>func</td><td>set_ftrace_filter</td><td>用于显示指定要跟踪的函数</td></tr><tr><td>func</td><td>set_ftrace_notrace</td><td>用于指定不跟踪的函数，缺省为空</td></tr><tr><td>graph</td><td>max_graph_depth</td><td>函数嵌套的最大深度</td></tr><tr><td>graph</td><td>set_graph_function</td><td>设置要清晰显示调用关系的函数<br>缺省对所有函数都生成调用关系</td></tr><tr><td>Stack</td><td>stack_max_size</td><td>当使用 stack 跟踪器时，记录产生过的最大 stack size</td></tr><tr><td>Stack</td><td>stack_trace</td><td>显示 stack 的 back trace</td></tr><tr><td>Stack</td><td>stack_trace_filter</td><td>设置 stack tracer 不检查的函数名称</td></tr></tbody></table>

### 2.3 输出类

<table><thead><tr><th>ftrace 文件</th><th>作用</th></tr></thead><tbody><tr><td>printk_formats</td><td>提供给工具读取原始格式 trace 的文件</td></tr><tr><td>trace</td><td>查看 ring buffer 内跟踪信息<br>echo &gt; trace 可以清空当前 RingBuffer</td></tr><tr><td>trace_pipe</td><td>输出和 trace 一样的内容，但输出 Trace 同时将 RingBuffer 清空<br>可避免 RingBuffer 的溢出<br>保存文件内容: cat trace_pipe &gt; trace.txt &amp;</td></tr><tr><td>snapshot</td><td>是对 trace 的 snapshot<br>echo 0 清空缓存，并释放对应内存<br>echo 1 进行对当前 trace 进行 snapshot，如没有内存则分配<br>echo 2 清空缓存，不释放也不分配内存</td></tr><tr><td>trace_clock</td><td>显示当前 Trace 的 timestamp 所基于的时钟，默认使用 local 时钟<br>local：默认时钟；可能无法在不同 CPU 间同步<br>global：不同 CUP 间同步，但是可能比 local 慢<br>counter：跨 CPU 计数器，需要分析不同 CPU 间 event 顺序比较有效</td></tr><tr><td>trace_marker</td><td>从用户空间写入标记到 trace 中，用于用户空间行为和内核时间同步</td></tr><tr><td>trace_stat</td><td>每个 CPU 的 Trace 统计信息</td></tr><tr><td>per_cpu/</td><td>trace 等文件的输出是综合所有 CPU 的，如果你关心单个 CPU 可以进入 per_cpu 目录，里面有这些文件的分 CPU 版本</td></tr><tr><td>enabled_functions</td><td>显示有回调附着的函数名称</td></tr><tr><td>saved_cmdlines</td><td>放 pid 对应的 comm 名称作为 ftrace 的 cache，这样 ftrace 中不光能显示 pid 还能显示 comm</td></tr><tr><td>saved_cmdlines_size</td><td>saved_cmdlines 的数目</td></tr></tbody></table>

3 ftrace 的基础用法
--------------

### 3.1 内核配置

ftrace 提供了**不同的跟踪器**，以用于**不同的场合**，比如跟踪**内核函数调用**、对**上下文切换**进行跟踪、查看**中断被关闭的时长**、跟踪**内核态中的延迟**以及**性能问题**等。

系统开发人员可以使用 ftrace 对内核进行**跟踪调试**，以找到内核中出现的问题的根源，方便对其进行修复。

使用 ftrace ，首先要将其**编译进内核**，内核源码目录下的 kernel/trace/Makefile 文件给出了 ftrace 相关的**编译选项**

```
CONFIG_FTRACE=y
CONFIG_HAVE_FUNCTION_TRACER=y
CONFIG_HAVE_FUNCTION_GRAPH_TRACER=y
CONFIG_HAVE_DYNAMIC_FTRACE=y
CONFIG_FUNCTION_TRACER=y
CONFIG_IRQSOFF_TRACER=y
CONFIG_SCHED_TRACER=y
CONFIG_ENABLE_DEFAULT_TRACERS=y
CONFIG_FTRACE_SYSCALLS=y
CONFIG_PREEMPT_TRACER=y

```

### 3.2 ftrace 三板斧

1.  设置 tracer 类型
2.  设置 tracer 参数
3.  使能 tracer

4 总结
----

Ftrace 由 RedHat 的 Steve Rostedt 负责维护。到 2.6.30 为止，ftrace 提供了**不同的跟踪器**，以用于**不同的场合**，比如跟踪**内核函数调用**、对**上下文切换**进行跟踪、查看**中断被关闭的时长**、跟踪**内核态中的延迟**以及**性能问题**等。

<table><thead><tr><th>功能</th><th>功能描述</th></tr></thead><tbody><tr><td>Function tracer Function graph tracer</td><td>跟踪函数调用</td></tr><tr><td>Schedule switch tracer</td><td>跟踪进程调度情况</td></tr><tr><td>Wakeup tracer</td><td>跟踪进程的调度延迟，即高优先级进程从进入 ready 状态到获得 CPU 的延迟时间。该 tracer 只针对实时进程。</td></tr><tr><td>Irqsoff tracer</td><td>当中断被禁止时，系统无法相应外部事件，比如键盘和鼠标，时钟也无法产生 tick 中断。这意味着系统响应延迟，irqsoff 这个 tracer 能够跟踪并记录内核中哪些函数禁止了中断，对于其中中断禁止时间最长的，irqsoff 将在 log 文件的第一行标示出来，从而使开发人员可以迅速定位造成响应延迟的罪魁祸首</td></tr><tr><td>Preemptoff tracer</td><td>和前一个 tracer 类似，preemptoff tracer 跟踪并记录禁止内核抢占的函数，并清晰地显示出禁止抢占时间最长的内核函数。</td></tr><tr><td>Preemptirqsoff tracer</td><td>同上，跟踪和记录禁止中断或者禁止抢占的内核函数，以及禁止时间最长的函数</td></tr><tr><td>Branch tracer</td><td>跟踪内核程序中的 likely/unlikely 分支预测命中率情况。 Branch tracer 能够记录这些分支语句有多少次预测成功。从而为优化程序提供线索。</td></tr><tr><td>Hardware branch tracer</td><td>利用处理器的分支跟踪能力，实现硬件级别的指令跳转记录。在 x86 上，主要利用了 BTS 这个特性。</td></tr><tr><td>Initcall tracer</td><td>记录系统在 boot 阶段所调用的 init call</td></tr><tr><td>Mmiotrace tracer</td><td>记录 memory map IO 的相关信息</td></tr><tr><td>Power tracer</td><td>记录系统电源管理相关的信息</td></tr><tr><td>Sysprof tracer</td><td>缺省情况下，sysprof tracer 每隔 1 msec 对内核进行一次采样，记录函数调用和堆栈信息</td></tr><tr><td>Kernel memory tracer</td><td>内存 tracer 主要用来跟踪 slab allocator 的分配情况。包括 kfree，kmem_cache_alloc 等 API 的调用情况，用户程序可以根据 tracer 收集到的信息分析内部碎片情况，找出内存分配最频繁的代码片断，等等。</td></tr><tr><td>Workqueue statistical tracer</td><td>这是一个 statistic tracer，统计系统中所有的 workqueue 的工作情况，比如有多少个 work 被插入 workqueue，多少个已经被执行等。开发人员可以以此来决定具体的 workqueue 实现，比如是使用</td></tr><tr><td>Event tracer</td><td>跟踪系统事件，比如 timer，系统调用，中断等</td></tr></tbody></table>

![](https://img-blog.csdnimg.cn/d610aed5f6e14c1bab2e67ec90f7f3aa.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

Ftrace 的实现依赖于其他很多内核特性，比如 tracepoint[3]，debugfs[2]，kprobe[4]，IRQ-Flags[5] 等

### 5 参考文档

https://hotttao.github.io/2020/01/03/linux_perf/03_ftrace/

http://tinylab.org/ftrace-principle-and-practice/

https://www.cnblogs.com/arnoldlu/p/7211249.html

[在 Linux 下做性能分析 2：ftrace](https://zhuanlan.zhihu.com/p/22130013)