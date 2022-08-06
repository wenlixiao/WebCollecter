> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/28680568)

**摘要**_：_ SystemTap 是一个 Linux 非常有用的调试（跟踪 / 探测）工具，常用于 Linu 内核或者应用程序的信息采集，本篇文章介绍其原理、安装、入门、脚本语言及技巧，由阿里云 CDN 安防专家金九撰写。

**1. 简介**
---------

![](https://pic2.zhimg.com/v2-828fe7b70a9635084c558494dc85b189_r.jpg)

**2. 何时使用**
-----------

![](https://pic1.zhimg.com/v2-fc6ce5eac9ba8d2f1afc64a17af18a84_r.jpg)

**3. 原理**
---------

在网上找了个原理图：

![](https://pic4.zhimg.com/v2-dec55ad7146faa4b0d7700f2d31696b7_r.jpg)

SystemTap 的处理流程有 5 个步骤：解析 script 文件 (parse)、细化（elaborate）、script 文件翻译成 C 语言代码（translate）、编译 C 语言代码（生成内核模块）（build）、加载内核模块（run）

![](https://pic1.zhimg.com/v2-e267fbb7b7c58417e7c4623934a9cd6c_r.jpg)

**4. 安装**
---------

SystemTap 依赖的 package：

elfutils、gcc、kernel-devel、kernel-debuginfo

如果调用用户态进程，还需要该程序有调试符号，否则无法调试。

推荐使用最新稳定版的 SystemTap，目前最新稳定版为：systemtap-2.9.tar.gz

**5. 入门**
---------

**5.1 stap 命令**

![](https://pic3.zhimg.com/v2-ae3e98a92a81443320527875fbaeb98e_r.jpg)

Hello World:

![](https://pic4.zhimg.com/v2-13360c90429430d51d2cdf78821b9bc3_r.jpg)

**5.2 staprun 命令**

![](https://pic3.zhimg.com/v2-9b9bcc9853b252fc7f99624a698c30fe_r.jpg)

stap 命令与 staprun 命令的区别在于：

stap 命令的操作对象是 stp 文件或 script 命令等，而 staprun 命令的操作对象是编译生成的内核模块。

**6. 脚本语言**
-----------

**6.1 probe**

“probe” <=> “探测”, 是 SystemTap 进行具体地收集数据的关键字。

![](https://pic4.zhimg.com/v2-c484c897a84e455d0c93ef592be22143_r.jpg)

“probe point” 是 probe 动作的时机，也称探测点。也就是 probe 程序监视的某事件点，一旦侦测的事件触发了，则 probe 将从此处插入内核或者用户进程中。

“probe handle” 是当 probe 插入内核或者用户进程后所做的具体动作。

probe 用法：

![](https://pic1.zhimg.com/v2-71f0e404be2205c08cf66e72ad5e67d0_r.jpg)

在 Hello World 例子中 begin 和 end 就是 probe-point， statement 就是该探测点的处理逻辑，在 Hello World 例子中 statement 只有一行 print，statement 可以是复杂的代码块。

探测点语法：

![](https://pic3.zhimg.com/v2-05b57212ad83e137acb07fdc1676768e_r.jpg)

PATTERN 语法为：

![](https://pic4.zhimg.com/v2-5e7eed7cab210c56b9bb827cc58193a7_r.jpg)

例如：

![](https://pic3.zhimg.com/v2-d5187d44aafbf420e1c7cf99205b1a1e_r.jpg)

在 return 探测点可以用 \$return 获取该函数的返回值。

inline 函数无法安装. return 探测点，也无法用 $return 获取其返回值。

**6.2 基本语法**

SystemTap 脚本语法比较简单，与 C 语言类似，只是每一行结尾 ";" 是可选的。主要语句如下：

if/else、while、for/foreach、break/continue、return、next、delete、try/catch

其中：

next：主要在 probe 探测点逻辑处理中使用，调用此语句时，立刻从调用函数中退出。不同于 exit() 的是，next 只是退出当前的调用函数，而此 SystemTap 并没有终了，但 exit() 则会终止 SystemTap。

**6.2.1 变量**

不需要明确声明变量类型，脚本语言会根据函数参数等自动判断变量是什么类型的。

局部变量：在声明的 probe 和 block（”{ }“范围内的部分）内有效。

全局变量：用”global“声明的变量，在此 SystemTap 的整个动作过程中都有效。全局变量的声明位置没有具体要求。需要注意的是，全局变量默认有锁保护，使用过多会有性能损失，如果用全局变量保存指针，可能出现指针所指的内容被进程修改，在探测点中拿不到真正的数据。

获取进程中的变量（全局变量、局部变量、参数）直接在变量名前面加 $ 即可（后面会有例子）

**6.2.2 注释**

![](https://pic1.zhimg.com/v2-72842353178bdb76f703994bac53c6ac_r.jpg)

**6.2.3 操作符**

比较运算符、算数运算符基本上与 C 语言一样，需要特别指出的是：

(1)、. 操作符：连接两个字符串，类似于 php；

(2)、=~ 和!~：正则匹配和正则不匹配；

**6.2.4 函数**

函数定义例子：

![](https://pic4.zhimg.com/v2-aa4559f6404a41ee679c53cd79b1805b_r.jpg)

官方有很多很有用的函数，详情请参考：[https://sourceware.org/systemtap/tapsets/](https://link.zhihu.com/?target=https%3A//sourceware.org/systemtap/tapsets/%3Fspm%3D5176.100239.blogcont174916.17.632Iqx)

以及在本机安装了 SystemTap 之后在目录 / usr/local/share/systemtap/tapset / 下也可以看具体函数的实现以及一些奇特的用法。

**7. 技巧**
---------

**7.1 定位函数位置**

在一个大型项目中找出函数在哪里定义有时很有用，特别是一些比较难找出在哪里定义的函数，比如内核或者 glibc 中的某个函数想要看其实现时，首先得找出其在哪个文件的哪一行定义，用 SystemTap 一行命令就可以搞定。

比如要看 printf 在 glibc 中哪里定义的：

![](https://pic1.zhimg.com/v2-6f3cab6c2d3b5a15e5698fb73b515298_r.jpg)

可以看出 printf 是在 printf.c 第 29 行定义的。

再比如要看内核中 recv 系统的调用是在哪里定义的：

![](https://pic1.zhimg.com/v2-f6a951289d65b335ce82b69ef6388494_r.jpg)

可以看出 recv 是在 socket.c 第 1868 行定义的。

甚至可以 * 号来模糊查找：

![](https://pic1.zhimg.com/v2-8c4380d84922bf64ce8aab6eabb00218_r.jpg)

同理，也可以用来定位用户进程的函数位置：

比如 tengine 的文件 ngx_shmem.c 里面为了兼容各个操作系统而实现了三个版本的 ngx_shm_alloc，用 #if (NGX_HAVE_MAP_ANON)、#elif (NGX_HAVE_MAP_DEVZERO)、#elif (NGX_HAVE_SYSVSHM)、#endif 来做条件编译，那怎么知道编译出来的是哪个版本呢，用 SystemTap 的话就很简单了，否则要去 grep 一下这几宏有没有定义才知道了。

![](https://pic4.zhimg.com/v2-2c766d258baab073241a117ebde50d43_r.jpg)

**7.2 查看可用探测点以及该探测点上可用的变量**

在一些探测点上能获取的变量比较有限，这是因为这些变量可能已经被编译器优化掉了，优化掉的变量就获取不到了。一般先用 - L 参数来看看有哪些变量可以直接使用：

![](https://pic4.zhimg.com/v2-61dd72bb0a91730782c0e0610590e09f_r.jpg)

可见在该探测点上可以直接使用 $shm 这个变量，其类型是 ngx_shm_t*。

statement 探测点也类似：

![](https://pic2.zhimg.com/v2-bf884076114a31d25fa117d620604e21_r.jpg)

**7.3 输出调用堆栈**

用户态探测点堆栈：print_ubacktrace()、sprint_ubacktrace()

内核态探测点堆栈：print_backtrace()、sprint_backtrace()

不带 s 和带 s 的区别是前者直接输出，后者是返回堆栈字符串。

这几个函数非常有用，在排查问题时可以根据一些特定条件来过滤函数被执行时是怎么调用进来的，比如排查 tengine 返回 5xx 时的调用堆栈是怎样的：

![](https://pic3.zhimg.com/v2-18881c1e68256fe781206916a993493a_r.jpg)

比如看看内核是怎么收包的：

![](https://pic3.zhimg.com/v2-d3c908cbedf97de112acb49d5e1aeb2e_r.jpg)

**7.4 获取函数参数**

一些被编译器优化掉的函数参数用 - L 去看的时候没有找到，这样的话在探测点里面也不能直接用 $ 方式获取该参数变量，这时可以使用 SystemTap 提供的 *_arg 函数接口，* 是根据类型指定的，比如 pointer_arg 是获取指针类型参数，int_arg 是获取整型参数，类似的还有 long_arg、longlong_arg、uint_arg、ulong_arg、ulonglong_arg、s32_arg、s64_arg、u32_arg、u64_arg：

![](https://pic3.zhimg.com/v2-d4d4a691856b6dbd628bcf6266df181a_r.jpg)![](https://pic3.zhimg.com/v2-107b3b72925b3ba90d41d87835eac156_r.jpg)

再比如两个函数的函数参数类型兼容也可以使用这种方法获取：

![](https://pic4.zhimg.com/v2-57f972336a3bcb32166ddfda6c0e441f_r.jpg)

这两个函数的参数完全兼容，只是第二个参数命名不一样而已，可以像下面这么用：

![](https://pic2.zhimg.com/v2-82c4acfd5c5afa778be50ac34f9d995d_r.jpg)

**7.5 获取全局变量**

有时候用 $ 可以直接获取到全局变量，但有时候又获取不到，那可以试试 @var：

比如获取 nginx 的全局变量 ngx_cycyle：

![](https://pic4.zhimg.com/v2-0eaea34f42470a60327196f1bfeb698f_r.jpg)

**7.6 获取数据结构成员用法**

![](https://pic1.zhimg.com/v2-df8769598835f51d58a8f4e2154d0414_r.jpg)

上面这个是 nginx 里面的 http 请求结构里面的几个成员，在 C 语言里，如果 r 是 struct ngx_http_request_t *，那么要获取 uri 的 data 是这样的：r->uri.data，但在 SystemTap 里面，不管是指针还是数据结构，都是用 -> 访问其成员：

![](https://pic3.zhimg.com/v2-008d1ef5eb8888a24757e29cae2acb4e_r.jpg)

**7.7 输出整个数据结构**

SystemTap 有两个语法可以输出整个数据结构：在变量的后面加一个或者两个

$ 即可，例子如下：

![](https://pic4.zhimg.com/v2-28873865111f837b0ce6fedeceb41eef_r.jpg)

其中 r->pool 的结构如下：

![](https://pic1.zhimg.com/v2-fbff482d2bab73683f7d765e01bed088_r.jpg)

ngx_pool_s 包含了结构 ngx_pool_data_t。变量后面加和 $ 的区别是后者展开了里面的结构而前者不展开，此用法只输出基本数据类型的值。

**7.8 输出字符串指针**

用户态使用：user_string、user_string_n

内核态使用：kernel_string、kernel_string_n、user_string_quoted

![](https://pic2.zhimg.com/v2-c8a605c6f3008eda890103924795cc89_r.jpg)

user_string_quoted 是获取用户态传给内核的字符串，代码中一般有__user 宏标记：

![](https://pic1.zhimg.com/v2-fb3d8f3b9d894aa0ae6a587986ee6dd4_r.jpg)![](https://pic1.zhimg.com/v2-f62e30454a786bccbb10699f7280a208_r.jpg)

**7.9 指针类型转换**

SystemTap 提供 @cast 来实现指针类型转换，比如可以将 void * 转成自己需要的类型：

![](https://pic4.zhimg.com/v2-fa5964a0e2a89acf5ee0e2a82354b77f_r.jpg)![](https://pic3.zhimg.com/v2-8e5fe57333c6b974e47f1b6ded5a510e_r.jpg)![](https://pic3.zhimg.com/v2-621f9b37b41c8e748a610d3b4363e3ea_r.jpg)

**7.10 定义某个类型的变量**

同样是用 @cast，定义一个变量用来保存其转换后的地址即可，用法如下：

![](https://pic4.zhimg.com/v2-acd0ab880d482e4dd1d281dacd6324db_r.jpg)

**7.11 多级指针用法**

![](https://pic4.zhimg.com/v2-38851aa79c9dc9570e86e408a8610b7b_r.jpg)

简言之：通过 [0] 去解引用即可。

**7.12 遍历 C 语言数组**

下面是在 nginx 处理请求关闭时遍历请求头的例子：

![](https://pic3.zhimg.com/v2-d3d1cd541378fbb2c8d94c320ef7723a_r.jpg)

**7.13 查看函数指针所指的函数名**

获取一个地址所对应的符号：

用户态：usymname

内核态：symname

![](https://pic3.zhimg.com/v2-cd63e9a9af1623db7ea6414d1acac812_r.jpg)

**7.14 修改进程中的变量**

![](https://pic4.zhimg.com/v2-16f6d0f056e4ddca3755ca7342a77fc7_r.jpg)![](https://pic3.zhimg.com/v2-1fcf44ac5bf494e129f5109ce8602202_r.jpg)

可以看出在第 17 行用 SystemTap 修改后的值在第 19 行就生效了。

需要注意的是 stap 要加 - g 参数在 guru 模式下才能修改变量的值。

**7.15 跟踪进程执行流程**

thread_indent(n): 补充空格

ppfunc(): 当前探测点所在的函数

在 call 探测点调用 thread_indent(4) 补充 4 个空格，在 return 探测点调用 thread_indent(-4) 回退 4 个空格，效果如下：

![](https://pic2.zhimg.com/v2-db315986a30de64880f8b04f5744ff2d_r.jpg)![](https://pic4.zhimg.com/v2-e03f8a243aaad9eedc837ed9aff5beb3_r.jpg)![](https://pic1.zhimg.com/v2-98976c01bba2b21bcab9192b5b08ffec_r.jpg)

**7.16 查看代码执行路径**

pp(): 输出当前被激活的探测点

![](https://pic1.zhimg.com/v2-fe036849ed72e18fa70ae24e691f7a14_r.jpg)

可以看出该函数哪些行被执行了。

**7.17 巧用正则匹配过滤**

在排查问题时，可以利用一些正则匹配来获取自己想要的信息，比如下面是只收集 *.[http://j9.com](https://link.zhihu.com/?target=http%3A//j9.com) 的堆栈：

![](https://pic1.zhimg.com/v2-9be4e2311116dc5f84ebe7d45ff14354_r.jpg)![](https://pic3.zhimg.com/v2-9fc12e872337d2fb0c215c382f5e422a_r.jpg)

**7.18 关联数组用法**

SystemTap 的关联数组必须是全局变量，需要用 global 进行声明，其索引可以支持多达 9 项索引域, 各域间以逗号隔开。支持 =, ++ 与 += 操作, 其默认的初始值为 0。

例如：

![](https://pic4.zhimg.com/v2-169ae8d504f297dfa53d896e18bf7d1f_r.jpg)![](https://pic4.zhimg.com/v2-a1a3b972c3f5fcc09fc5cea581ca34a3_r.jpg)

**7.19 调试内存泄漏以及内存重复释放**

![](https://pic4.zhimg.com/v2-7f8a0f613af222df8d32825d5a4f5acf_r.jpg)![](https://pic1.zhimg.com/v2-0a1fc3a9a606b43eb1614201bcd37a4c_r.jpg)

详细请看：[http://blog.csdn.net/wangzuxi/article/details/44901285](https://link.zhihu.com/?target=http%3A//blog.csdn.net/wangzuxi/article/details/44901285)

**7.20 嵌入 C 代码**

在进程 fork 出子进程时打印出进程 id 和进程名:

![](https://pic2.zhimg.com/v2-eb4119697a782777920fdc95758b38fd_r.jpg)

**有三个需要注意的地方：**

1）、SystemTap 脚本里面嵌入 C 语言代码要在每个大括号前加 % 前缀，是 %{…… %} 而不是 %{ …… }%；

2）、获取脚本函数参数要用 STAP_ARG_前缀；

3）、一般 long 等返回值用 STAP_RETURN，而 string 类型返回值要用 snprintf、strncat 等方式把字符串复制到 STAP_RETVALUE 里面。

**7.21 调试内核模块**

这小节就不细讲了，这篇博客 ([http://blog.chinaunix.net/uid-14528823-id-4726046.html](https://link.zhihu.com/?target=http%3A//blog.chinaunix.net/uid-14528823-id-4726046.html)) 写得很详细，这里只 copy 两个关键点过来记录一下：

要调试自己的内核模块，需要注意的有两个关键点：

1)、使用 SystemTap 调试内核模块，探测点的编写格式示例：

module("ext3").function("ext3_*")

2)、需要将自己的模块 cp 到 / lib/modules/uname -r/extra 目录中，否则找不到符号，如果 / lib/modules/uname -r / 目录下没有 extra 这个目录，自己 mkdir 一下就可以。

**7.22 一些错误提示及解决办法**

错误提示 1：

![](https://pic1.zhimg.com/v2-7e544841a50688937481119139f0cbc4_r.jpg)

解决办法：

加上 stap 参数：-DMAXACTION=102400，如果还报这种类型的错误，只需把 102400 调成更大的值即可。

错误提示 2：

![](https://pic1.zhimg.com/v2-940ce20c5005d0c1f24d85816a49fad0_r.jpg)

解决办法：

加上 - DMAXSKIPPED=102400 和 - DSTP_NO_OVERLOAD 参数

还有一些可以去掉限制的宏：

MAXSTRINGLEN：这个宏会影响 sprintf 的 buffer 大小，默认为 512 字节。

MAXTRYLOCK：对全局变量进行 try lock 操作的次数，超过则次数还拿不到锁则放弃和跳过该探测点，默认值为 1000. 全局变量多的时候可以把这个宏开大一点。

（完）

原文链接：[【技术干货】听阿里云 CDN 安防技术专家金九讲 SystemTap 使用技巧 - 博客 - 云栖社区 - 阿里云](https://link.zhihu.com/?target=http%3A//click.aliyun.com/m/28902/)

**更多技术干货敬请关注云栖社区知乎机构号：[阿里云云栖社区 - 知乎](https://www.zhihu.com/org/a-li-yun-yun-qi-she-qu-48)**
-------------------------------------------------------------------------------------------