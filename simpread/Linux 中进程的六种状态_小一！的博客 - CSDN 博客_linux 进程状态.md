> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq_49613557/article/details/120294908)

**Linux 中进程的六种状态**

### 目录

*   [R 运行状态（running）](#Rrunning_12)
*   [S 睡眠状态（sleeping)](#Ssleeping_25)
*   [D 磁盘休眠状态（Disk sleep）](#DDisk_sleep_38)
*   [T 停止状态（stopped）](#Tstopped_45)
*   [Z 僵尸状态（Zombies）](#ZZombies_47)
*   *   [僵尸进程是什么](#_55)
    *   [为什么要有僵尸进程](#_58)
    *   [僵尸进程的危害](#_60)
*   [X 死亡状态（dead）](#Xdead_69)
*   [孤儿进程](#_72)

为了弄明白正在运行的进程是什么意思，我们需要知道进程的不同状态。一个进程可以有几个状态（在 Linux 内核里，进程有时候也叫做任务）。

![](https://img-blog.csdnimg.cn/93b7bc5766f14b05961ab3378fa9fbbc.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bCP5LiA77yB,size_20,color_FFFFFF,t_70,g_se,x_16)

![](https://img-blog.csdnimg.cn/58f1a98e297148498f2fe624b7fe0181.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bCP5LiA77yB,size_20,color_FFFFFF,t_70,g_se,x_16)

R 运行状态（running）
===============

并不意味着进程一定在运行中，它表明进程要么是在运行中要么在运行队列。

问题：如果只有一个 CPU, 可不可以存在多个 R 状态的进程。  
现场试一下：用 fork 创建子进程，试一下  
![](https://img-blog.csdnimg.cn/2879b7678ce041f3b4806b3beb19f29b.png)  
![](https://img-blog.csdnimg.cn/b585e86aedd0491682f82b117d3a7662.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bCP5LiA77yB,size_20,color_FFFFFF,t_70,g_se,x_16)

![](https://img-blog.csdnimg.cn/5572bb627eb64b148958ad06e92f02c8.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bCP5LiA77yB,size_20,color_FFFFFF,t_70,g_se,x_16)  
可以看到，是可以存在多个运行状态的。  
进程是 R 状态，不代表正在运行，代表可被调度。换句话说，进程只有是 R 状态才可被调度，其他状态要先转为 R 状态，才能被 OS 调度。  
![](https://img-blog.csdnimg.cn/7a0f56e0b93e4eb09365f78587bce695.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bCP5LiA77yB,size_20,color_FFFFFF,t_70,g_se,x_16)

S 睡眠状态（sleeping)
================

意味着进程在等待事件完成（这里的睡眠有时候也叫做可中断睡眠（interruptible sleep））。  
代码实现一下：  
![](https://img-blog.csdnimg.cn/7298253e85e14088a1e29ffce739c002.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bCP5LiA77yB,size_20,color_FFFFFF,t_70,g_se,x_16)

![](https://img-blog.csdnimg.cn/d4eff6ce8ce5433b9a1bb744c2324406.png)  
休眠状态是在等待某种条件就绪，在休眠状态，可被操作系统杀死，也叫浅度睡眠。

处于这个状态的进程因为等待某某事件的发生（比如等待 socket 连接、等待信号量），而被挂起。这些进程的 task_struct 结构被放入对应事件的等待队列中。当这些事件发生时（由外部中断触发、或由其他进程触发），对应的等待队列中的一个或多个进程将被唤醒。

![](https://img-blog.csdnimg.cn/5572bb627eb64b148958ad06e92f02c8.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bCP5LiA77yB,size_20,color_FFFFFF,t_70,g_se,x_16)  
由上图看到，一般情况下，进程列表中的绝大多数进程都处于 S 状态（除非机器的负载很高）。毕竟 CPU 就这么一两个，进程动辄几十上百个，如果不是绝大多数进程都在睡眠，CPU 又怎么响应得过来。

D 磁盘休眠状态（Disk sleep）
====================

有时候也叫不可中断睡眠状态（uninterruptible sleep），在这个状态的进程通常会等待 IO 的结束。

可是为什么有了 S 状态，又来一个 D 状态呢？有什么作用呢？  
举例说明：  
![](https://img-blog.csdnimg.cn/42e59e90545649cb9c0066e60d59bd1f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bCP5LiA77yB,size_20,color_FFFFFF,t_70,g_se,x_16)

T 停止状态（stopped）
===============

可以通过发送 SIGSTOP 信号给进程来停止（T）进程。这个被暂停的进程可以通过发送 SIGCONT 信号让进程继续运行。

Z 僵尸状态（Zombies）
===============

代码实现一下僵尸状态：  
子进程 5 秒结束，父进程死循环，不读取子进程结束信息。

![](https://img-blog.csdnimg.cn/7c555ca3161441afa5f18b4cdd97c6a3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bCP5LiA77yB,size_20,color_FFFFFF,t_70,g_se,x_16)  
![](https://img-blog.csdnimg.cn/29eb761d62824b0abff214f291bccb33.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bCP5LiA77yB,size_20,color_FFFFFF,t_70,g_se,x_16)  
![](https://img-blog.csdnimg.cn/ef521e74962345daaf4faaeadbbd2ff0.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bCP5LiA77yB,size_20,color_FFFFFF,t_70,g_se,x_16)  
5 秒之后，子进程变为僵尸状态。

[僵尸进程](https://so.csdn.net/so/search?q=%E5%83%B5%E5%B0%B8%E8%BF%9B%E7%A8%8B&spm=1001.2101.3001.7020)是什么
-------------------------------------------------------------------------------------------------------

是一个比较特殊的状态。当子进程退出并且父进程没有读取到子进程退出的返回代码时，就会产生僵死 (尸) 进程，  
僵死进程会以终止状态保持在进程表中，并且会一直在等待父进程读取退出状态代码。所以，只要子进程退出，父进程还在运行，但父进程没有读取子[进程状态](https://so.csdn.net/so/search?q=%E8%BF%9B%E7%A8%8B%E7%8A%B6%E6%80%81&spm=1001.2101.3001.7020)，子进程进入 Z 状态。

为什么要有僵尸进程
---------

因为我们必须得保证一个进程跑完，启动这个进程的父进程或是操作系统必须得知道这个进程退出时，把我们交代得任务完成得怎么样了，成功还是失败了。必须要知道子进程得运行结果。当子进程退出的时候，它的信息不会立即释放，会存在 PCB 中，没有人读取，这个状态不会被释放掉，这个状态就是僵尸状态。

僵尸进程的危害
-------

进程的退出状态必须被维持下去，因为他要告诉关心它的进程（父进程），你交给我的任务，我办的怎样了。可父进程如果一直不读取，那子进程就一直处于 Z 状态？**是的！**

维护退出状态本身就是要用数据维护，也属于进程基本信息，所以保存在 task_struct(PCB) 中，换句话说，Z 状态一直不退出，PCB 一直都要维护？**是的！**

那一个父进程创建了很多子进程，就是不回收，是不是就会造成内存资源的浪费？**是的！**

因为数据结构对象本身就要占用内存，想想 C 中定义一个结构体变量（对象），是要在内存的某个位置进行开辟空  
间！内存泄漏? **是的！**

X 死亡状态（dead）
============

这个状态只是一个返回状态，你不会在任务列表里看到这个状态。当父进程读取子进程的返回结果时，子进程立刻释放资源。死亡状态是非常短暂的，几乎不可能通过 ps 命令捕捉到。

孤儿进程
====

所谓孤儿进程，故名思义，和现实生活中的孤儿有点类似，当一个进程的父进程结束时，但是它自己还没有结束，那么这个进程将会成为孤儿进程。

孤儿进程会被 init 进程（1 号进程）的进程收养，当然在子进程结束时也会由 init 进程完成对它的状态收集工作，因此一般来说，孤儿进程并不会有什么危害.  
代码实现一下：

```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
int main()
{
pid_t id = fork();
if(id < 0){
perror("fork");
return 1;
}
else if(id == 0){//child
printf("I am child, pid : %d\n", getpid());
sleep(10);
}else{//parent
printf("I am parent, pid: %d\n", getpid());
sleep(3);
exit(0);
}
return 0;
}

```

![](https://img-blog.csdnimg.cn/c0fdb2da2c83481ebe86fca674fe768c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bCP5LiA77yB,size_20,color_FFFFFF,t_70,g_se,x_16)  
由图可以看出，当父进程结束时，父进程由 4416 变为 1 号进程，子进程 4417 被 1 号进程领养。