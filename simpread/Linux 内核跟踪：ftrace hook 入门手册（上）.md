> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/H8tpt7aWR6pvMAWi5-qVww)

一、什么是 ftrace

  

ftrace(FunctionTracer) 是 Linux 内核的一个跟踪框架，它从 2008 年 10 月 9 日发布的内核版本 2.6.27 开始并入 Linux 内核主线 [1]。官方文档 [2] 中的描述大致翻译如下：

> ftrace 是一个内部跟踪程序，旨在帮助系统的开发人员和设计人员弄清楚内核内部发生的情况。它可以用于调试或分析在用户空间之外发生的延迟和性能问题。虽然 ftrace 通常被认为是函数跟踪程序，但它实际上是几个不同的跟踪实用程序的框架。…

![](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUbia4u4qqaf3I1eh1iayveYhGA2EXBoAgKwoJVGMMaEPUia4xq6KTgWhP2icFAY49DrjS4XKBBibFQtNbw/640?wx_fmt=png)

图 1：ftrace 是一个功能强大的内核函数追踪框架 [3]

使用 ftrace 需要目标 Linux 操作系统在编译时启用 CONFIG_FUNCTION_TRACER 内核配置选项（该选项默认启用）。此时大部分非内联内核函数的开头会出现一个对 mcount 函数 (或__fentry__函数，若 gcc>=4.6 且为 x86 架构) 的调用。mcount 函数本身只是一个简单的返回指令，并没有什么实际意义，但动态 ftrace 框架会在启动时将所有对 mcount 的调用位置都填充为 nop 指令，这样一来就在这些内核函数的开头产生了足以容纳一个 call 指令的空白区。这个空白区可以在需要的时候被替换为对 ftrace 相关函数的调用，从而实现对特定内核函数的调用追踪，而不会过度影响其它内核函数的运行性能。

关于 ftrace 的详细内部机制，受限于篇幅，本文不详细介绍。但总之，通过 ftrace 框架，我们得以对大部分内核函数（尤其是各种系统调用）进行劫持，从而实现各种各样的主机侧访问控制功能。由于不同版本的 Linux 内核机制差异较大，笔者在多个不同版本的 CentOS 和 Ubuntu 环境中进行了测试。如果您在实践过程中遇到了其它环境适配的问题，不妨在评论区留言补充。

二、经典 Hook 方案

  

目前网络上大多数公开的 ftracehook 实现方案原理上大同小异。感兴趣的读者可以参考以下链接：

https://www.apriorit.com/dev-blog/546-hooking-linux-functions-2

https://xcellerator.github.io/posts/linux_rootkits_02/

https://github.com/ilammy/ftrace-hook/

