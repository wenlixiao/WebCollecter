> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.junmajinlong.com](https://www.junmajinlong.com/linux/systemd/service_1/)

> 回到 Linux 基础系列文章大纲回到 Systemd 系列文章大纲回到 Shell 系列文章大纲 systemd 服务配置文件编写 (1)systemd service：简介 Systemd Service 是 systemd 提供的用于管理服务启动、停止和相关操作的功能，它极大的简化了服务管理的配置过程，用户只需要配置几项指令即可。

* * *

**[回到 Linux 基础系列文章大纲](https://www.junmajinlong.com/linux/index)**  
**[回到 Systemd 系列文章大纲](https://www.junmajinlong.com/linux/index#systemd)**  
**[回到 Shell 系列文章大纲](https://www.junmajinlong.com/shell/index)**

* * *

[](#systemd-service：简介 "systemd service：简介")systemd service：简介
--------------------------------------------------------------

Systemd Service 是 systemd 提供的用于管理服务启动、停止和相关操作的功能，它极大的简化了服务管理的配置过程，用户只需要配置几项指令即可。相比于 SysV 的服务管理脚本，用户不需要去编写服务的启动、停止、重启、状态查看等等一系列复杂且有重复造轮子嫌疑的脚本代码了，相信写过 SysV 服务管理脚本的人都深有体会。所以，Systemd Service 是面向所有用户的，即使对于新手用户来说，配置门槛也非常低。

systemd service 是 systemd 所管理的其中一项内容。实际上，systemd service 是 [Systemd Unit](https://www.freedesktop.org/software/systemd/man/systemd.unit.html) 的一种，除了 Service，systemd 还有其他几种类型的 unit，比如 socket、slice、scope、target 等等。在这里，暂时了解两项内容：

*   Service 类型，定义服务程序的启动、停止、重启等操作和进程相关属性
*   Target 类型，主要目的是对 Service(也可以是其它 Unit) 进行分组、归类，可以包含一个或多个 Service Unit(也可以是其它 Unit)

此外，Systemd 作为管家，还将一些功能集成到了 Systemd Service 中，个人觉得比较出彩的两个集成功能是：

*   用户可以直接在 Service 配置文件中定义 CGroup 相关指令来对该服务程序做资源限制。在以前，对服务程序做 CGroup 资源控制的步骤是比较繁琐的
*   用户可以选择 Journal 日志而非采用 rsyslog，这意味着用户可以不用单独去配置 rsyslog，而且可以直接通过 systemctl 或 journalctl 命令来查看某服务的日志信息。当然，该功能并不适用于所有情况，比如用户需要管理日志时

Systemd Service 还有其它一些特性，比如可以动态修改服务管理配置文件，比如可以并行启动非依赖的服务，从而加速开机过程，等等。例如，使用`systemd-analyze blame`可分析开机启动各服务占用的时长：

```
$ systemd-analyze blame
          3.557s network.service
          1.567s lvm2-pvscan@8:2.service
          1.060s lvm2-monitor.service
          1.046s dev-mapper-centos\x2droot.device
           630ms cgconfig.service
           581ms tuned.service
           488ms mysqld.service
           270ms postfix.service
           138ms auditd.service
            91ms polkit.service
            66ms boot.mount
            43ms systemd-logind.service
            ......


$ systemd-analyze time
Startup finished in 818ms (kernel) + 2.228s (initrd) + 3.325s (userspace) = 6.372s
multi-user.target reached after 2.214s in userspace
```

[](#systemd服务配置文件存放路径 "systemd服务配置文件存放路径")systemd 服务配置文件存放路径
------------------------------------------------------------

![](https://www.junmajinlong.com/img/linux/1594542888543.png)

如果用户需要，可以将服务配置文件手动存放至用户配置目录 / etc/systemd/system 下。该目录下的服务配置文件可以是普通`.service`文件，也可以是链接至 / usr/lib/systemd/system 目录下服务配置文件的软链接。

例如：

```
$ ls -1 /usr/lib/systemd/system/*.service | head
/usr/lib/systemd/system/arp-ethers.service
/usr/lib/systemd/system/auditd.service
/usr/lib/systemd/system/autovt@.service
/usr/lib/systemd/system/blk-availability.service
/usr/lib/systemd/system/brandbot.service
/usr/lib/systemd/system/cgconfig.service
/usr/lib/systemd/system/cgred.service
/usr/lib/systemd/system/console-getty.service
/usr/lib/systemd/system/console-shell.service
/usr/lib/systemd/system/container-getty@.service






$ ls -1dF /etc/systemd/system/*
/etc/systemd/system/basic.target.wants/
/etc/systemd/system/default.target
/etc/systemd/system/default.target.wants/
/etc/systemd/system/getty.target.wants/
/etc/systemd/system/local-fs.target.wants/
/etc/systemd/system/multi-user.target.wants/
/etc/systemd/system/nginx.service.d/
/etc/systemd/system/sockets.target.wants/
/etc/systemd/system/sysinit.target.wants/
/etc/systemd/system/system-update.target.wants/


$ ls -l /etc/systemd/system/multi-user.target.wants/ | awk '{print $9,$10,$11}'
auditd.service -> /usr/lib/systemd/system/auditd.service
crond.service -> /usr/lib/systemd/system/crond.service
irqbalance.service -> /usr/lib/systemd/system/irqbalance.service
mysqld.service -> /usr/lib/systemd/system/mysqld.service
postfix.service -> /usr/lib/systemd/system/postfix.service
remote-fs.target -> /usr/lib/systemd/system/remote-fs.target
rhel-configure.service -> /usr/lib/systemd/system/rhel-configure.service
rsyslog.service -> /usr/lib/systemd/system/rsyslog.service
sshd.service -> /usr/lib/systemd/system/sshd.service
tuned.service -> /usr/lib/systemd/system/tuned.service
```

[](#systemd-service文件格式说明 "systemd service文件格式说明")systemd service 文件格式说明
------------------------------------------------------------------------

一个 Systemd Service 的服务配置文件大概长这样：

```
[Unit]
Description = some descriptions
Documentation = man:xxx(8) man:xxx_config(5)
Requires = xxx1.target xxx2.target
After = yyy1.target yyy2.target

[Service]
Type = <TYPE>
ExecStart = <CMD_for_START>
ExecStop = <CMD_for_STOP>
ExecReload = <CMD_for_RELOAD>
Restart = <WHEN_TO_RESTART>
RestartSec = <TIME>

[Install]
WantedBy = xxx.target yy.target
```

一个`.Service`配置文件分为三部分：

*   Unit：定义该服务作为 Unit 角色时相关的属性
*   Service：定义本服务相关的属性
*   Install：定义本服务在设置服务开机自启动时相关的属性。换句话说，只有在创建 / 移除服务配置文件的软链接时，Install 段才会派上用场。这一配置段不是必须的，当未配置`[Install]`时，设置开机自启动或禁止开机自启动的操作将无任何效果

`[Unit]`和`[Install]`段的配置指令都来自于`man systemd.unit`，这些指令都用于描述作为 Unit 时的属性，`[Service]`段则专属于`.Service`服务配置文件。

这里先介绍一些常见的`[Unit]`和`[Install]`相关的指令 (虽然支持的配置指令很多，但只需熟悉几个即可)，之后再专门介绍 Service 段落的配置指令。

[](#Unit-段落指令 "[Unit]段落指令")[Unit] 段落指令
--------------------------------------

<table><thead><tr><th>Unit 指令</th><th>含义</th></tr></thead><tbody><tr><td>Description</td><td>Unit 的描述信息</td></tr><tr><td>Documentation</td><td>本 Unit 的 man 文档路径</td></tr><tr><td>After</td><td>本服务在哪些服务启动之后启动，仅定义启动顺序，不定义服务依赖关系，即使要求先启动的服务启动失败，本服务也依然会启动</td></tr><tr><td>Before</td><td>本服务在哪些服务启动之前启动，仅定义启动顺序，不定义服务依赖关系。通常用于定义在关机前要关闭的服务，如 Before=shutdown.target</td></tr><tr><td>Wants</td><td>本服务在哪些服务启动之后启动，定义服务依赖关系，不定义服务启动顺序。启动本服务时，如果被依赖服务未启动，则也会启动被依赖服务。如果被依赖服务启动失败，本服务不会受之影响，因此本服务会继续启动。如果未结合 After 使用，则本服务和被依赖服务同时启动。<br>当配置在<code>[Install]</code>段落中时，systemctl enable 操作将会将本服务安装到对应的<code>.wants</code>目录下 (在该目录下创建一个软链接)，在开机自启动时，<code>.wants</code>目录中的服务会被隐式添加至目标 Unit 的<code>Wants</code>指令后。</td></tr><tr><td>Requires</td><td>本服务在哪些服务启动之后启动，定义服务<strong>强依赖</strong>关系，不定义服务启动顺序。启动本服务时，如果被依赖服务未启动，则也会启动被依赖服务。如果结合了 After，当存在非 active 状态的被依赖服务时，本服务不会启动。且当被依赖服务被手动停止时，本服务也会被停止，但有例外。如果要保证两服务之间状态必须一致，使用 BindsTo 指令。<br>当配置在<code>[Install]</code>段落中时，systemctl enable 操作将会将本服务安装到对应的<code>.requires</code>目录下 (在该目录下创建一个软链接)，在开机自启动时，<code>.requires</code>目录中的服务会被隐式添加至目标 Unit 的<code>Requires</code>指令后。</td></tr><tr><td>Requisite</td><td>本服务在哪些服务启动之后启动，定义服务依赖关系，不定义服务启动顺序。启动本服务时，如果被依赖服务处于尚未启动状态，则不会主动去启动这些服务，所以本服务直接启动失败。该指令一般结合 After 一起使用，以便保证启动顺序。</td></tr><tr><td>BindsTo</td><td>绑定两个服务，两服务的状态保证一致。如服务 1 为 active，则本服务也一定为 active。</td></tr><tr><td>PartOf</td><td>本服务是其它服务的一部分，定义了单向的依赖关系，且只对 stop 和 restart 操作有效。当被依赖服务执行 stop 或 restart 操作时，本服务也会执行操作，但本服务执行这些操作，不会影响被依赖服务。一般用于组合 target 使用，比如 a.service 和 b.service 都配置 PartOf=c.target，那么 stop c 的时候，也会同时 stop a 和 b。</td></tr><tr><td>Conflicts</td><td>定义冲突的服务，本服务和被冲突服务的状态必须相反。当本服务要启动时，将会停止目标服务，当启动目标服务时，将会停止本服务。启动和停止的操作同时进行，所以，如果想要让本服务在目标服务启动之前就已经处于停止状态，则必须定义 After/Before。</td></tr><tr><td>OnFailure</td><td>当本服务处于 failed 时，将启动目标服务。如果本服务配置了 Restart 重启指令，则在耗尽重启次数之后，本服务才会进入 failed。<br>有时候这是非常有用的，一个典型用法是本服务失败时调用定义了邮件发送功能的 service 来发送邮件，特别地，可以结合 systemd.timer 定时任务实现 cron 的 MAILTO 功能。</td></tr><tr><td>RefuseManualStart, RefuseManualStop</td><td>本服务不允许手动启动和手动停止，只能被依赖时的启动和停止，如果手动启动或停止，则会报错。有些特殊的服务非常关键，或者某服务作为一个大服务的一部分，为了保证安全，都可以使用该特性。例如，系统审计服务 auditd.service 中配置了不允许手动停止指令 RefuseManualStop，network.target 中配置了不允许手动启动指令 RefuseManualStart。</td></tr><tr><td>AllowIsolated</td><td>允许使用 systemctl isolate 切换到本服务，只配置在 target 中。一般来说，用户服务是绝不可能用到这一项的。</td></tr><tr><td>ConditionPathExists, AssertPathExists</td><td>要求给定的绝对路径文件已经存在，否则不做任何事 (condition) 或进入 failed 状态(assert)，可在路径前使用<code>!</code>表示条件取反，即不存在时才启动服务。</td></tr><tr><td>ConditionPathIsDirectory, AssertPathIsDirectory</td><td>如上，路径存在且是目录时启动。</td></tr><tr><td>ConditionPathIsReadWrite, AssertPathIsReadWrite</td><td>如上，路径存在且可读可写时启动。</td></tr><tr><td>ConditionDirectoryNotEmpty, AssertDirectoryNotEmpty</td><td>如上，路径存在且是非空目录时启动。</td></tr><tr><td>ConditionFileNotEmpty, AssertFileNotEmpty</td><td>如上，路径存在且是非空文件时启动。</td></tr><tr><td>ConditionFileIsExecutable, AssertFileIsExecutable</td><td>如上，路径存在且是可执行普通文件时启动。</td></tr></tbody></table>

对于自定义的服务配置文件来说，需要定义的常见指令包括 Description、After、Wants 及可能需要的条件判断类指令。所以，Unit 段落是非常简单的。

[](#Install-段落指令 "[Install]段落指令")[Install] 段落指令
-----------------------------------------------

下面是`[Install]`段落相关的指令，它们只在`systemctl enable/disable`操作时有效。如果期望服务开机自启动，一般只配置一个 WantedBy 指令，如果不期望服务开机自启动，则 Install 段落通常省略。

<table><thead><tr><th>Install 指令</th><th>含义</th></tr></thead><tbody><tr><td>WantedBy</td><td>本服务设置开机自启动时，在被依赖目标的<code>.wants</code>目录下创建本服务的软链接。例如<code>WantedBy = multi-user.target</code>时，将在 / etc/systemd/multi-user.target.wants 目录下创建本服务的软链接。</td></tr><tr><td>RequiredBy</td><td>类似 WantedBy，但是是在<code>.requires</code>目录下创建软链接。</td></tr><tr><td>Alias</td><td>指定创建软链接时链接至本服务配置文件的别名文件。例如 reboot.target 中配置了 Alias=ctrl-alt-del.target，当执行 enable 时，将创建 / etc/systemd/system/ctrl-alt-del.service 软链接并指向 reboot.target。</td></tr><tr><td>DefaultInstance</td><td>当是一个模板服务配置文件时 (即文件名为<code>Service_Name@.service</code>)，该指令指定该模板的默认实例。例如 trojan@.service 中配置了 DefaultInstall=server 时，systemctl enable trojan@.service 时将创建名为 <a href="mailto:trojan@server.service">trojan@server.service</a> 的软链接。</td></tr></tbody></table>

例如，下面是 sshd 的服务配置文件 / usr/lib/systemd/system/sshd.service，只看 Unit 段落和 Install 段落，是否很简单？

![](https://www.junmajinlong.com/img/linux/1594542968859.png)

再来一个 auditd.service 的配置文件示例：

```
$ cat /usr/lib/systemd/system/auditd.service 
[Unit]
Description=Security Auditing Service
DefaultDependencies=no
After=local-fs.target systemd-tmpfiles-setup.service
Before=sysinit.target shutdown.target  
Conflicts=shutdown.target      
RefuseManualStop=yes      
ConditionKernelCommandLine=!audit=0
Documentation=man:auditd(8)

[Service]
......

[Install]
WantedBy=multi-user.target
```

[](#Service-段配置 "[Service]段配置")[Service] 段配置
--------------------------------------------

Systemd Service 配置文件中的`[Service]`段落可配置的指令很多，可配置在此段落中的指令来源有多处，包括：

![](https://www.junmajinlong.com/img/linux/1594543035735.png)

例如，/usr/lib/systemd/system/rsyslog.service 文件的内容：

```
[Service]
EnvironmentFile=-/etc/sysconfig/rsyslog  # 来自systemd.exec
UMask=0066            # 来自systemd.exec
StandardOutput=null   # 来自systemd.exec

Type=notify           # 来自systemd.service
ExecStart=/usr/sbin/rsyslogd -n $SYSLOGD_OPTIONS  # 来自systemd.service
Restart=on-failure    # 来自systemd.service
```

再比如，想要限制一个服务最多允许使用 300M 内存 (比如 512M 的 vps 主机运行一个比较耗内存的博客系统时，可设置内存使用限制)，最多 30%CPU 时间：

```
[Service]
MemoryLimit=300M
CPUQuota=30%
ExecStart=xxx
```

此外还需要了解 systemd 的一项功能，`systemctl set-property`，它可以在线修改已启动服务的属性。例如

```
systemctl set-property nginx MemoryLimit=100M


systemctl set-property nginx MemoryLimit=100M --runtime
```

目前来说，可以不用过多关注来自其它位置的指令，应该给予重点关注的是来自 systemd.service 自身的指令，比如：

*   Type：指定服务的管理类型
*   ExecStart：指定启动服务时执行的命令行
*   ExecStop：指定停止服务时运行的命令
*   ExecReload：指定重载服务进程时运行的命令
*   Restart：指定 systemd 是否要自动重启服务进程以及什么情况下重启

特别是 Type 指令，它直接影响`[Service]`段中的多项配置方式。

下面将从 Type 指令开始引入 Service 段的配置方式。

根据`man systemd.service`，Type 指令支持多种值：

*   simple
*   exec
*   forking
*   oneshot
*   dbus
*   notify
*   idle

如果配置的是服务进程，Type 的值很可能是 forking 或 simple，如果是普通命令的进程，Type 的值可能是 simple、oneshot。而 dbus 类型一般情况下用不上，notify 要求服务程序中使用代码对 systemd notify 进行支持，所以多数情况下可能也用不上。

关于 Type，内容较长，见下一篇文章 [systemd service 之：服务配置文件编写 (2)](https://www.junmajinlong.com/linux/systemd/service_2)。

版权声明: 本博客所有文章除特别声明外，均采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可协议。转载请注明来自 [骏马金龙](https://www.junmajinlong.com/)！

* * *