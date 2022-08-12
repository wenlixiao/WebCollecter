> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.junmajinlong.com](https://www.junmajinlong.com/linux/systemd/service_2/)

> 回到 Linux 基础系列文章大纲回到 Systemd 系列文章大纲回到 Shell 系列文章大纲 systemd 服务配置文件编写 (2) 接下来会通过示例来描述不同 Service Type 值的应用场景。

**[回到 Linux 基础系列文章大纲](https://www.junmajinlong.com/linux/index)**  
**[回到 Systemd 系列文章大纲](https://www.junmajinlong.com/linux/index#systemd)**  
**[回到 Shell 系列文章大纲](https://www.junmajinlong.com/shell/index)**

接下来会通过示例来描述不同 Service Type 值的应用场景。在此之前，强烈建议先阅读[前后台进程父子关系和 daemon 类进程](https://www.junmajinlong.com/linux/process_relationship)来搞懂进程之间的关系和 Daemon 类进程的特性。

[](#systemd-service：Type-forking "systemd service：Type=forking")systemd service：Type=forking
--------------------------------------------------------------------------------------------

当使用 systemd 去管理一个长久运行的服务进程时，最常用的 Type 是 forking 类型。

**使用 Type=forking 时，要求 ExecStart 启动的命令自身就是以 daemon 模式运行的**。而以 daemon 模式运行的进程都有一个特性：总是会有一个瞬间退出的中间父进程，如果不了解这点特性，请看[前后台进程父子关系和 daemon 类进程](https://www.junmajinlong.com/linux/process_relationship)。

例如，nginx 命令默认以 daemon 模式运行，所以可直接将其配置为 forking 类型：

```
$ cat test.service 
[Unit]
Description = Test

[Service]
Type = forking
ExecStart = /usr/sbin/nginx

$ systemctl daemon-reload
$ systemctl start test
$ systemctl status test
● test.service - Test
   Loaded: loaded
   Active: active (running)
  Process: 7912 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
 Main PID: 7913 (nginx)
    Tasks: 5
   Memory: 4.6M
   CGroup: /system.slice/test.service
           ├─7913 nginx: master process /usr/sbin/nginx
           ├─7914 nginx: worker process
           ├─7915 nginx: worker process
           ├─7916 nginx: worker process
           └─7917 nginx: worker process
```

注意上面 status 报告的信息中，ExecStart 启动的 nginx 的进程 PID=7912，且该进程的状态是已退出，退出状态码为 0，这个进程是 daemon 类进程创建过程中瞬间退出的中间父进程。在 forking 类型中，该进程称为初始化进程。同时还有一行`Main PID: 7913 (nginx)`，这是 systemd 真正监控的 nginx 服务主进程，其 PID=7913，是 PID=7912 进程的子进程。

`Type=forking`类型代表什么呢？要解释清楚该 type，需从进程创建开始说起。

![](https://www.junmajinlong.com/img/linux/1594543080380.png)

![](https://www.junmajinlong.com/img/linux/1593590225497.png)

对于 Type=forking 来说，pid=1 的 systemd 进程 fork 出来的子进程正是瞬间退出的中间父进程，且 systemd 会在中间父进程退出后就认为服务启动成功，此时 systemd 可以立即去启动后续需要启动的服务。

如果 Type=forking 服务中的启动命令是一个前台命令会如何呢？比如将 sleep 配置为 forking 模式，将 nginx daemon off 配置为 forking 模式等。

答案是 systemd 会一直等待中间 ExecStart 启动的进程作为中间父进程退出，在等待过程中，systemctl start 会一直卡住，直到等待超时而失败，在此阶段中，systemctl status 将会查看到服务处于 activating 状态。

```
$ cat test.service 
[Unit]
Description = Test

[Service]
Type = forking
ExecStart = /usr/sbin/nginx -g 'daemon off;'

$ systemctl daemon-reload
$ systemctl start test    
$ systemctl status test   
● test.service - Test
   Loaded: loaded
   Active: activating (start)
  Control: 9227 (nginx)
    Tasks: 1
   Memory: 2.0M
   CGroup: /system.slice/test.service
           └─9227 /usr/sbin/nginx -g daemon off;
```

回到 forking 类型的服务。由于 daemon 类的进程会有一个瞬间退出的中间父进程 (如上面 PID=7913 的 nginx 进程)，systemd 是如何知道哪个进程是应该被监控的服务主进程(Main PID) 呢？

答案是靠猜。没错，systemd 真的就是靠猜的。当设置 Type=forking 时，有一个`GuessMainPID`指令其默认值为 yes，它表示 systemd 会通过一些算法去猜测 Main PID。当 systemd 的猜测无法确定哪个为主进程时，后果是严重的：systemd 将不可靠。因为 systemd 无法正确探测服务是否真的失败，当 systemd 误认为服务失败时，如果本服务配置了自动重启 (配置了 Restart 指令)，重启服务时可能会和当前正在运行但是 systemd 误认为失败的服务冲突 (比如出现端口已被占用问题)。

多数情况下的猜测过程很简单，systemd 只需去找目前存活的属于本服务的 leader 进程即可。但有些服务 (少数) 情况可能比较复杂，在多进程之间做简单的猜测并非总是可靠。

好在，Type=forking 时的 systemd 提供了 PIDFile 指令 (Type=forking 通常都会结合 PIDFile 指令)，systemd 会从 PIDFile 指令所指定的 PID 文件中获取服务的主进程 PID。

例如，编写一个 nginx 的服务配置文件：

```
$ cat test.service 
[Unit]
Description = Test

[Service]
Type = forking
PIDFile = /run/nginx.pid
ExecStartPre = /usr/bin/rm -f /run/nginx.pid
ExecStart = /usr/sbin/nginx
ExecStartPost = /usr/bin/sleep 0.1
```

### [](#Type-forking时PIDFile指令的坑 "Type=forking时PIDFile指令的坑")Type=forking 时 PIDFile 指令的坑

关于 PIDFile，有必要去了解一些注意事项，否则它们可能就会成为你的坑。

首先，PIDFile 只适合在 Type=forking 模式下使用，其它时候没必要使用，因为其它类型的 Service 主进程的 PID 都是确定的。systemd 推荐 PIDFile 指定的 PID 文件在 / run 目录下，所以，可能需要修改服务程序的配置文件，将其 PID 文件路径修改为 / run 目录之下，当然这并非必须。

但有一点必须注意，**PIDFile 指令的值要和服务程序的 PID 文件路径保持一致**。

例如 nginx 的相关配置：

```
$ grep -i 'pid' /etc/nginx/nginx.conf    
pid /run/nginx.pid;

$ cat /usr/lib/systemd/system/nginx.service
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid    
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

其次，**systemd 会在中间父进程退出后立即读取这个 PID 文件，读取成功后就认为该服务已经启动成功**。但是，systemd 读取 PIDFile 的时候，服务主进程**可能**还未将 PID 写入到 PID 文件中，这时 systemd 将出现问题。所以，对于服务程序的开发人员来说，应尽早将主进程写入到 PID 文件中，比如可以在中间父进程 fork 完之后立即写入 PID 文件，然后再退出，而不是在 fork 出来的服务主进程内部由主进程负责写入。

上面的 nginx 服务配置文件是某个 nginx 版本 yum 包提供的，但却是有问题的，我曾经踩过这个坑，网上甚至将其报告为一个 Bug。

上面的 nginx.service 文件可以正常启动服务，但无法`systemctl reload`，只要 reload 就报错，而且报错时提示 kill 命令语法错误。kill 语法错误显然是因为没有获取到`$MAINPID`变量的值，而这正是因为 systemd 在 nginx 写入 PID 文件之前先去读取了 PID 文件，因为没有读取到内容，所以`$MAINPID`变量为空值。

解决办法是使用`ExecStartPost=/usr/bin/sleep 0.1`，让 systemd 在初始化进程 (即中间父进程) 退出之后耽搁 0.1 秒再继续向下执行，即推迟了 systemd 读取 PID 的过程，保证能让 systemd 从 PID 文件中读取到值。

![](https://www.junmajinlong.com/img/linux/1594543157624.png)

最后，systemd 只会读 PIDFile 文件而不会写，也不会创建它。但是，在停止服务的时候，systemd 会尝试删除 PID 文件。因为服务进程可能会异常终止，导致已终止的服务进程的 PID 文件仍然保留着，所以在使用 PIDFile 指令时，通常还会使用`ExecStartPre`指令来删除可能已经存在的 PID 文件。正如上面给出的 nginx 配置文件一样。

[](#systemd-service：Type-simple "systemd service：Type=simple")systemd service：Type=simple
-----------------------------------------------------------------------------------------

`Type=simple`是一种最常见的通过 systemd 服务系统运行用户自定义命令的类型，也是省略 Type 指令时的默认类型。

`Type=simple`类型的服务**只适合那些在 shell 下运行在前台的命令**。也就是说，当一个命令本身会以 daemon 模式运行时，将不能使用 simple，而应该使用`Type=forking`。比如 ls 命令、sleep 命令、非 daemon 模式运行的 nginx 进程以及那些以前台调试模式运行的进程，在理论上都可以定义为 simple 类型的服务。至于为何有如此规则，稍后会解释的明明白白。

例如，编写一个 / usr/lib/systemd/system/test.service 运行 sleep 进程：

```
[Unit]
Description = test

[Service]
Type = simple
ExecStart = /usr/bin/sleep 10  # 命令必须使用绝对路径
```

使用 daemon-reload 重载并启动该服务进程：

```
$ systemctl daemon-reload
$ systemctl start test
$ systemctl status test
● test.service - Test
   Loaded: loaded
   Active: active (running)
 Main PID: 6902 (sleep)
    Tasks: 1
   Memory: 96.0K
   CGroup: /system.slice/test.service
           └─6902 /usr/bin/sleep 10
```

10 秒内，sleep 进程以 daemon 模式运行在后台，就像一个服务进程一样。10 秒之后，sleep 退出，于是 systemd 将该进程从监控队列中踢出。再次查看进程的状态将是 inactive：

```
$ systemctl status test
● test.service - Test
   Loaded: loaded
   Active: inactive (dead)
```

再来分析上面的服务配置文件中的指令。

`ExecStart`指令指定启动本服务时执行的命令，即启动一个**本该前台运行的** sleep 进程作为服务进程在后台运行。

> 需注意，systemd service 的命令行中必须使用绝对路径，且只能编写单条命令 (Type=oneshot 时除外)，如果要命令续行，可在尾部使用反斜线符号`\`等。
> 
> 此外，命令行中支持部分类似 Shell 的特殊符号，但不支持重定向`> >> << <`、管道`|`、后台符号`&`，具体可参考`man systemd.service`中 command line 段落的解释说明。

对于`Type=simple`来说，systemd 系统在 fork 出子 systemd 进程后就认为服务已经启动完成了，所以 systemd 可以紧跟着启动排在该服务之后启动的服务。它的伪代码模型大概是这样的：

```
# pid = 1: systemd

# start service1 with Type=simple
pid=fork()
if(pid=0){
  # Child Process: sub systemd process
  exec(<Service_Cmd>)
}

# start other services after service1
...
```

例如，先后连续启动两个 Type=simple 的服务，进程流程图大概如下：

![](https://www.junmajinlong.com/img/linux/1594543253065.png)

换句话说，**当 Type=simple 时，systemd 只在乎 fork 阶段是否成功，只要 fork 子进程成功，这个子进程就受 systemd 监管，systemd 就认为该 Unit 已经启动**。

因为子进程已成功被 systemd 监控，无论子进程是否启动成功，**在子进程退出时，systemd 都会将其从监控队列中踢掉，同时杀掉所有附属进程** (默认行为是如此，杀进程的方式由 systemd.kill 中的 KillMode 指令控制)。所以，查看服务的状态将是`inactive(dead)`。

例如，下面的配置种，睡眠 1 秒后，该服务的状态将变为`inactive(dead)`。

```
[Service]
ExecStart = /usr/bin/sleep 1
```

这没什么疑问。但考虑一下，如果 simple 类型下 ExecStart 启动的命令本身就是以 daemon 模式运行的呢？其结果是 **systemd 默认会立刻杀掉所有属于服务的进程**。

原因也很简单，daemon 类进程总是会有一个瞬间退出的中间父进程，而在 simple 类型下，systemd 所 fork 出来的子进程正是这个中间父进程，所以 systemd 会立即发现这个中间父进程的退出，于是杀掉其它所有服务进程。

例如，以运行`bash -c '(sleep 3000 &)'`的 simple 类型的服务，被 systemd 监控的 bash 进程会在启动 sleep 后立即退出，于是 systemd 会立即杀掉属于该服务的 sleep 进程。

```
$ cat test.service    
[Unit]
Description = Test

[Service]
ExecStart = bash -c '( sleep 3000 & )'

$ systemctl daemon-reload
$ systemctl start test
$ systemctl status test
● test.service - Test
   Loaded: loaded
   Active: inactive (dead)
```

再例如，nginx 命令默认是以 daemon 模式运行的，simple 类型下直接使用 nginx 命令启动服务，systemd 会立刻杀掉所有 nginx，即 nginx 无法启动成功。

```
$ cat test.service    
[Unit]
Description = Test

[Service]
ExecStart = /usr/sbin/nginx

$ systemctl daemon-reload
$ systemctl start test
$ systemctl status test
● test.service - Test
   Loaded: loaded
   Active: inactive (dead)
```

但如果将 nginx 进程以非 daemon 模式运行，simple 类型的 nginx 服务将正常启动：

```
$ cat test.service 
[Unit]
Description = Test

[Service]
ExecStart = /usr/sbin/nginx -g 'daemon off;'

$ systemctl daemon-reload
$ systemctl start test
$ systemctl status test
● test.service - Test
   Loaded: loaded
   Active: active (running)
 Main PID: 7607 (nginx)
    Tasks: 5
   Memory: 4.6M
   CGroup: /system.slice/test.service
           ├─7607 nginx: master process /usr/sbin/nginx -g daemon off;
           ├─7608 nginx: worker process
           ├─7609 nginx: worker process
           ├─7610 nginx: worker process
           └─7611 nginx: worker process
```

[](#Systemd-Service：其它Type类型 "Systemd Service：其它Type类型")Systemd Service：其它 Type 类型
----------------------------------------------------------------------------------

除了 simple 和 forking 类型，还有 exec、oneshot、idle、notify 和 dbus 类型，这里不考虑 notify 和 dbus，剩下的 exec、oneshot 和 idle 都类似于 simple 类型。

*   simple：在 fork 出子 systemd 进程后，systemd 就认为该服务启动成功了
*   exec：在 fork 出子 systemd 进程且子 systemd 进程 exec() 调用 ExecStart 命令成功后，systemd 认为该服务启动成功
*   oneshot：在 ExecStart 命令执行完成退出后，systemd 才认为该服务启动成功
    *   因为服务进程退出后 systemd 才继续工作，所以在未配置 RemainAfterExit 指令时，oneshot 类型的服务永远无法出现 active 状态，它直接从启动状态到 activating 到 deactivating 再到 dead 状态
    *   当结合 RemainAfterExit 指令时，在服务进程退出后，systemd 会继续监控该 Unit，所以服务的状态为`active(exited)`，通过这个状态可以让用户知道，该服务曾经已经运行成功，而不是从未运行过
    *   通常来说，对于那些执行单次但无需长久运行的进程来说，可以采用 type=oneshot，比如启动 iptables，挂载文件系统的操作、关机或重启的服务等
*   idle：无需考虑这种类型

[](#模板型服务配置文件 "模板型服务配置文件")模板型服务配置文件
-----------------------------------

systemd service 支持简单的模板型 Unit 配置文件，在 Unit 配置文件中可以使用`%n %N %p %i...`等特殊符号进行占位，在 systemd 读取配置文件时会将这些特殊符号解析并替换成对应的值。

这些特殊符号的含义可参见`man systemd.unit`。通常情况下只会使用到`%i`或`%I`，其它特殊符号用到的机会较少。

使用`%i %I`这两个特殊符号时，要求 Unit 的文件名以`@`为后缀，即文件名格式为`Service_Name@.service`。当使用 systemctl 管理这类服务时，`@`符号后面的字符会传递到 Unit 模板文件中的`%i`或`%I`。

例如，执行下面这些命令时，会使用`abc`替换`service_name@.service`文件中的`%i`或`%I`。

```
systemctl start service_name@abc
systemctl status service_name@abc
systemctl stop service_name@abc
systemctl enable service_name@abc
systemctl disable service_name@abc
```

有时候这是很实用的。比如有些程序即是服务端程序又是客户端程序，区分客户端和服务端的方式是使用不同配置文件。

假设用户想在一个机器上同时运行 xyz 程序的服务端和客户端，可编写如下 Unit 服务配置文件：

![](https://www.junmajinlong.com/img/linux/1594543380723.png)

现在用户可以在 / etc/server 目录下同时提供服务程序的服务端配置文件和客户端配置文件。

```
/etc/server/server.conf
/etc/server/client.conf
```

如果要管理该主机上的服务端：

```
systemctl start xyz@server
systemctl status xyz@server
systemctl stop xyz@server
systemctl enable xyz@server
systemctl disable xyz@server
```

如果要管理该主机上的客户端：

```
systemctl start xyz@client
systemctl status xyz@client
systemctl stop xyz@client
systemctl enable xyz@client
systemctl disable xyz@client
```

[](#使用target组合多个服务 "使用target组合多个服务")使用 target 组合多个服务
----------------------------------------------------

有些时候，一个大型服务可能由多个小服务组成。

比如 c 服务由 a.service 和 b.service 组成，因为组合了两个服务，所以 c 服务可以定义为 c.target。

a.service 内容：

```
[Unit]
Description = a.service
PartOf = c.target
Before = c.target

[Install]
ExecStart = /path/to/a.cmd
```

b.service 内容：

```
[Unit]
Description = b.service
PartOf = c.target
Before = c.target

[Install]
ExecStart = /path/to/b.cmd
```

c.target 内容：

```
[Unit]
Description = c.service, consists of a.service and b.service
After = a.service b.service
Wants = a.service b.service
```

c 中配置 Wants 表示 a 和 b 先启动，但启动失败不会影响 c 的启动。如果要求 c.target 和 a.service、b.service 的启动状态一致，可将 Wants 替换成 Requires 或 BindsTo 指令。

PartOf 指令表明 a.service 和 b.service 是 c.target 的一部分，停止或重启 c.target 的同时，也会停止或重启 a 和 b。再加上 c.target 中配置了 Wants 指令 (也可以改成 Requires 或 BindsTo)，使得启动 c 的时候，a 和 b 也已经启动完成。

但是要注意，PartOf 是单向的，停止和重启 a 或 b 的时候不会影响 c。