> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/f-ck-need-u/p/7429520.html)

另一篇终端会话共享的文章：[Linux 终端会话实时共享 (kibitz)](https://www.cnblogs.com/f-ck-need-u/p/9583753.html)

使用 script 命令录制，使用 scriptreplay 播放录制的操作。共享终端的操作，则需要使用命名管道来实现。

1.1 录制
------

```
[root@xuexi ~]# cd /tmp

[root@xuexi tmp]# script -t 2> timing.log -a output.session  # 开始录制
Script started, file is output.session

```

```
[root@xuexi tmp]# ls                 # 执行一个操作：命令ls
abc.sh  ab.sh  index.html  lost+found  output.session  scriptfifo  test  test1  timing.log  vmware-root

[root@xuexi tmp]# cd /tmp/test      # 再执行一个操作：命令cd

```

```
[root@xuexi test]# exit  # 结束录制
exit
Script done, file is output.session

```

其中 "-t 2> timing.log" 是要回放的必须选项，不加 "2>" 将导致开启录制后的任何输入都是乱码状态，不加 "-t timing.log" 将不能使用 scriptreplay 来回放。timing.log 记录的是每个时间段输入了多少字符。通过 timing.log 和 output.session 配合可以实现回放。

注意点是，录制前保证 timing.log 和 output.session 是空文件，否则将导致回放时操作不一致。

1.2 回放
------

```
[root@xuexi test]# scriptreplay timing.log output.session

```

如果觉得回放的速度过慢 (录制时有些地方停顿，比如输入了一个命令后，隔了一段时间才输入另一个命令，这段时间对于回放来说显得慢很正常)，可以修改 timing.log 文件。这个文件中分两个字段，第一个字段记录的是从上次输出后到该次输出的时间间隔，第二个字段是从 output.session 中读取的字符数。要修改回放速度，只需将第一个字段较长的间隔改短一点就可以。但是不应该改的太短，否则回放速度过快。我觉得将间隔较长的改成 0.3-0.7 秒，效果还不错。

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
[root@xuexi ~]# cat timing.log
0.117244 16
0.007955 1
0.298074 1       # 此处将原来的2.298074改为0.3秒
0.216628 1
0.092781 1
0.081659 1
0.083258 1
0.419445 1
0.314128 1
0.100810 1
0.083998 30
0.491283 1
0.266129 1   # 此处原来也是2.266129秒，显然经过一次命令输出之后停顿了2秒多
0.099767 1
0.127625 1
0.078809 1
0.181493 1
0.147795 1
0.115808 1
0.077416 1
0.274658 1
0.257042 1
0.524460 4
0.297133 38
0.458018 1
0.416350 1
0.187270 1
0.125467 1
0.100756 8

```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

1.3 终端屏幕分享
----------

通过管道来传输信息实现。需要一个 pipe 文件，并在需要展示的终端打开这个管道文件。

在终端 1（作为主终端，即演示操作的终端）上使用 mkfifo 创建管道文件。

```
[root@xuexi tmp]# mkfifo scriptfifo

[root@xuexi tmp]# ll scriptfifo
prw-r--r-- 1 root root 0 Sep 26 13:04 scriptfifo   # 权限位前面的第一个p代表的就是pipe文件。

```

![](https://images2017.cnblogs.com/blog/733013/201708/733013-20170825200421605-2073081875.png) 

在终端 2 上打开 pipe 文件。

```
[root@xuexi ~]# cat /tmp/scriptfifo

```

![](https://images2017.cnblogs.com/blog/733013/201708/733013-20170825200439636-531572923.png) 

在终端 1 上使用 script -f 开始记录操作，之后的操作将会分享在终端 2 上。

```
[root@xuexi tmp]# script -f scriptfifo

```

 ![](https://images2017.cnblogs.com/blog/733013/201708/733013-20170825200455511-557663889.png)

![](https://images2017.cnblogs.com/blog/733013/201708/733013-20170825200500464-2021334641.png)

使用 exit 即可停止分享并退出记录行为。

```
[root@xuexi tmp]# exit
exit
Script done, file is scriptfifo

```

在被分享终端上参与分享状态后将不能执行任何操作，执行的操作会被记录下来，并在主终端停止分享后自动执行。

需要注意的是，只能给一个会话共享会话终端。如果多个会话 cat scriptfifo ，会导致共享切开显示在多个不同会话上。

**转载请注明出处：[https://www.cnblogs.com/f-ck-need-u/p/7429520.html](https://www.cnblogs.com/f-ck-need-u/p/7429520.html)**

**如果觉得文章不错，不妨给个打赏，写作不易，各位的支持，能激发和鼓励我更大的写作热情。谢谢！**  

![](https://files.cnblogs.com/files/f-ck-need-u/wealipay.bmp)