![](https://mmbiz.qpic.cn/mmbiz_jpg/hiayDdhDbxUbia4u4qqaf3I1eh1iayveYhGs8lgjLXEy2u1zic4swRnicuwHyC1q2n4C0s3LRuqbyZs6QA8lxzWibBDg/640?wx_fmt=jpeg)

图 2：经典 ftrace hook 方案中的执行流程 [4]

适当建议有余力的读者首先了解一下上述经典方案，但跳过这个步骤并不会过多地影响您阅读本文的其它内容。

三、环境准备

  

开始前请注意，安装和卸载内核模块通常需要 root 权限。以下所有操作方法默认都是在 root 用户下进行的，如有需要请自行添加 sudo 或 su -c。

3.1 **安装编译环境**

通过 yum 源进行安装：

```
yum install gcc kernel-devel-$(uname -r)

```

成功安装后，会在 / usr/src/kernels / 目录内出现一个以当前内核版本和架构命名的子目录，内含大量的 C 语言头文件：

![](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUaD8KNyUrUFJ7zibuOFS64fSfTxJcVmo9qPYdtxickTHqk4MIIIgr69qsj6zXk4ic2zHxRXkjaQ4a2QQ/640?wx_fmt=png)

图 3：正确安装情况下的 kernels 目录

由于目前部分 Linux 内核函数 / 结构体的系统性文档比较少，必要时可以在这里直接阅读头文件源码。

另外推荐一个网站 https://elixir.bootlin.com/linux/latest/source，可以非常方便直观地阅读和搜索各个版本的 Linux 内核源码（该网站还有 glibc、grab 等源码，如果需要的话）。

3.2 **一个简单的内核模块**

要制作一个 Linux 内核模块，项目目录需要至少两个文件：一个. c 文件，一个 Makefile 文件：

![](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUaD8KNyUrUFJ7zibuOFS64fS4aTtGQib9Daud2q4Lz5Job3D05SKkaicpGKw9bUdcapLl2o2RyV9d1fw/640?wx_fmt=png)

图 4：一个最简单的 Linux 内核模块项目目录  

HelloWorld.c：

```
#include <linux/module.h>
static int __init Initialize(void)
{
    pr_info("Hello, world!\n");
    return 0;
}
static void __exit Finalize(void)
{
    pr_info("Bye, world!\n");
}
module_init(Initialize);
module_exit(Finalize);

```

内核模块并没有一般意义上的主函数，module_init 和 module_exit 分别设置了模块加载和卸载时所执行的函数。

需要注意，内核模块应当尽量实现并设置 module_init 和 module_exit 函数，即使它们不包含实质上的业务逻辑。虽然不设置它们也可以正常构建得到. ko 文件，但这可能产生一些预期之外的问题（例如，一个不定义 / 不设置 module_exit 函数的内核模块，可能无法被正常卸载）。

Makefile：

```
obj-m += HelloWorld.o
all:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

```

上文中的文件名前缀必须与. c 文件一致（严格来说，是必须与 gcc 编译所产生的. o 文件名一致）。如果源文件位于子目录内，此处也需要加上目录前缀。

接下来我们切换到项目目录内，执行构建：

```
make

```

正常运行会得到如下结果：

![](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUbia4u4qqaf3I1eh1iayveYhGyFn2tjiaxiaeYzlsq8TngQEzsW29aibjTJibxaI7KSyicYIOFEWPlZ1icia2Q/640?wx_fmt=png)

图 5：构建命令输出

此时应该会产生一个. ko 文件，就是我们刚刚制作的内核模块的可执行文件了：

![](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUbia4u4qqaf3I1eh1iayveYhGZdPTIPbuYELGicdwibzH2cGibLc5MaAPfZyB9tcvCdOUVUbmDJCpwp8lA/640?wx_fmt=png)

图 6：构建完毕的内核模块  

接下来我们安装这个新的内核模块：

```
insmod HelloWorld.ko

```

这个命令正常运行时不会产生任何输出。

随后，我们可以列出内核模块：

```
lsmod

```

如果此前已经安装成功，应该可以在列表中看到它：

![](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUbia4u4qqaf3I1eh1iayveYhGwk9NjZUlqqlmia1RoRict9VHKsFEH6xCiatPT7Ibft6UUIPcdnRBSyhUw/640?wx_fmt=png)

图 7：列出内核模块

类似地，我们也可以卸载已安装的内核模块：

```
rmmod HelloWorld

```

这个命令正常运行时也不会产生任何输出。

特别注意，这个命令中并不包含 “.ko” 后缀，也不要求必须在项目目录内执行。此外，一个正在使用中的内核模块是不能被卸载的（比如，某个用户进程打开了一个通往该内核模块的 Netlink 连接）。

那么，此前代码中通过 pr_info 输出的信息跑到哪里去了呢？答案是位于 Linux 内核中的环缓冲区（ring buffer）。我们可以通过下面的命令访问它：

```
dmesg            /*一次性打印整个缓冲区*/
dmesg --follow    /*持续打印缓冲区，直到Ctrl+C中断*/
dmesg --clear     /*清空缓冲区*/

```

就可以看到我们的模块此前在加载和卸载时所产生的输出信息了：

![](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUbia4u4qqaf3I1eh1iayveYhGW00sfqIwkeHVnAuBpsWO9Weibf302eHT57aPnVon7SKSyicgicJ4ngbzA/640?wx_fmt=png)

图 8：查看调试输出

除了 dmesg 命令外，您也可以在 / var/log/messages 文件中找到这些输出。

至此，我们就实现了一个简单的内核模块。

3.3 **在内核模块中包含多个源文件**

实际操作中，我们的项目可能同时包含多个. c 文件，例如这样：

![](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUbia4u4qqaf3I1eh1iayveYhGdnnucTeZ1dWic2sSUf5cfsvqiac95P6RicWRaJibrKPqt04b4qDRokw8Xw/640?wx_fmt=png)

图 9：包含多个源文件的内核模块项目

entry.c：

```
# include "function.h"
static int __init Initialize(void)
{
    pr_info("3+5=%d!\n",Add(3,5));
    return 0;
}
static void __exit Finalize(void)
{
}
module_init(Initialize);
module_exit(Finalize);

```

function.c:  

```
#include "function.h"
int Add(int a,int b)
{
    return a+b;
}

```

function.h:

```
#ifndef LIB_FUNCTION
#define LIB_FUNCTION
#include <linux/module.h>
int Add(int a,int b);
#endif

```

以上三个文件的内容都没有什么特别之处。但下面的 Makefile 文件需要进行一些特别的调整。

Makefile：

```
obj-m += MultipleCFiles.o
MultipleCFiles-objs := entry.o function.o
all:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

```

接下来在工作目录内正常使用 make 命令进行构建，即可得到 MultipleCFiles.ko：

![](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUbia4u4qqaf3I1eh1iayveYhGad6XGNJTIXNLqexQlLYyve1KBMwI8qdcUtibPIoGOctopib2l1YicPwDQ/640?wx_fmt=png)

图 10：多个源文件构建内核模块的运行结果

此处需要注意以下三点：

1、Makefile 第一行 “obj-m” 后面的应当是一个**不**存在对应. c 文件的名称，它将成为最终构建输出的. ko 文件的名称。如果使用实际存在的. c 文件的名称，make 命令虽然也可能不报错，但产生的. ko 模块会无法正常运行；

2、Makefile 第二行 “MultipleCFiles-objs” 中“-objs”前面的部分应当与第一行中配置的名称一致，否则 make 命令会报错而无法生成. ko 模块；

3、如果希望将函数的声明和定义分别放置在. h 文件和. c 文件中（就像上面例子中的 Add 函数一样），那么该函数应当**不**加 static 修饰，否则它们无法被编译器正确链接起来。此时虽然能够产生. ko 模块但可能无法正常运行：

![](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUbia4u4qqaf3I1eh1iayveYhGgpicSjyxMs1o7at7Rklr7TZ5uGibMk9QQlSaxxgbHCKonia1uqzPxp88Q/640?wx_fmt=png)

图 11：不正确的函数 static 修饰导致模块无法安装

四、Hook 案例完整源代码

  

FTraceHook.h：

```
#ifndef LIB_FTRACE_HOOK
#define LIB_FTRACE_HOOK
#include <linux/version.h>
#include <linux/ftrace.h>
#include <linux/kprobes.h>
#include <linux/kallsyms.h>
#include <asm/unwind.h>
//关于系统调用符号解析的版本差异处理
#if LINUX_VERSION_CODE>=KERNEL_VERSION(5,7,0)
static size_t kallsyms_lookup_name(const char *name)
{
    struct kprobe kp = { .symbol_name = name };
    size_t retval;
    if (register_kprobe(&kp) < 0)
        return 0;
    retval = (size_t)kp.addr;
    unregister_kprobe(&kp);
    return retval;
}
#endif
//关于ftrace框架的版本差异处理
#if LINUX_VERSION_CODE<KERNEL_VERSION(5,11,0)
#define FTRACE_OPS_FL_RECURSION 0
#define ftrace_regs pt_regs
static __always_inline struct pt_regs *ftrace_get_regs(struct ftrace_regs *fregs)
{
    return fregs;
}
#endif
//关于系统调用函数签名的版本差异处理
#if defined(CONFIG_X86_64)&&(LINUX_VERSION_CODE>=KERNEL_VERSION(4,17,0))
#define PTREGS_SYSCALL_STUBS 1
#define SYSCALL_NAME(name) ("__x64_" name)
#else
#define PTREGS_SYSCALL_STUBS 0
#define SYSCALL_NAME(name) (name)
#endif
//关于ret指令机器码的架构差异处理
#if defined(CONFIG_X86_64)||defined(CONFIG_X86_32)
#define RET_CODE 0xC3
#else
#error Unsupported architecture config?
#endif
struct FTraceHook;
struct FTraceHookContext;
struct FTraceHook
{
    const char *Symbol;
    bool (*Handler)(struct FTraceHookContext *);
    size_t SysCallEntry;
    struct ftrace_ops FTraceOPS;
};
struct FTraceHookContext
{
    struct FTraceHook *const Hook;
    struct pt_regs *const KernelRegisters;
    struct pt_regs *const UserRegisters;
    size_t *const SysCallNR;
    const size_t *const Arguments[6];
    size_t *const ReturnValue;
};
#define FTRACE_HOOK(_symbol,_handler) {.Symbol=SYSCALL_NAME(_symbol),.Handler=(_handler)}
struct pt_regs *GetUserRegisters(struct task_struct *task);
size_t FTraceHookCallOriginal(struct FTraceHookContext *context);
int FTraceHookInstall(struct FTraceHook *hook);
int FTraceHookUninstall(struct FTraceHook *hook);
int FTraceHookInitialize(struct FTraceHook *hooks, size_t hooks_size);
int FTraceHookFinalize(struct FTraceHook *hooks, size_t hooks_size);
#endif

```

FTraceHook.c:  

----------------

```
#include "FTraceHook.h"
static size_t RET_ADDRESS;
//获取用户线程原本的寄存器保存位置
struct pt_regs *GetUserRegisters(struct task_struct *task)
{
    //用户线程原本的寄存器保存在内核栈地址最高处，为pt_regs结构体
    //实际上，我们只是想要unwind_state里面的stack_info，但有些编译环境中不知为何会报错：ERROR: modpost: "get_stack_info" undefined!，有知情的读者还请不吝赐教，非常感谢
    struct unwind_state state;
    task = task ? : current;
    unwind_start(&state, task, NULL, NULL);
    return (struct pt_regs *)(((size_t)state.stack_info.end) - sizeof(struct pt_regs));
}
//代理调用原始函数
size_t FTraceHookCallOriginal(struct FTraceHookContext *context)
{
#if PTREGS_SYSCALL_STUBS
    return ((asmlinkage size_t (*)(struct pt_regs *)) context->Hook->SysCallEntry)(context->UserRegisters);
#else
    //asmlinkage的参数，传多了似乎没什么影响，有switch的功夫还不如多push几个参数咧
    return ((asmlinkage size_t (*)(size_t, size_t, size_t, size_t, size_t, size_t)) context->Hook->SysCallEntry)(*context->Arguments[0], *context->Arguments[1], *context->Arguments[2], *context->Arguments[3], *context->Arguments[4], *context->Arguments[5]);
#endif
}
static void notrace FTraceHookHandler(size_t ip, size_t parent_ip, struct ftrace_ops *ops, struct ftrace_regs *fregs)
{
    struct pt_regs *kernel_regs = ftrace_get_regs(fregs);
    struct pt_regs *user_regs = GetUserRegisters(NULL);
#if PTREGS_SYSCALL_STUBS
#define argument_regs user_regs
#else
#define argument_regs kernel_regs
#endif
#if defined(CONFIG_X86_64)
#define INSTRUCTION_POINTER kernel_regs->ip
    struct FTraceHookContext context =
    {
        .Hook = container_of(ops, struct FTraceHook, FTraceOPS),
        .KernelRegisters = kernel_regs,
        .UserRegisters = user_regs,
        .SysCallNR = &argument_regs->ax,
        .Arguments =
        {
            &argument_regs->di,
            &argument_regs->si,
            &argument_regs->dx,
            &argument_regs->r10,
            &argument_regs->r8,
            &argument_regs->r9
        },
        .ReturnValue = &argument_regs->ax
    };
#elif defined(CONFIG_X86_32)
#define INSTRUCTION_POINTER kernel_regs->ip
    struct FTraceHookContext context =
    {
        .Hook = container_of(ops, struct FTraceHook, FTraceOPS),
        .KernelRegisters = kernel_regs,
        .UserRegisters = user_regs,
        .SysCallNR = &argument_regs->ax,
        .Arguments =
        {
            &argument_regs->bx,
            &argument_regs->cx,
            &argument_regs->dx,
            &argument_regs->si,
            &argument_regs->di,
            &argument_regs->bp
        },
        .ReturnValue = &argument_regs->ax
    };
#else
#error Unsupported architecture config?
#endif
    if (!context.Hook->Handler(&context)) //返回false则阻止原始函数执行（直接返回到原始函数的调用方），其余情况不需要特殊操作，任由ftrace框架恢复执行流程即可
        INSTRUCTION_POINTER = RET_ADDRESS;
}
int FTraceHookInstall(struct FTraceHook *hook)
{
    int err;
    //使用kallsyms_lookup_name()在内核内存中查找地址。
    hook->SysCallEntry = kallsyms_lookup_name(hook->Symbol);
    if (!hook->SysCallEntry)
    {
        pr_err("[FTraceHook] Unresolved symbol: %s\n", hook->Symbol);
        return ENOENT;
    }
    //我们的hook子程并不是通过修改ip跳转过去的，可以使用ftrace自带的防递归，而且实测效率还不错
    hook->FTraceOPS.func = FTraceHookHandler;
    hook->FTraceOPS.flags = FTRACE_OPS_FL_SAVE_REGS | FTRACE_OPS_FL_IPMODIFY | FTRACE_OPS_FL_RECURSION;
    err = ftrace_set_filter_ip(&hook->FTraceOPS, hook->SysCallEntry, 0, 0);
    if (err)
    {
        pr_err("[FTraceHook] ftrace_set_filter_ip() failed: %d\n", err);
        return err;
    }
    err = register_ftrace_function(&hook->FTraceOPS);
    if (err)
    {
        pr_err("[FTraceHook] register_ftrace_function() failed: %d\n", err);
        return err;
    }
    pr_info("[FTraceHook] Installed hook '%s': %d\n", hook->Symbol, err);
    return err;
}
int FTraceHookUninstall(struct FTraceHook *hook)
{
    int err;
    //注意与安装过程相反的顺序
    err = unregister_ftrace_function(&hook->FTraceOPS);
    if (err)
    {
        pr_err("[FTraceHook] unregister_ftrace_function() failed: %d\n", err);
        return err;
    }
    err = ftrace_set_filter_ip(&hook->FTraceOPS, hook->SysCallEntry, 1, 0);
    if (err)
    {
        pr_err("[FTraceHook] ftrace_set_filter_ip() failed: %d\n", err);
        return err;
    }
    pr_info("[FTraceHook] Uninstalled hook '%s': %d\n", hook->Symbol, err);
    return err;
}
int FTraceHookInitialize(struct FTraceHook *hooks, size_t hooks_size)
{
    int err = 0;
    size_t i;
    //随便找一个ret指令的地址，基本上就用当前函数尾部的ret就好；如果求稳（比如担心当前函数内存在复杂的跳转等），可以另外定义一个空函数，注意避免选取内联函数
    RET_ADDRESS = (size_t)FTraceHookInitialize;
    while (* (unsigned char *) RET_ADDRESS != RET_CODE)
        ++RET_ADDRESS;
    //安装钩子
    for (i = 0; i < hooks_size && !err; ++i)
        err = FTraceHookInstall(hooks + i);
    return err;
}
int FTraceHookFinalize(struct FTraceHook *hooks, size_t hooks_size)
{
    int err = 0;
    size_t i;
    for (i = 0; i < hooks_size; ++i)
        err = FTraceHookUninstall(hooks + i);
    return err;
}

```

Entry.c:
--------

```
#include <linux/kernel.h>
#include <linux/module.h>
#include "FTraceHook.h"
MODULE_LICENSE("GPL");//使用ftrace的模块必须是GPL License，不然不能编译
MODULE_VERSION("0.01");
static int ReturnSwitch = 0;
static bool MySysExecve(struct FTraceHookContext *context)
{
    char filename[256];//用kmalloc+kfree也是可以的，但栈容量允许的情况下，时间效率还是局部变量比较快
    //输出第一个参数值
    if (strncpy_from_user(filename, (char *)*context->Arguments[0], sizeof(filename)) >= 0)
        pr_info("[FTraceHook] PID %u calling sys_execve: %s\n", current->pid, filename);
    //轮流测试两种方法来恢复原系统调用流程
    if (++ReturnSwitch % 2)
    {
        //模拟经典方案的机制，代理调用原始函数
        *context->ReturnValue = FTraceHookCallOriginal(context);
        pr_info("[FTraceHook] execve() return: %ld\n", *context->ReturnValue);
        return false;//中止系统调用
    }
    else
    {
        //优化方案的新机制，不重新push参数而直接恢复原系统调用流程，但此时无法获取原系统调用流程的返回值
        return true;
    }
}
static struct FTraceHook GlobalHooks[] =
{
    FTRACE_HOOK("sys_execve", MySysExecve)
};
static int __init Initialize(void)
{
    int err = FTraceHookInitialize(GlobalHooks, ARRAY_SIZE(GlobalHooks));
    if(err)
        pr_err("[FTraceHook] Failed to initialize ftrace hooks...\n");
    return err;
}
static void __exit Finalize(void)
{
    if(FTraceHookFinalize(GlobalHooks, ARRAY_SIZE(GlobalHooks)))
        pr_err("[FTraceHook] Failed to finalize ftrace hooks...\n");
}
module_init(Initialize);
module_exit(Finalize);

```

Makefile:  

```
obj-m += FTraceHookExample.o
FTraceHookExample-objs := Entry.o FTraceHook.o
all:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

```

运行效果：

![](https://mmbiz.qpic.cn/mmbiz_png/hiayDdhDbxUbia4u4qqaf3I1eh1iayveYhG1h3mNzcBCXUgJYCpibrdtt7WLZCRwKoyJLk9qJPQGKqCgLmnGp78RiaA/640?wx_fmt=png)

图 12：完整运行效果展示

五、对经典方案的优化

  

实际上，上面的代码也参考自经典方案 [4][5][6] 中的内容，但进行了一些优化，比较重要的部分包括：

1.  由于 Linux 内核 4.17 版本前后的系统调用函数签名不同，经典方案中需要通过条件编译的方式为每个 hook 定义两个功能相同但签名不同的 hook 子程：
    

1.  如果 hook 子程本身逻辑简单、数量少倒还好，否则每个 hook 子程都要写两份，实际使用中非常不便；
    

3.  经典方案（至少前述三个参考链接实现）中大多没有考虑不同位宽 / 架构的寄存器差异：
    

1.  例如，hook 子程中始终读取 pt_regs 的成员 di 作为第一个参数，这在 x86_64 架构下没有问题，但对 x86_32（应为 bx）、arm（应为 uregs[0]）等架构则是不适用的；
    
2.  但是，如果在每个 hook 子程中都进行条件编译，实际使用中也是非常不便的；
    

5.  经典方案通过在回调函数（ftrace_set_filter_ip 的第二个参数）中修改原始系统调用的 ip（x86 架构的指令指针寄存器，不是网际协议，后文皆同不再另行说明）来使得执行流程跳转到 hook 子程，因此 hook 子程的函数签名必须与原始系统调用一致：
    

1.  这导致 hook 子程中需要系统调用参数之外的信息时逻辑变得很复杂，hook 子程通常需要提前将一些必要的信息（例如，原始函数的真实地址等）通过另外的机制保存或传递，而无法封装框架统一提供；
    
2.  除此之外，对多个不同的系统调用使用同一个 hook 子程也会比较麻烦（因为不易确定原始系统调用函数的地址以进行代理，可能需要通过系统调用号重新查表等），尤其是对于业务上希望监控大量系统调用的场景；
    
3.  如果 hook 子程中需要调用原始函数，通常需要将调用参数重新入栈（Linux 系统调用多有 asmlinkage 修饰，即全部参数通过堆栈传递而不使用 fastcall），而不易让执行流程直接恢复到原始函数中。这将导致少许额外开销，尤其是对于 4.17 以前的内核版本。
    

7.  修改 ip 的跳转方法导致经典方案中对 hook 子程的执行发生在 ftrace 相关函数返回之后（而非 ftrace 相关函数栈内），因此 ftrace 自带的防递归功能无法作用于经典方案。为此，经典方案中自行实现了两套防递归方法，但它们看上去都不是非常完善：
    

1.  第一种方法通过 within_module 检查直接调用方是否位于当前模块中，这对于涉及多个模块的调用（hook 子程位于模块 A 中，A 调用了模块 B 的函数，而模块 B 尝试调用被 hook 的原始函数）可能是不完善的；
    
2.  第二种方法在执行递归调用时跳过系统调用开头的 “空白区”，这意味着需要对于所有调用原始函数的代码进行修改。这不仅实现起来比较麻烦，而且同样面临多个模块间调用时可能不完善的问题（因为难以修改其它模块对原始函数的调用）；
    

9.  （实现细节）关于 FTRACE_OPS_FL_RECURSION 标志的使用可能有误
    

1.  ftrace 框架的防递归选项在内核版本 5.11 前后发生了变化：
    

1.  在 5.10.113 及以前版本中，默认会添加防递归检查，除非 ftrace_ops.flags 设置了 FTRACE_OPS_FL_RECURSION_SAFE 标志；
    
2.  而从 5.11rc1 开始，默认不会添加防递归检查，除非 ftrace_ops.flags 设置了 FTRACE_OPS_FL_RECURSION 标志；
    

3.  因此，这实际上是两个功能相反的标志选项。第一个经典方案 [4] 中 “#define FTRACE_OPS_FL_RECURSIONFTRACE_OPS_FL_RECURSION_SAFE” 的写法可能是不正确的，而第二个经典方案 [5] 没有对这个标志进行版本差异处理；  
    
4.  不过，因为经典方案并不需要 ftrace 框架提供防递归检查，所以这个错误应该不会造成什么实质上的影响。
    

连同这些优化的细节在内，本系列的下一篇文章中会着重讲解上述代码实现的各个细节原理。

六、后记

  

更多前沿资讯，还请继续关注绿盟科技研究通讯。

如果您发现文中描述有不当之处，还请留言指出。在此致以真诚的感谢~

  

---

**参考文献**
--------

1.  KERNELNEWBIES.ORG. Linux kernel 2.6.27 [J/OL]2008,
    
    https://kernelnewbies.org/Linux_2_6_27.
    
2.  ROSTEDT S. ftrace - Function Tracer [J/OL]2008,
    
    https://www.kernel.org/doc/Documentation/trace/ftrace.txt.
    
3.  YEMELIANOV A. Kernel Tracing with Ftrace[J/OL] 2017,
    
    https://blog.selectel.com/kernel-tracing-ftrace/.
    
4.  ALEXEY LOZOVSKY S S. Hooking Linux KernelFunctions, Part 2: How to Hook Functions with FtraceIt [J/OL] 2018,
    
    https://www.apriorit.com/dev-blog/546-hooking-linux-functions-2.
    
5.  PHILLIPS H. Linux Rootkits Part 2: Ftrace and Function Hooking [J/OL] 2020,
    
    https://xcellerator.github.io/posts/linux_rootkits_02/.
    
6.  OLEKSII LOZOVSKYI M G, KRZYSZTOF ZDULSKI.ftrace-hook [J/OL] 2021,
    
    https://github.com/ilammy/ftrace-hook/.
    

内容编辑：天枢实验室 吴复迪

责任编辑：董炳佑

本公众号原创文章仅代表作者观点，不代表绿盟科技立场。所有原创内容版权均属绿盟科技研究通讯。未经授权，严禁任何媒体以及微信公众号复制、转载、摘编或以其他方式使用，转载须注明来自绿盟科技研究通讯并附上本文链接。

**关于我们**

  

绿盟科技研究通讯由绿盟科技创新中心负责运营，绿盟科技创新中心是绿盟科技的前沿技术研究部门。包括云安全实验室、安全大数据分析实验室和物联网安全实验室。团队成员由来自清华、北大、哈工大、中科院、北邮等多所重点院校的博士和硕士组成。  

绿盟科技创新中心作为 “中关村科技园区海淀园博士后工作站分站” 的重要培养单位之一，与清华大学进行博士后联合培养，科研成果已涵盖各类国家课题项目、国家专利、国家标准、高水平学术论文、出版专业书籍等。

我们持续探索信息安全领域的前沿学术方向，从实践出发，结合公司资源和先进技术，实现概念级的原型系统，进而交付产品线孵化产品并创造巨大的经济价值。

![](https://mmbiz.qpic.cn/mmbiz_jpg/hiayDdhDbxUbrbTJxY0Qv9BtgtXZsYVvaVUtlPicCUV6qDBGgZnrxicAMwvibG73JUu0w1UweTicfkuTRIyJyt77C5Q/640.jpeg?wx_fmt=jpeg)

**长按上方二维码，即可关注我**