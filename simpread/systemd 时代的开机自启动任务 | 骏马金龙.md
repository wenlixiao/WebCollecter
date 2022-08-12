> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.junmajinlong.com](https://www.junmajinlong.com/linux/systemd/auto_tasks_on_boot/)

> 回到 Linux 基础系列文章大纲回到 Systemd 系列文章大纲回到 Shell 系列文章大纲 systemd 时代的开机自启动任务如果要让任务开机自启动，需将对应的 Unit 文件存放于 / etc/systemd/system 下。

* * *

**[回到 Linux 基础系列文章大纲](https://www.junmajinlong.com/linux/index)**  
**[回到 Systemd 系列文章大纲](https://www.junmajinlong.com/linux/index#systemd)**  
**[回到 Shell 系列文章大纲](https://www.junmajinlong.com/shell/index)**

* * *

如果要让任务开机自启动，需将对应的 Unit 文件存放于 / etc/systemd/system 下。本文以 Service Unit 为例，但也支持让 path Unit、timer Unit 等类型的任务开机自启动。

[](#systemd中服务开机自启动 "systemd中服务开机自启动")systemd 中服务开机自启动
------------------------------------------------------

用户可以手动将服务配置文件存放至此路径，但更建议采用 systemd 系统提供的上层工具 systemctl 来操作。

```
systemctl enable Service_Name


systemctl disable Service_Name


systemctl is-enabled Service_Name


systemctl list-unit-files --type service | grep 'enabled'
```

使用 systemctl 命令时，可以指定服务名称，也可以指定服务对应的服务配置 unit 文件。

例如下面两条命令是等价的。

```
systemctl enable sshd          
systemctl enable sshd.service
```

systemctl 的很多操作都具备幂等性，这意味着如果要操作的服务已经处于目标状态，则什么都不会做。

比如 systemctl 启动服务 sshd，但如果 sshd 服务已经处于目标状态：已启动，则本次启动什么操作也不做，systemctl 会直接退出。再比如上面将 sshd 加入开机自启动的操作，sshd 服务在安装 openssh-server 的时候就已经自动加入了开机自启动，用户再手动加入开机自启动，实际上什么也不会做。

如果是未开机自启动的服务加入开机自启动呢？比如，拷贝 sshd 服务的配置文件，并将拷贝后的服务 sshd1 加入开机自启动：

```
$ cp /usr/lib/systemd/system/{sshd,sshd1}.service

$ systemctl enable sshd1
Created symlink from /etc/systemd/system/multi-user.target.wants/sshd1.service to /usr/lib/systemd/system/sshd1.service.
```

从结果可看到，systemctl 将服务加入开机自启动的操作，实际上是在 / etc/systemd/system 某个 target.wants 目录下创建服务配置文件的软链接文件。

```
$ readlink /etc/systemd/system/multi-user.target.wants/sshd1.service 
/usr/lib/systemd/system/sshd1.service
```

显然，禁用服务开机自启动的操作是移除软链接。

```
$ systemctl disable sshd1
Removed symlink /etc/systemd/system/multi-user.target.wants/sshd1.service.
```

最后，如果服务已经加入开机自启动，但想要再次加入 (比如更新了 / usr/lib/systemd/system 下的服务配置文件)，可在 enable 时加上–force 选项：

```
systemctl --force enable Service_Name
```

[](#systemd中自定义开机自启动命令-脚本 "systemd中自定义开机自启动命令/脚本")systemd 中自定义开机自启动命令 / 脚本
--------------------------------------------------------------------------

![](https://www.junmajinlong.com/img/linux/1594542766431.png)

但更建议的方案是编写开机自启动服务，后面会专门介绍服务管理配置文件如何编写。

下面是一个简单的让命令 (脚本) 开机自启动的配置文件：

```
$ cat /usr/lib/systemd/system/mycmd.service
[Unit]
Description = some shell script

ConditionFileIsExecutable=/usr/bin/some.sh


[Service]
ExecStart = /usr/bin/some.sh


[Install]
WantedBy = multi-user.target

$ systemctl daemon-reload
$ systemctl enable mycmd.service
```

如果要使用 / etc/rc.local 的方式呢？systemd 提供了 rc-local.service 服务来加载 / etc/rc.d/rc.local 文件中的命令。

```
$ cat /usr/lib/systemd/system/rc-local.service 


[Unit]
Description=/etc/rc.d/rc.local Compatibility
ConditionFileIsExecutable=/etc/rc.d/rc.local
After=network.target

[Service]
Type=forking
ExecStart=/etc/rc.d/rc.local start
TimeoutSec=0
RemainAfterExit=yes
```

这个文件缺少了`[Install]`段且没有 WantedBy，后面将会解释 Install 中的 WantedBy 表示设置该服务开机自启动时，该服务加入到哪个『运行级别』中启动。

但这个文件的注释中说明了，如果 / etc/rc.d/rc.local 文件存在且具有可执行权限，则 systemd-rc-local-generator 将会自动添加到 multi-user.target 中，所以，即使没有 Install 和 WantedBy 也无关紧要。

另一方面需要注意，和 SysV 系统在系统启动的最后阶段运行 rc.local 不太一样，systemd 兼容的 rc.local 是在 network.target 即网络相关服务启动完成之后就启动的，这意味着 rc.local 可能在开机启动过程中较早的阶段就开始运行。

如果想要将命令加入到 / etc/rc.local 中实现开机自启动，直接写入该文件，并设置该文件可执行权限即可。

例如：

```
echo -e '#!/bin/bash\ndate +"%F %T" >/tmp/a.log' >>/etc/rc.d/rc.local
chmod +x /etc/rc.d/rc.local
```

版权声明: 本博客所有文章除特别声明外，均采用 [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可协议。转载请注明来自 [骏马金龙](https://www.junmajinlong.com/)！

* * *