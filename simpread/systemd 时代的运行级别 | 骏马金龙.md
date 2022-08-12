> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.junmajinlong.com](https://www.junmajinlong.com/linux/systemd/runlevel/)

> 回到 Linux 基础系列文章大纲回到 Systemd 系列文章大纲回到 Shell 系列文章大纲 systemd 时代的运行级别在 CentOS 6 及之前的版本中有运行级别的概念，Systemd 系统内没有直接定义运行级别的概念，但是通过 Target Unit 兼容模拟了运行级别。

* * *

**[回到 Linux 基础系列文章大纲](https://www.junmajinlong.com/linux/index)**  
**[回到 Systemd 系列文章大纲](https://www.junmajinlong.com/linux/index#systemd)**  
**[回到 Shell 系列文章大纲](https://www.junmajinlong.com/shell/index)**

* * *

在 CentOS 6 及之前的版本中有运行级别的概念，Systemd 系统内没有直接定义运行级别的概念，但是通过 Target Unit 兼容模拟了运行级别。

可以查看 / usr/lib/systemd/system / 下的一些 target 文件。为了节省篇幅，下面我列出了部分 target：

```
$ ls -l /usr/lib/systemd/system/*.target | grep -o '/.*' 


/usr/lib/systemd/system/default.target -> graphical.target


/usr/lib/systemd/system/runlevel0.target -> poweroff.target
/usr/lib/systemd/system/runlevel1.target -> rescue.target
/usr/lib/systemd/system/runlevel2.target -> multi-user.target
/usr/lib/systemd/system/runlevel3.target -> multi-user.target
/usr/lib/systemd/system/runlevel4.target -> multi-user.target
/usr/lib/systemd/system/runlevel5.target -> graphical.target
/usr/lib/systemd/system/runlevel6.target -> reboot.target


/usr/lib/systemd/system/emergency.target
/usr/lib/systemd/system/rescue.target
/usr/lib/systemd/system/multi-user.target
/usr/lib/systemd/system/graphical.target


/usr/lib/systemd/system/halt.target
/usr/lib/systemd/system/poweroff.target
/usr/lib/systemd/system/shutdown.target
/usr/lib/systemd/system/reboot.target
/usr/lib/systemd/system/ctrl-alt-del.target -> reboot.target
```

![](https://www.junmajinlong.com/img/linux/1594542846345.png)

而 target 的主要作用是对服务进行分组、归类。所以，只需要定义几个代表不同运行级别的 target，并在不同的 target 中放入不同的服务程序即可 (除了服务程序还可以包含其它的 Unit)。

target 又是如何对服务进行分组、归类的呢？作为初步了解，可在 / etc/systemd/system 中寻找答案。在此目录下，有一些`*.target.wants`目录，该目录定义了该 target 中包含了哪些 Unit，systemd 会在处理到对应 target 时会寻找 wants 后缀的目录，并加载启动该目录下的所有 Unit，这就是 target 对服务 (及其它 Unit) 分组的方式。

例如：

```
$ ls -1F /etc/systemd/system/
basic.target.wants/
default.target@
default.target.wants/
getty.target.wants/
local-fs.target.wants/
multi-user.target.wants/
nginx.service.d/
remote-fs.target.wants/
sockets.target.wants/
sysinit.target.wants/
system-update.target.wants/


$ ls -1 /etc/systemd/system/sysinit.target.wants/
cgconfig.service
lvm2-lvmetad.socket
lvm2-lvmpolld.socket
lvm2-monitor.service
rhel-autorelabel-mark.service
rhel-autorelabel.service
rhel-domainname.service
rhel-import-state.service
rhel-loadmodules.service


$ ls -1 /etc/systemd/system/multi-user.target.wants/
auditd.service
crond.service
irqbalance.service
mysqld.service
nfs-client.target
postfix.service
remote-fs.target
rhel-configure.service
rpcbind.service
rsyslog.service
sshd.service
tuned.service
```

之所以有这些 wants 目录，并且其中有一些 Unit 文件，是因为在 Service 配置文件 (或其它 Unit) 中的`[Install]`段落使用了`WantedBy`指令。例如：

```
$ cat /usr/lib/systemd/system/sshd.service
[Unit]
......

[Service]
......

[Install]
WantedBy=multi-user.target
```

当使用`systemctl enable Unit_Name`让 Unit_Name 开机自启动时，会寻找该`[Install]`中的 WantedBy 和 RequiredBy，并在对应的`/etc/systemd/system/xxx.target.wants`或`/etc/systemd/system/xxx.target.requires`目录下创建软链接。

如果 Service 配置文件中没有定义 WantedBy 和 RequiredBy，则`systemctl enable`操作不会有任何效果。

此外，可以在 target 配置文件内部使用`Wants、Requires`等表示依赖含义的指令来定义该 target 依赖哪些 Unit。

例如：

```
$ cat /usr/lib/systemd/system/sysinit.target
[Unit]
Description=System Initialization
Documentation=man:systemd.special(7)
Conflicts=emergency.service emergency.target
Wants=local-fs.target swap.target
```

`.target`文件中`Wants`指令定义的更符合依赖的含义，而`.target.wants`目录更倾向于表明该 target 中归类了哪些要运行的服务。

比如负责系统环境初始化的 sysinit.target，其中的 Wants 指令定义了必须先运行且成功运行文件系统相关任务 (local-fs.target 和 swap.target) 后才运行 sysinit.target，也就是开始启动`.target.wants`目录下的 Unit。

执行`systemctl list-units --type target`可以查看系统当前已经加载的所有 target，包括那些开机自启动过程中启动的。

```
$ systemctl list-units --type target
UNIT                  LOAD   ACTIVE SUB    DESCRIPTION
basic.target          loaded active active Basic System
cryptsetup.target     loaded active active Local Encrypted Volumes
getty.target          loaded active active Login Prompts
local-fs-pre.target   loaded active active Local File Systems (Pre)
local-fs.target       loaded active active Local File Systems
multi-user.target     loaded active active Multi-User System
network-online.target loaded active active Network is Online
network-pre.target    loaded active active Network (Pre)
network.target        loaded active active Network
paths.target          loaded active active Paths
remote-fs.target      loaded active active Remote File Systems
slices.target         loaded active active Slices
sockets.target        loaded active active Sockets
swap.target           loaded active active Swap
sysinit.target        loaded active active System Initialization
timers.target         loaded active active Timers
```

除了上面展示的 target，在 / usr/lib/systemd/system 目录下还有很多 target。而且，只要用户想要对一类 Unit 进行分组归类，那么也可以自己定义 target。

但需要明确的是，**target 可分为两类**：

*   可直接切换的 target(模拟运行级别)
*   不可直接切换的 target

切换是什么意思？比如从当前的运行级别 3 切换到运行级别 5，将会启动运行级别 5 上的所有程序以及依赖程序，并停止当前已启动但运行级别 5 不需要的服务程序。这就是运行级别的切换，只是停止一些服务 (或程序)、并启动另外一些服务而已。

切换 target 也一样，比如切换到 graphical.target 时，会启动目标 graphical.target 需要的所有服务，并停止当前已运行但目标 target 不需要的服务。

切换 target 的方式如下：

```
systemctl isolate Target_Name

systemctl isolate default.target  
systemctl isolate rescue.target   


systemctl default
systemctl resuce
systemctl emergency
systemctl halt
systemctl poweroff
systemctl reboot
```

可查看或设置默认的运行级别：

```
systemctl get-default
systemctl set-default Target_Name
```

设置默认运行级别，实际上是创建 / etc/systemd/system/default.target 指向对应 target 配置文件的软链接。

比如：

```
$ systemctl set-default multi-user.target
Removed symlink /etc/systemd/system/default.target.
Created symlink from /etc/systemd/system/default.target to /usr/lib/systemd/system/multi-user.target.
```

target 是否可直接切换，取决于 target 配置文件中是否定义了`AllowIsolate=yes`指令。比如 multi-user.target 是模拟运行级别的 target，肯定允许直接切换，而 network.target 定义的是网络启动任务，肯定不可以直接切换。

```
$ systemctl cat multi-user.target 


[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target
Conflicts=rescue.service rescue.target
After=basic.target rescue.service rescue.target
AllowIsolate=yes


$ systemctl show -p AllowIsolate network.target
AllowIsolate=no
```

版权声明: 本博客所有文章除特别声明外，均采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可协议。转载请注明来自 [骏马金龙](https://www.junmajinlong.com/)！

* * *