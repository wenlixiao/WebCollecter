> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.junmajinlong.com](https://www.junmajinlong.com/linux/systemd/systemd_path/)

> 回到 Linux 基础系列文章大纲回到 Systemd 系列文章大纲回到 Shell 系列文章大纲 systemd path：实时监控文件和目录的变动 systemd path 工具提供了监控文件、目录变化并触发执行指定操作的功能。

* * *

**[回到 Linux 基础系列文章大纲](https://www.junmajinlong.com/linux/index)**  
**[回到 Systemd 系列文章大纲](https://www.junmajinlong.com/linux/index#systemd)**  
**[回到 Shell 系列文章大纲](https://www.junmajinlong.com/shell/index)**

* * *

systemd path 工具提供了监控文件、目录变化并触发执行指定操作的功能。

有时候这种监控功能是非常实用的，比如监控到`/etc/nginx/nginx.conf`或`/etc/nginx/conf.d/`发生变化后，立即 reload nginx。虽然，用户也可以使用 inotify 类的工具来监控，但远不如 systemd path 更方便、更简单且更易于观察监控效果和调试。

其实，systemd path 的底层使用的是 inotify，所以受限于 inotify 的缺陷，systemd path 只能监控本地文件系统，而无法监控网络文件系统。

[](#systemd-path能监控哪些操作 "systemd path能监控哪些操作")systemd path 能监控哪些操作
------------------------------------------------------------------

systemd path 暴露的监控功能并不多，它能监控的动作包括：

![](https://www.junmajinlong.com/img/linux/1594016427150.png)

这些指令监控的路径必须是绝对路径。

可以多次使用这些指令，且同一个指令也可以使用多次，这样就能够同时监控多个文件或目录，它们将共用事件触发后执行的操作。如果想要对不同监控目录执行不同操作，那只能定义多个 systemd path 的监控实例。

如果监控某路径时发现权限不足，则一直等待，直到有权监控。

如果在启动 Path Unit 时 (systemctl start xxx.path)，指定的路径已经存在(对于 PathExists 与 PathExistsGlob 来说) 或者指定的目录非空(对于 DirectoryNotEmpty 来说)，将会立即触发并执行对应操作。不过，对于 PathChanged 与 PathModified 来说，并不遵守这个规则。

[](#systemd-path使用示例 "systemd path使用示例")systemd path 使用示例
---------------------------------------------------------

要使用 systemd path 的功能，需至少编写两个文件，一个`.path`文件和一个`.service`文件，这两个文件的前缀名称通常保持一致，但并非必须。这两个文件可以位于以下路径：

*   /usr/lib/systemd/system/
*   /etc/systemd/system/
*   ~/.config/systemd/user/：用户级监控，只在该用户登录后才监控，该用户所有会话都退出后停止监控

例如：

```
/usr/lib/systemd/system/test.path
/usr/lib/systemd/system/test.service

/etc/systemd/system/test.path
/etc/systemd/system/test.service

~/.config/systemd/user/test.path
~/.config/systemd/user/test.service
```

例如，有以下监控需求：

1.  监控 / tmp/foo 目录下的所有文件修改、创建、删除等操作
2.  如果被监控目录 / tmp/foo 不存在，则创建
3.  监控 / tmp/a.log 文件的更改
4.  监控 / tmp/file.lock 锁文件是否存在

为了简化，这些监控触发的事件都执行同一个操作：向 / tmp/path.log 中写入一行信息。

此处将 path_test.path 文件和 path_test.service 文件放在 / etc/systemd/system / 目录下。

path_test.path 内容如下：

```
$ cat /etc/systemd/system/path_test.path
[Unit]
Description = monitor some files

[Path]
PathChanged = /tmp/foo
PathModified = /tmp/a.log
PathExists = /tmp/file.lock
MakeDirectory = yes
Unit = path_test.service

# 如果不需要开机后就自动启动监控的话，可省略下面这段
# 如果开机就监控，则加上这段，并执行systemctl enable path_test.path
[Install]
WantedBy = multi-user.target
```

其中 MakeDirectory 指令默认为 no，当设置为 yes 时表示如果监控的目录不存在，则自动创建目录，但该指令对 PathExists 指令无效。

Unit 指令表示该 sysmted path 实例监控到符合条件的事件时启动的服务单元，即要执行的对应操作。通常省略该指令，这时启动的服务名称和 path 实例的名称一致 (除了后缀)，例如`path_test.path`默认启动的是`path_test.service`服务。

path_test.service 内容如下：

```
$ cat /etc/systemd/system/path_test.service
[Unit]
Description = path_test.service

[Service]
ExecStart = /bin/bash -c 'echo file changed >>/tmp/path.log'
```

然后执行如下操作启动该 systemd path 实例：

```
systemctl daemon-reload
systemctl start path_test.path
```

使用如下命令可以列出当前已启动的所有 systemd path 实例：

```
$ systemctl --type=path list-units --no-pager
UNIT                               LOAD   ACTIVE SUB     DESCRIPTION                              
systemd-ask-password-console.path  loaded active waiting Dispatch Password Requests to Console
systemd-ask-password-wall.path     loaded active waiting Forward Password Requests to Wall Dir
path_test.path                     loaded active waiting monitor some files
```

然后测试该 systemd path 能否如愿工作。

```
$ touch /tmp/foo/a
$ touch /tmp/foo/a
$ touch /tmp/a.log
$ echo 'hello world' >>/tmp/a.log
$ rm -rf /tmp/a.log
...
```

如果想观察触发情况，可使用 journalctl。例如：

```
$ journalctl -u path_test.service
Jul 05 16:09:43 junmajinlong.com systemd[1]: Started path_test.service.
Jul 05 16:09:45 junmajinlong.com systemd[1]: Started path_test.service.
Jul 05 16:09:47 junmajinlong.com systemd[1]: Started path_test.service.
Jul 05 16:09:49 junmajinlong.com systemd[1]: Started path_test.service.
Jul 05 16:09:51 junmajinlong.com systemd[1]: Started path_test.service.
Jul 05 16:09:55 junmajinlong.com systemd[1]: Started path_test.service.
```

[](#systemd-path临时监控 "systemd path临时监控")systemd path 临时监控
---------------------------------------------------------

使用 systemd-run 命令可以临时监控路径。

```
$ systemd-run --path-property=PathModified=/tmp/b.log echo 'file changed'
Running path as unit: run-rb6f67e732fb243c7b530673cac867582.path
Will run service as unit: run-rb6f67e732fb243c7b530673cac867582.service
```

可以查看当前已启动的 systemd path 实例，包括临时监控实例：

```
$ systemctl --type=path list-units --no-pager
```

如果需要停止，使用 run-xxxxxx 名称即可：

```
systemctl stop run-rb6f67e732fb243c7b530673cac867582.path
```

[](#systemd-path资源控制 "systemd path资源控制")systemd path 资源控制
---------------------------------------------------------

systemd path 触发的任务可能会消耗大量资源，比如执行 rsync 的定时任务、执行数据库备份的定时任务，等等，它们可能会消耗网络带宽，消耗 IO 带宽，消耗 CPU 等资源。

想要控制这些定时任务的资源使用量也非常简单，因为真正执行任务的是`.service`，而 Service 配置文件中可以轻松地配置一些资源控制指令或直接使用 Slice 定义的 CGroup。这些资源控制类的指令可参考`man systemd.resource-control`。

例如，直接在`[Service]`中定义资源控制指令：

```
[Service]
Type=simple
MemoryLimit=20M
ExecStart=/usr/bin/backup.sh
```

又或者让 Service 使用定义好的 Slice：

```
[Service]
ExecStart=/usr/bin/backup.sh
Slice=backup.slice
```

其中 backup.slice 的内容为：

```
$ cat /usr/lib/systemd/system/backup.slice
[Unit]
Description=Limited resources Slice
DefaultDependencies=no
Before=slices.target

[Slice]
CPUQuota=50%
MemoryLimit=20M
```

[](#systemd-path的『Bug』 "systemd path的『Bug』")systemd path 的『Bug』
---------------------------------------------------------------

systemd path 监控路径上所产生的事件是需要时间的，如果两个事件发生时的时间间隔太短，systemd path 可能会丢失第二个甚至后续第三个第四个等等事件。

例如，使用`PathChanged`或`PathModified`监控路径 / tmp/foo 目录时，执行以下操作触发事件：

```
$ touch /tmp/foo/a && rm -rf /tmp/foo/a
```

期待的是 systemd path 能够捕获这两个事件并执行两次对应的操作，但实际上只会执行一次对应操作。换句话说，systemd path 丢失了一次事件。

之所以会丢失事件，是因为 touch 产生的事件被 systemd path 捕获，systemd path 立即启动对应`.service`服务做出对应操作，在本次操作还未执行完时，rm 又立即产生了新的事件，于是 systemd path 再次启动服务，但此时服务尚未退出，所以本次启动服务实际上什么事也不做。

所以，从结果上看去就像是 systemd path 丢失了事件，但实际上是因为服务尚未退出的情况下再次启动服务不会做任何事情。

可以加上一点休眠时间来耽搁一会：

```
$ touch /tmp/foo/a && sleep 0.1 && rm -rf /tmp/foo/a
```

上面的命令会成功执行两次对应操作。

再比如，将`.service`文件中的 ExecStart 设置为`/usr/bin/sleep 5`，那么在 5 秒内的所有操作，除了第一次触发的事件外，其它都会丢失。

systemd path 的这个『bug』也有好处，因为可以让**瞬间产生的多个有关联关系的事件只执行单次任务**，从而避免了中间过程产生的事件也重复触发相关操作。

版权声明: 本博客所有文章除特别声明外，均采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可协议。转载请注明来自 [骏马金龙](https://www.junmajinlong.com/)！

* * *