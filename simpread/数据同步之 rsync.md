> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/wujuntian/p/16410235.html) **何为 rsync？**

　　rsync 全称 remote synchronize，即 远程同步。

　　rsync 是 linux 系统下的数据镜像备份工具，可用于本地文件复制，也可与其他 SSH、rsync 主机远程同步文件和目录。

　　使用 rsync 进行数据同步时，第一次进行全量备份，以后则是增量备份，利用 rsync 算法（差分编码），只传输差异部分数据。

 

 

**如何 rsync？** **1. 安装**

```
yum install rsync

```

**2. 配置**

rsyncd 服务配置文件 /etc/rsyncd.conf： ![](https://img2022.cnblogs.com/blog/747151/202206/747151-20220624205735960-2085328305.png) 默认使用端口 873，可通过配置项目 port 进行修改。 **3. 三种工作模式** **（1）本地复制** 将本地目录 /var/rsync-src/ 下的文件同步至本地目录 /var/rsync-dest/

```
rsync -r /var/rsync-src/ /var/rsync-dest/

```

**（2）将本地数据同步到远程**（push）

将本地目录 /var/rsync-src/ 下的文件同步至远程主机 10.101.11.11 目录 /var/rsync-dest/

```
rsync -r /var/rsync-src/ username@10.101.11.11:/var/rsync-dest/

```

**（3）将远程数据同步到本地**（pull）

将远程主机 10.101.11.11 目录 /var/rsync-dest/ 下的文件同步至本地目录 /var/rsync-dest/

```
rsync -r username@10.101.11.11:/var/rsync-dest/ /var/rsync-dest/

```

**4. 两种认证协议**

　　rsync 进行远程同步时需要认证远程主机的账号密码，支持两种认证方式：ssh 协议认证与 rsync 协议认证。 **（1）ssh 认证** 　　**rsync 默认使用 ssh 协议进行远程登录和数据传输。**远程主机需要开启 sshd 服务，rsync 在传输数据之前会先与远程主机进行一次 ssh 登录认证，然后通过 ssh 隧道进行数据传输。**只需数据同步双方安装 rsync，但不必启动 rsyncd 服务。** 　　可用 -e 选项指定协议：

```
rsync -r -e ssh /var/rsync-src/ username@10.101.11.11:/var/rsync-dest/

```

　　也可省略 -e：

```
rsync -r /var/rsync-src/ username@10.101.11.11:/var/rsync-dest/

```

 　　使用 ssh 认证与传输的缺点是不安全：

<1> 登录认证使用的账号是远程主机可登录的系统账号，且需要手动输入密码；

<2> 同步数据不受目录限制。

**（2）rsync 协议认证** 　　与 ssh 认证不同，rsync 协议认证不需要依赖远程主机的 sshd 服务，但**需要远程主机开启 rsyncd 服务，本地 rsyncd 服务可不必开启。**另外，rsync 协议认证不是直接使用远程主机的真实系统账号，而是虚拟账号和虚拟密码，且可实现无需手动输入密码，同时 rsync 协议认证需要配置模块对远程同步的目录进行限制。对比 ssh 认证，rsync 协议认证安全性更高。 　　下面直接实践。（远程主机为服务端，本地主机为客户端） <1>rsyncd 配置（远程） ![](https://img2022.cnblogs.com/blog/747151/202206/747151-20220624210409642-830448427.png) 这里定义了一个模块，名为 wjt-rsync。 path：模块 wjt-rsync 对应的真实目录 /var/rsync-dest。此处是对客户端可操作目录的限制。 auth users：用于数据认证与传输的虚拟账号 jet，客户端以虚拟账号 jet 登录成功后，会转换成配置项 uid 指定的系统用户身份 rsync，以该身份完成对目标目录的读写操作。此处是对客户端操作权限的限制。 secrets file：虚拟账号与密码设置文件。 fake super：true 表示 uid 可以不为 root。 read only：指定当前模块是否只读，false 表示可读写，即可上传文件，true 表示只读，不可上传文件，默认为 true。 write only：true 表示不能下载，false 表示可下载，默认为 false。 <2> 创建用户与组（远程）

```
useradd rsync -s /sbin/nologin -M

```

<3> 设置虚拟账号密码（远程）

```
echo "jet:123456" > /etc/rsyncd.passwd  
chmod 600 /etc/rsyncd.passwd

```

<4> 修改同步目录属主（远程）

```
chown -R rsync.rsync /var/rsync-dest

```

<5> 开启 rsyncd 服务（远程）

```
systemctl start rsyncd

```

<6> 请求同步（本地）

```
rsync -r /var/rsync-src/ jet@10.101.11.11::wjt-rsync

```

也可以使用 url 格式语法：

```
rsync -r /var/rsync-src/ rsync://jet@30.102.74.67/wjt-rsync

```

 可通过配置本地密码文件实现无需手动输入密码：

```
echo 123456 > /etc/rsync.jet.passwd
chmod 600 /etc/rsync.jet.passwd
rsync -r /var/rsync-src/ jet@10.101.11.11::wjt-rsync --password-file=/etc/rsync.jet.passwd

```

或者通过环境变量进行设置：（变量名是固定的）

```
export RSYNC_PASSWORD=123456
rsync -r /var/rsync-src/ jet@30.102.74.67::wjt-rsync

```

 **两种认证方式的本质区别：**

　　ssh 协议认证连接的两端是通过管道完成通信和数据传输的，当连接到远程主机时，将在远程主机 fork 出 rsync 进程使其成为 rsync server；而 rsync 协议认证是事先在远程主机上运行 rsync 守护进程，监听套接字等待客户端的连接，建立连接后所有通信方式都是通过套接字完成的。

**5. 更多功能选项**

（1）-a：归档模式，表示递归传输并保持文件属性。 （2）-v：显示同步过程中详细信息（文件列表）。可以使用 "-vvvv" 获取更详细信息。 ![](https://img2022.cnblogs.com/blog/747151/202206/747151-20220624211204925-1096501537.png) 建立链接： ![](https://img2022.cnblogs.com/blog/747151/202206/747151-20220624211426038-1678691964.png) 开始传送增量文件列表：（部分截图） ![](https://img2022.cnblogs.com/blog/747151/202206/747151-20220624211512167-123178429.png) 开始传送文件数据：（部分截图） ![](https://img2022.cnblogs.com/blog/747151/202206/747151-20220624211553578-1665794237.png) 同步完成，输出统计信息： ![](https://img2022.cnblogs.com/blog/747151/202206/747151-20220624211644303-504180526.png) （3）-z：传输时进行压缩提高效率。 （4）-n：模拟执行，不会真正执行。 （5）--delete：删除只存在于目标目录、不存在于源目录的文件（服务端计算文件数据块校验码之前先执行此动作，--exclude 指定的文件会被标记为保护文件，不会被删除，但可通过 --delete-exclude 取消保护，强制删除）。 （6）--exclude：排除某些文件或目录，不参与操作。 （7）--include：指定必须参与操作的文件或目录，--include 必须在 --exclude 之前。 （8）--link-dest：基准目录，指定基准目录以后，rsync 会将源目录与基准目录之间变动的部分同步到目标目录。那些没变动的文件则会生成硬链接，硬链接指向上一次备份的对应文件，这与全量备份类似，但其数据量小很多（使用 rsync 实现全量备份的大致做法：使用备份时间命名区分多次备份的多个目标目录，每次备份之前，先将基准目录设置为上一次备份的目标目录，恢复时直接找到对应时间点的目录，里面的文件即是当时全部的文件数据）。 （9）--bwlimit：限速传输，单位 MB/s。 ......（还有更多功能选项）     **rsync 原理？** 　　**rsync 远程同步流程：** **1.** 客户端发起同步请求，与服务端建立连接。 **2.** 服务端验证用户身份。 **3.** 客户端扫描 rsync 命令输入的目录或文件，根据命令参数（--exclude、--include 等）筛选出需要同步的文件列表，发送给服务端。 **4.** 服务端将需要同步的文件按一定大小切分成一系列数据块，对它们进行编号，同时对每个数据块的内容计算两个校验码：32 位的弱滚动校验码 (rolling checksum，使用 adler-32 算法) 和 128 位的 MD5 强校验码。每个数据块包含五个属性：数据块编号、起始偏移地址、长度、32 位的弱滚动校验码、128 位的 MD5 强校验码。将所有这些数据块信息发送给客户端。 **5.** 客户端收到数据块信息集合后，对每个数据块的 32 位的弱滚动校验码计算得到 16 位长度的 hash 值。 ![](https://img2022.cnblogs.com/blog/747151/202206/747151-20220625105614392-1116560218.png)

（图片来自：[https://www.cnblogs.com/f-ck-need-u/p/7226781.html](https://www.cnblogs.com/f-ck-need-u/p/7226781.html)）

碰撞概率大小： hash 值 > 32 位的弱滚动校验码 > 128 位的 MD5 强校验码 **6.** 客户端将对应的本地文件进行处理与对比，具体流程如下： （1）从文件的第一个字节开始取相同大小的数据块，计算出 32 位的弱滚动校验码，再计算得到该验证码的 16 位长度的 hash 值； （2）根据 hash 值在 hash table 中查找匹配项，若找不到匹配项则说明该数据块与服务端数据块不同，执行第（5）步骤，若找到了匹配项则进行下一步比较； （3）将该数据块的 32 位的弱滚动校验码与 hash table 匹配项中的 32 位的弱滚动校验码进行比较，若找不到相同的，则说明该数据块与服务端数据块不同，执行第（5）步骤，若找到了匹配项则进行下一步比较； （4）对该数据块计算得到 128 位的 MD5 强校验码，与 hash table 匹配项中的 128 位的 MD5 强校验码进行比较，若相同则说明其内容与服务端数据块内容完全相同，否则说明存在差异； （5）若比较结果为相同，将该数据块信息发送给服务端，然后跳过该数据块，从该数据块的结尾偏移地址处继续取下一个数据块进行匹配比较；若比较结果为不同，则只跳过该数据块的第一个字节，从第二个字节开始取相同大小的数据块进行匹配比较，找到匹配的数据块之后，将之前不匹配的数据逐字节发送给服务端。 **7.** 服务端收到客户端的反馈信息之后，会创建一个临时文件，接收来自客户端的差异数据，相同的数据块则从本地对应的文件中读取，重组成一个新的文件，然后修改这个临时文件的属性信息，重命名该文件替换掉原来对应的文件，至此同步完成。 ![](https://img2022.cnblogs.com/blog/747151/202206/747151-20220625105934028-1891756055.png) （图片来自 [https://www.cnblogs.com/f-ck-need-u/p/7226781.html](https://www.cnblogs.com/f-ck-need-u/p/7226781.html)） 　　**数据块大小会影响 rsync 算法的性能：** 　　如果数据块大小太小，则数据块的数量就太多，需要计算和匹配的数据块校验码就太多，性能就差，而且出现 hash 值重复、rolling checksum 重复的可能性也增大；如果数据块大小太大，则可能会出现很多数据块都无法匹配的情况，导致这些数据块都被传输，降低了增量传输的优势。默认情况下，rsync 会根据文件大小自动判断数据块大小，但 rsync 命令的 "-B"(或 "--block-size") 选项支持手动指定大小，如果手动指定，官方建议大小在 500-1000 字节之间。 　　**每个文件的同步过程是持续进行的：** 　　客户端并不是对比完所有数据块之后才将相关数据发送给服务端的，而是每搜索出一个匹配数据块，就会立即将匹配块的相关信息以及当前匹配块和上一个匹配块中间的非匹配数据发送给服务端，并开始处理下一个数据块，服务端也是每收到一段数据后就会立即将其重组到临时文件中。     **rsync 的优缺点与适用场景？** **1. 优点** （1）可以镜像保存整个目录树和文件系统。 （2）可以很容易做到保持原来文件的权限、时间、软硬链接等等。 （3）无须特殊权限即可安装。 （4）快速：第一次同步时 rsync 会复制全部内容，但在下一次只传输修改过的文件。rsync 在传输数据的过程中可以实行压缩及解压缩操作，因此可以使用更少的带宽。 （5）安全：可以使用 scp、ssh 等方式来传输文件，当然也可以通过直接的 socket 连接。 （6）支持匿名传输，以方便进行网站镜像。 （7）跨平台：可在不同操作系统之间同步数据。 **2. 缺点** （1）客户端需要对多个文件数据块进行多次计算与比较验证码，对 CPU 的消耗比较大；服务端需要根据原文件和客户端传送过来的差异数据进行文件内容重组，对 IO 的消耗比较大。 （2）rsync 每次同步都需要先进行所有文件的扫描和计算、对比，最后才能进行差量传输。如果文件数量达到了百万甚至千万量级，扫描所有文件将是非常耗时的。而且如果改动的只是其中很小的一部分，这就是非常低效的方式。 （3）rsync 不能实时的去监测、同步数据，虽然它可以通过 crontab 守护进程的方式进行触发同步，但是两次触发动作一定会有时间差，这样就导致了服务端和客户端数据可能出现不一致，无法在应用故障时完全的恢复数据（无法实现实时同步），而且繁忙的轮询会消耗大量的资源。 **3. 适用场景** 　　由 rsync 工作原理可知，需要同步的文件改动越频繁，则客户端需要计算和比较的数据块验证码就越多（遇到数据块内容不相同时，只能跳过一个字节继续往后计算与比较，相同则可跳过一个数据块），对 CPU 的消耗就会越大；需要同步的文件越大，服务端每次都需要从一个很大的文件中复制相同的数据块进行新文件重组，几乎相当于直接 cp 了一个大文件，对 IO 的消耗也就越大。 　　所以 rsync 适合对改动不频繁、大小比较小的文件进行同步，对于改动频繁的大文件，只能偶尔同步一次，相当于备份的功能，而不是同步。     **rsync 如何实时同步？—— rsync + inotify！**     **何为 inotify？** 　　Inotify 是 Linux 内核从 2.6.13 开始引入的特性，它是一个内核用于通知用户空间程序文件系统变化的机制。 　　Inotify 监控文件系统操作，比如读取、写入和创建，基于事件驱动，可以做到对事件的实时响应，高效，而且没有轮询造成的系统资源消耗。     **如何 inotify？** **1. 安装** （1）检查当前系统内核是否支持 inotify

```
uname -r

```

若内核版本大于 2.6.13 则支持，或者：

```
grep INOTIFY_USER /boot/config-$(uname -r)

```

若输出 CONFIG_INOTIFY_USER=y，则支持。 （2）安装

```
yum install inotify-tools

```

**2. 命令** 　　inotify-tools 包含了两个命令：inotifywait 与 inotifywatch。 （1）inotifywait：在被监控的文件或目录上等待特定文件系统事件发生，执行后处于阻塞状态，适合在 shell 脚本中使用。 （2）inotifywatch：用于收集文件系统的统计数据，例如发生了多少次 inotify 事件，某文件被访问了多少次等等。 **3. 内核参数** /proc/sys/fs/inotify/ 目录下包含三个文件，分别设置 inotify 相关的三个内核参数。 （1）max_queued_events：inotify 事件队列可容纳的事件数量，超出的事件被丢弃，但会触发队列溢出 Q_OVERFLOW 事件。 （2）max_user_instances：每个用户可运行的 inotifywait 或 inotifywatch 命令的进程数。 （3）max_user_watches：每个 inotifywait 或 inotifywatch 命令可以监控的文件数量。如果监控的文件数目巨大，需要根据情况适当增加此值。 默认设置： ![](https://img2022.cnblogs.com/blog/747151/202206/747151-20220625110549063-1022952230.png) **4. 事件** inotify 监控的文件系统事件： （1）access：文件被访问。 （2）modify：文件被修改。 （3）attrib：文件元数据被修改。 （4）open：文件被打开。 （5）create：在被监控的目录中创建了文件或目录。 （6）delete：删除了被监控目录中的某个文件或目录。 ......（还有更多事件） 注意： **　　对文件的某个操作往往会触发多个事件，用户应用程序需要自己防止做出重复响应。** **5. 实践** 　　不指定监控事件，分别打开两个 shell 窗口，使用 inotifywait 和 inotifywatch 监控某个目录：

```
mkdir /var/inotify-test
inotifywait -m /var/inotify-test
inotifywatch -v /var/inotify-test

```

在此目录下创建两个文件：

```
touch /var/inotify-test/file1
touch /var/inotify-test/file2

```

看看 inotify 监控发生了什么：

![](https://img2022.cnblogs.com/blog/747151/202206/747151-20220625110845715-1014740461.png)

![](https://img2022.cnblogs.com/blog/747151/202206/747151-20220625110901696-353059488.png)

其中 inotifywait 的信息是在进程运行过程中输出的，而 inotifywatch 的信息是在进程结束时输出的。 　　以上测试没有指定监听事件，所以监听的是所有的事件，可以通过 -e 选项来指定监听事件，如：

```
inotifywait -m -e create,modify,delete /var/inotify-test

```

**inotify 原理？** 　　inotify 实现了基于 inode 的文件监控，采用在文件系统的处理函数中放置 hook 函数的方式实现。 　　......（还没详细了解）     **rsync + inotify 实践？** 　　有了 inotify 之后，通过 inotify 监控文件系统变化，发生变化则触发 rsync 执行同步，这样就解决了 rsync 无法实时同步的问题。 　　编写一个同步测试脚本 /var/inotify-test/real_time_sync： ![](https://img2022.cnblogs.com/blog/747151/202206/747151-20220625111127460-535591496.png) 赋予执行权限：

```
chmod a+x /var/inotify-test/real_time_sync

```

执行脚本，在 /var/rsync-src/ 目录下创建文件 my-test，同步脚本运行结果如下：

![](https://img2022.cnblogs.com/blog/747151/202206/747151-20220625111251103-1078015315.png)

验证服务端文件已自动同步更新，由此可实现实时同步！ 优点：只同步发生改动的文件或目录，而不是同步一整个监控目录，降低了资源消耗，提升了速度。 不足：单次文件操作触发了多个事件从而发起多次 rsync，并未对此做出处理。     **rsync + inotify 问题？** 1. 同一个操作会触发多个事件。 2. inotifywait 存在缺陷，当向监控目录下拷贝复杂层次目录 (多层次目录中包含文件)，或者向其中拷贝大量文件时，inotifywait 经常会随机性地遗漏某些文件。 3. 并发如果大于 200 个文件（10-100K），同步会有延迟。 4. 监控到事件后，调用 rsync 同步是单线程的。     **更完美的方案？ —— sersync！** 　　sersync 是金山的周洋基于 rsync + inotify-tools 开发的工具，它克服了 inotify 的缺陷，可以过滤重复事件减轻负担，并且自带 crontab 功能、多线程调用 rsync、失败重传等功能。   　　当同步数据量不大时，还是建议使用 rsync + inotify，当数据量很大（几百 G 甚至 1T 以上）时，建议使用 rsync + sersync。     　　未完，不续......         参考： [https://zhuanlan.zhihu.com/p/395994766](https://zhuanlan.zhihu.com/p/395994766) [https://www.cnblogs.com/f-ck-need-u/p/7226781.html](https://www.cnblogs.com/f-ck-need-u/p/7226781.html) [https://www.linuxprobe.com/use-rsync-file.html](https://www.linuxprobe.com/use-rsync-file.html) [https://www.ruanyifeng.com/blog/2020/08/rsync.html](https://www.ruanyifeng.com/blog/2020/08/rsync.html) [https://blog.csdn.net/qq_58268157/article/details/120717446](https://blog.csdn.net/qq_58268157/article/details/120717446) [https://www.cnblogs.com/f-ck-need-u/p/7220193.html](https://www.cnblogs.com/f-ck-need-u/p/7220193.html) [https://zhuanlan.zhihu.com/p/85005210](https://zhuanlan.zhihu.com/p/85005210) [https://www.cnblogs.com/zoe233/p/12035383.html](https://www.cnblogs.com/zoe233/p/12035383.html) [https://www.sohu.com/a/244164762_467784](https://www.sohu.com/a/244164762_467784) [https://blog.csdn.net/qq_58268157/article/details/120717446](https://blog.csdn.net/qq_58268157/article/details/120717446)