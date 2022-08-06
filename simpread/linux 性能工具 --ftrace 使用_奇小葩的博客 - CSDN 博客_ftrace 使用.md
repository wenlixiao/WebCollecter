> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/u012489236/article/details/119519361)

Ftrace 设计作为一个内部的 [tracer](https://so.csdn.net/so/search?q=tracer&spm=1001.2101.3001.7020) 提供给系统的开发者和设计者，帮助他们弄清 kernel 正在发生的行为，它能够调式分析延迟和性能问题。对于前一章节，我们学习了 Ftrace 发展到现在已经不仅仅是作为一个 function tracer 了，它实际上成为了一个通用的 trace 工具的框架

*   一方面已经从 function tracer 扩展到 irqsoff tracer、preemptoff tracer
*   另一方面静态的 trace event 也成为 trace 的一个重要组成部分

通过前面两节的学习，我们知道了什么是 ftrace，能够解决什么问题，从这章开始我们主要是学习，怎么去使用 ftreace 解决问题。

1 ftrace 基础用法
-------------

ftrace 通过 debugfs 向用户态提供访问接口。配置内核时激活 debugfs 后会创建目录 /sys/kernel/debug ，debugfs 文件系统就是挂载到该目录。要挂载该目录，需要将如下内容添加到 /etc/fstab 文件：

```
debugfs  /sys/kernel/debug  debugfs  defaults  0  0

```

或者可以在运行时挂载：

```
mount  -t  debugfs  debugfs  /sys/kernel/debug

```

激活内核对 ftrace 的支持后会在 debugfs 下创建一个 **tracing 目录** /sys/kernel/debug/tracing 。该目录下包含了 ftrace 的控制和输出文件

> root@100ask:/sys/kernel/debug/tracing# ls  
> available_events events README snapshot trace_pipe  
> available_filter_functions free_buffer saved_cmdlines stack_max_size trace_stat  
> available_tracers function_profile_enabled saved_cmdlines_size stack_trace tracing_cpumask  
> buffer_percent hwlat_detector saved_tgids stack_trace_filter tracing_max_latency  
> buffer_size_kb instances set_event synthetic_events tracing_on  
> buffer_total_size_kb kprobe_events set_event_pid timestamp_mode tracing_thresh  
> current_tracer kprobe_profile set_ftrace_filter trace uprobe_events  
> dynamic_events max_graph_depth set_ftrace_notrace trace_clock uprobe_profile  
> dyn_ftrace_total_info options set_ftrace_pid trace_marker  
> enabled_functions per_cpu set_graph_function trace_marker_raw  
> error_log printk_formats set_graph_notrace trace_options

其中重点关注以下文件

*   **trace 查看选择器**
    
    查看支持的跟踪器 available_tracers
    
    > root@100ask:/sys/kernel/debug/tracing# cat available_tracers  
    > hwlat blk mmiotrace function_graph wakeup_dl wakeup_rt wakeup function nop
    
    <table><thead><tr><th>类型</th><th>含义</th></tr></thead><tbody><tr><td>function</td><td>函数调用追踪器，可以看出哪个函数何时调用，可以通过过滤器指定要跟踪的函数</td></tr><tr><td>function_graph</td><td>函数调用图表追踪器，可以看出哪个函数被哪个函数调用，何时返回</td></tr><tr><td>blk</td><td>block I/O 追踪器, blktrace 用户应用程序 使用的跟踪器</td></tr><tr><td>mmiotrace</td><td>MMIO(Memory Mapped I/O) 追踪器，用于 Nouveau 驱动程序等逆向工程</td></tr><tr><td>wakeup</td><td>跟踪进程唤醒信息，进程调度延迟追踪器</td></tr><tr><td>wakeup_rt</td><td>与 wakeup 相同，但以实时进程为对象</td></tr><tr><td>nop</td><td>不会跟踪任何内核活动，将 nop 写入 current_tracer 文件可以删除之前所使用的跟踪器，并清空之前收集到的跟踪信息，即刷新 trace 文件</td></tr><tr><td>wakeup_dl</td><td>跟踪并记录唤醒 SCHED_DEADLINE 任务所需的最大延迟（如 "wakeup” 和"wakeup_rt” 一样）</td></tr><tr><td>mmiotrace</td><td>一种特殊的跟踪器，用于跟踪二进制模块。它跟踪模块对硬件的所有调用</td></tr><tr><td>hwlat</td><td>硬件延迟跟踪器。它用于检测硬件是否产生任何延迟</td></tr></tbody></table>
    
    查看当前的跟踪器 current_tracer ，可以 echo 选择
    
    > root@100ask:/sys/kernel/debug/tracing# cat current_tracer  
    > nop
    
*   **trace 使能**
    
    tracing_on ：是否往循环 buffer 写跟踪记录，可以 echo 设置
    
    > root@100ask:/sys/kernel/debug/tracing# cat tracing_on  
    > 1
    
*   **trace 过滤器选择（可选）**
    
    *   set_ftrace_filter/set_graph_notrace：（function 跟踪器）函数过滤器，echo xxx 设置要跟踪的函数，
        
        > root@100ask:/sys/kernel/debug/tracing# cat set_ftrace_filter
        > 
        > all functions enabled
        
    *   set_ftrace_notrace 为反向过滤器，设置后输出除了函数以外内容
        
    *   set_graph_function/set_graph_notrace：（function_graph 跟踪器）函数过滤器，设置后只跟踪对应函数
        
    *   set_graph_notrace 为反向过滤器，打开后进入对应函数会关闭跟踪
        
*   **trace 数据读取**
    
    *   trace：可以 cat 读取跟踪记录的 buffer 内容（查看的时候会临时停止跟踪）
    *   trace_pipe：类似 trace 可以动态读取的流媒体文件（差异是每次读取后，再读取会读取新内容）

所以对于 ftrace 的三步法为：

*   1 设置 tracer 类型
*   2 设置 tracer 参数
*   3 使能 tracer

### 1.2 function trace 实例

function，函数调用追踪器， 跟踪函数调用，默认跟踪所有函数，如果设置 set_ftrace_filter， 则跟踪过滤的函数，可以看出哪个函数何时调用。

*   available_filter_functions：列出当前可以跟踪的内核函数，不在该文件中列出的函数，无法跟踪其活动
*   enabled_functions：显示有回调附着的函数名称。
*   function_profile_enabled：打开此选项，在 trace_stat 中就会显示 function 的统计信息。
*   set_ftrace_filter：用于指定跟踪的函数
*   set_ftrace_notrace：用于指定不跟踪的函数
*   set_ftrace_pid：用于指定要跟踪特定进程的函数

**Disable tracer：**

> echo 0 > tracing_on

**设置 tracer 类型为 function：**

> echo function > current_tracer

**set_ftrace_filter 表示要跟踪的函数，这里我们只跟踪 dev_attr_show 函数：**

> echo dev_attr_show > set_ftrace_filter

**Enable tracer：**

> echo 1 > tracing_on

**提取 trace 结果：**

![](https://img-blog.csdnimg.cn/e3b284c4d380481ea3173da5a2172a8d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

从上图可以看到 function trace 一个函数的方法基本就是三板斧：

1.  设置 current_tracer 为 function
2.  设置要 trace 的函数
3.  打开 trace 开关，开始 trace
4.  提取 trace 结果

从 trace 信息我们可以获取很多重要信息：

1.  进程信息，TASK-PID，对应任务的名字
2.  进程运行的 CPU
3.  执行函数时的系统状态，包括中断，抢占等状态信息
4.  执行函数的时间辍，字段 TIMESTAMP 是时间戳，其格式为 “.”，表示执行该函数时对应的**时间戳**
5.  FUNCTION 一列则给出了**被跟踪的函数**，**函数的调用者**通过符号 “<-” 标明，这样可以观察到函数的调用关系。

function 跟踪器可以跟踪内核函数的调用情况，可用于调试或者分析 bug ，还可用于了解和观察 Linux 内核的执行过程。同时 ftrace 允许你对一个特定的进程进行跟踪，在 / sys/kernel/debug/tracing 目录下，文件 set_ftrace_pid 的值要更新为你想跟踪的进程的 PID。

> echo $PID > set_ftrace_pid

### 1.3 function_graph Trace 实例

function_graph 跟踪器则可以提供类似 C 代码的函数**调用关系信息**。通过写文件 set_graph_function 可以显示**指定**要生成**调用关系的函数**，缺省会对所有可跟踪的内核函数生成函数调用关系图。

**函数图跟踪器**对**函数的进入**与**退出**进行跟踪，这对于跟踪它的执行时间很有用。函数执行时间**超过 10 微秒**的标记一个 “+” 号，**超过 1000 微秒**的标记为一个 “！” 号。通过 echo function_graph > current_tracer 可以启用函数图跟踪器。

与 function tracer 类似，设置 function_graph 的方式如下：

**设置 tracer 类型为 function_graph：**

> echo function_graph > current_tracer

**set_graph_function 表示要跟踪的函数：**

> echo __do_fault > set_graph_function
> 
> echo 1 > tracing_on

**捕捉到的 trace 内容：**  
![](https://img-blog.csdnimg.cn/625a1dd9de304a44a9aaec75a26ade09.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

我们跟踪的是 `__do_fault` 函数，但是 function_graph tracer 会跟踪函数内的调用关系和函数执行时间，可以协助我们确定代码执行流程。比如一个函数内部执行了很多函数指针，不能确定到底执行的是什么函数，可以用 function_graph tracer 跟踪一下。

*   CPU 字段给出了**执行函数的 CPU 号**，本例中都为 1 号 CPU。
*   DURATION 字段给出了**函数执行的时间长度**，以 us 为单位。
*   FUNCTION CALLS 则给出了**调用的函数**，并显示了**调用流程**。

需要注意：

*   对于**不调用其它函数的函数**，其对应行以 “;” 结尾，而且对应的 DURATION 字段给出其运行时长；
*   对于**调用其它函数的函数**，则在其 “}” 对应行给出了**运行时长**，该时间是一个**累加值**，包括了其内部调用的函数的执行时长。DURATION 字段给出的时长并**不是精确的**，它还包含了执行 ftrace **自身的代码所耗费的时间**，所以示例中将内部函数时长累加得到的结果会与对应的外围调用函数的执行时长并不一致；不过通过该字段还是可以大致了解函数在时间上的运行开销的。

### 1.4 wakeup

wakeup tracer 追踪普通进程从被唤醒到真正得到执行之间的延迟。

![](https://img-blog.csdnimg.cn/7a1579bd3dcf4fca9d9562d3fd27906d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

### 1.5 wakeup-rt

non-RT 进程通常看平均延迟。RT 进程的最大延迟非常有意义，反应了调度器的性能

![](https://img-blog.csdnimg.cn/d6a0bb6ce49e4e3fa79a1c5d5c274f50.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

2 trace event 用法
----------------

### 2.1 trace event 简介

trace event 就是利用 ftrace [框架](https://so.csdn.net/so/search?q=%E6%A1%86%E6%9E%B6&spm=1001.2101.3001.7020)，实现低性能损耗，对执行流无影响的一种信息输出机制。相比 printk，trace event：

*   不开启没有性能损耗
*   开启后不影响代码流程
*   不需要重新编译内核即可获取 debug 信息

### 2.2 使用实例

上面提到了 `function` 的 trace，在 ftrace 里面，另外用的多的就是 `event` 的 trace，我们可以在 `events` 目录下面看支持那些事件：

![](https://img-blog.csdnimg.cn/a9bc7c18bc6849fa8cdc796c8ec52731.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

上面列出来的都是分组的，我们可以继续深入下去，譬如下面是查看 `sched` 相关的事件

![](https://img-blog.csdnimg.cn/e83e729ce5b24987aae238349bcd1872.png#pic_center)

对于某一个具体的事件，我们也可以查看：

![](https://img-blog.csdnimg.cn/3fc9dac2d0bc4dee8817fe388955c9b4.png#pic_center)

上述目录里面，都有一个 `enable` 的文件，我们只需要往里面写入 1，就可以开始 trace 这个事件。譬如下面就开始 trace `sched_wakeup` 这个事件

![](https://img-blog.csdnimg.cn/65243a76295d483481fbf43dd570d03c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

我们也可以 trace `sched` 里面的所有事件

![](https://img-blog.csdnimg.cn/65cf956f977e459baf5192521eeaf2ca.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

当然也可以 trace 所有的事件：

![](https://img-blog.csdnimg.cn/cabc5f2c608249d98b6caa20c294aa6c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

3 高级技巧
------

**查看函数调用栈**

查看函数调用栈是内核调试最最基本得需求，常用方法：

*   函数内部添加 WARN_ON(1)
*   ftrace

trace 函数的时候，设置 `echo 1 > options/func_stack_trace` 即可在 trace 结果中获取追踪函数的调用栈。

以 `dev_attr_show` 函数为例，看看 ftrace 如何帮我们获取调用栈：

```
#cd /sys/kernel/debug/tracing
#echo 0 > tracing_on
#echo function > current_tracer
#echo schedule > set_ftrace_filter
// 设置 func_stack_trace
#echo 1 > options/func_stack_trace
#echo 1 > tracing_on

```

![](https://img-blog.csdnimg.cn/ff21304f176c421aa2329b35ad253b94.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

**如何跟踪一个命令，但是这个命令执行时间很短**

我们可以设置 ftrace 过滤器控制相关文件

<table><thead><tr><th>文件名</th><th>功能</th></tr></thead><tbody><tr><td>set_ftrace_filter</td><td>function tracer 只跟踪某个函数</td></tr><tr><td>set_ftrace_notrace</td><td>function tracer 不跟踪某个函数</td></tr><tr><td>set_graph_function</td><td>function_graph tracer 只跟踪某个函数</td></tr><tr><td>set_graph_notrace</td><td>function_graph tracer 不跟踪某个函数</td></tr><tr><td>set_event_pid</td><td>trace event 只跟踪某个进程</td></tr><tr><td>set_ftrace_pid</td><td>function/function_graph tracer 只跟踪某个进程</td></tr></tbody></table>

> 如果这时候问：如何跟踪某个进程内核态的某个函数？
> 
> 答案是肯定的，将被跟踪进程的 pid 设置到 `set_event_pid/set_ftrace_pid` 文件即可。
> 
> 但是如果问题变成了，我要调试 `kill` 的内核执行流程，如何办呢？
> 
> 因为 `kill` 运行时间很短，我们不能知道它的 pid，所以就没法过滤了。
> 
> 调试这种问题的小技巧，即 **脚本化**，这个技巧在很多地方用到：
> 
> ```
> sh -c "echo $$ > set_ftrace_pid; echo 1 > tracing_on; kill xxx; echo 0 > tracing_on"
> 
> ```

**如何跟踪过滤多个进程？多个函数？**

*   函数名雷同，可以使用正则匹配
    
    ```
    # cd /sys/kernel/debug/tracing
    # echo 'dev_attr_*' > set_ftrace_filter
    # cat set_ftrace_filter
    dev_attr_store
    dev_attr_show
    
    ```
    
*   追加某个函数
    

用法为：`echo xxx >> set_ftrace_filter`，例如，先设置 `dev_attr_*`：

```
# cd /sys/kernel/debug/tracing
# echo 'dev_attr_*' > set_ftrace_filter
# cat set_ftrace_filter
dev_attr_store
dev_attr_show

```

再将 `ip_rcv` 追加到跟踪函数中：

```
# cd /sys/kernel/debug/tracing
# echo ip_rcv >> set_ftrace_filter
# cat set_ftrace_filter
dev_attr_store
dev_attr_show
ip_rcv

```

*   基于模块过滤
    
    格式为：`<function>:<command>:<parameter>`，例如，过滤 ext3 module 的 write* 函数：
    
    ```
    $ echo 'write*:mod:ext3' > set_ftrace_filter
    
    ```
    
*   从过滤列表中删除某个函数，使用 “感叹号”
    
    感叹号用来移除某个函数，把上面追加的 `ip_rcv` 去掉：
    
    ```
    # cd /sys/kernel/debug/tracing
    # cat set_ftrace_filter
    dev_attr_store
    dev_attr_show
    ip_rcv
    # echo '!ip_rcv' >> set_ftrace_filter
    # cat set_ftrace_filter
    dev_attr_store
    dev_attr_show
    
    ```
    

4 前端工具
------

我们可以手工操作 / sys/kernel/debug/tracing 路径下的大量的配置文件接口，来使用 ftrace 的强大功能。但是这些接口对普通用户来说太多太复杂了，我们可以使用对 ftrace 功能进行二次封装的一些命令来操作。

trace-cmd 就是 ftrace 封装命令其中的一种。该软件包由两部分组成

*   trace-cmd：提供了数据抓取和数据分析的功能
*   kernelshark：可以用图形化的方式来详细分析数据，也可以做数据抓取

### 4.1 trace-cmd

下载编译 ARM64 trace-cmd 方法

```
git clone [https://github.com/rostedt/trace-cmd.git](https://github.com/rostedt/trace-cmd.git)
export CROSS_COMPILE=aarch64-linux-gnu-
export ARCH=arm64
make

```

![](https://img-blog.csdnimg.cn/0d70ae52668d4f468bb08fcf6db32c4f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

先通过 record 子命令将结果记录到 trace.dat，再通过 report 命令进行结果提取。命令解释：

*   `-p`：指定当前的 tracer，类似 `echo function > current_tracer`，可以是支持的 tracer 中的任意一个
*   `-l`：指定跟踪的函数，可以设置多个，类似 `echo function_name > set_ftrace_filter`
*   `--func-stack`：记录被跟踪函数的调用栈

在很有情况下不能使用函数追踪，需要依赖 **事件追踪** 的支持，例如：

![](https://img-blog.csdnimg.cn/6f125eb5fd40478b821d7c098597c34a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

### 4.2 kernelshark 图形化分析数据

`trace-cmd report`主要是使用统计的方式来找出热点。如果要看`vfs_read()`一个具体的调用过程，除了使用上一节的`trace-cmd report`命令，还可以使用 kernelshark 图形化的形式来查看，可以在板子上使用`trace-cmd record` 记录事件，把得到的 trace.data 放到 linux 桌面系统，用`kernelshark`打开，看到图形化的信息

![](https://img-blog.csdnimg.cn/28e0e24205414735a9dbecea733203a3.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0ODkyMzY=,size_16,color_FFFFFF,t_70#pic_center)

5 参考文档
------

[Linux ftrace 2.1、ftrace 的使用](https://blog.csdn.net/pwl999/article/details/80420905)

[Ftrace 基本用法](http://tinylab.org/ftrace-usage/)

[ftrace 调试 Linux 内核]([ftrace 调试 Linux 内核 | 码农家园 (codenong.com)](https://www.codenong.com/cs106475776/))

[Linux 进程管理 (7) 实时调度](https://my.oschina.net/u/4344778/blog/3872899)

[使用 ftrace 分析函数性能](https://blog.csdn.net/pwl999/article/details/107067509)

[Trace-cmd and kernelshark trace viewer - stm32mpu (stmicroelectronics.cn)](https://wiki.stmicroelectronics.cn/stm32mpu/wiki/Trace-cmd_and_kernelshark_trace_viewer)

[LTTng - stm32mpu (stmicroelectronics.cn)](https://wiki.stmicroelectronics.cn/stm32mpu/wiki/LTTng)

[linux ftrace 原理 - 极客分享 (geek-share.com)](https://www.geek-share.com/detail/2667839806.html)

[【原创】Ftrace 的配置和使用 - HPYU - 博客园 (cnblogs.com)](https://www.cnblogs.com/hpyu/p/14348151.html)