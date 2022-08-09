> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_67479400/article/details/124634112)

**目录**

[一. awk](#t0)

[1. 什么是 awk](#t1)

[2. awk 工具介绍](#t2)

[3. 命令格式](#t3)

[4. 工作原理](#t4)

[5. awk 内置变量](#t5)

[二. 示例](#t6)

[1. 打印文本内容](#t7)

[1.1 打印出网卡的 IP 地址](#t8)

[1.2 打印出网卡的流量](#t9)

[1.3 打印出根分区的可用量](#t10)

[2. BEGIN、END 模块](#t11)

[3. 模糊匹配](#t12)

[4. 数值与字符串比较](#t13)

[5. 逻辑运算 && 和 ||](#t14)

一. awk
======

1. 什么是 awk
----------

**awk** 是一种编程语言，用于在 linux/unix 下对文本和数据进行处理。数据可以来自标准输入 ([stdin](https://so.csdn.net/so/search?q=stdin&spm=1001.2101.3001.7020))、一个或多个文件，或其它命令的输出。它支持用户自定义函数和动态正则表达式等先进功能，是 linux/unix 下的一个强大编程工具。它在命令行中使用，但更多是作为脚本来使用。awk 有很多内建的功能，比如数组、函数等，这是它和 C 语言的相同之处，灵活性是 awk 最大的优势。

2. awk 工具介绍
-----------

功能强大的编剧工具

无交互的情况下实现复杂的文本操作

3. 命令格式
-------

> awk  选项  '模式或条件 {编辑指令}'  文件 1 文件 2
> 
> awk  -f  脚本文件  文件 1 文件 2

4. 工作原理
-------

awk 比较倾向于将一行分成多个 "字段" 然后再进行处理，且默认情况下字段的分隔符为空格或 tab 键。awk 执行结果可以通过 print 的功能将字段数据打印显示

在使用 awk 命令的过程中，可以使用逻辑操作符 "&&“表示" 与”、"“表示" 或”、"!“表示" 非”; 还可以进行简单的数学运算，如 +、-、*、/、%、^ 分别表示加、减、乘、除、取余和乘方

awk 后面接两个单引号并加上大括号 { } 来设置想要对数据进行的处理操作，awk 可以处理后续接的文件，也可以读取来自前个命令的标准输

5. awk 内置变量
-----------

<table border="1" cellpadding="1" cellspacing="1"><tbody><tr><td>FS</td><td>输入字段分隔符，默认为空格或制表位（tab）</td></tr><tr><td>OFS</td><td>输出字段的分割符（默认是空格）</td></tr><tr><td>RS</td><td>输入行分隔符</td></tr><tr><td>ORS</td><td>输出行的分割符，默认为换行符</td></tr><tr><td>NF</td><td>当前处理的行的字段个数</td></tr><tr><td>NR</td><td>当前处理的行的行号（序数）</td></tr><tr><td>FNR</td><td>读取文件的记录行号（从 1 开始，若读取新的文件依旧是从 1 开始）</td></tr><tr><td>$0</td><td>当前处理的行的整行内容</td></tr><tr><td>$n</td><td>当前处理行的第 n 个字段（第 n 列）</td></tr></tbody></table>

二. 示例
=====

1. 打印文本内容
---------

![](https://img-blog.csdnimg.cn/bda2f1c5aaf3480aae7f1e354efadf61.png)

![](https://img-blog.csdnimg.cn/7d22f45a854e4f95abfd0cff01a5098c.png)

![](https://img-blog.csdnimg.cn/d196dd2f82404b38a0ff25c699b5a9a6.png)

1.1 打印出网卡的 IP 地址
----------------

![](https://img-blog.csdnimg.cn/03462221effe448b9e278083b2a439a8.png)

1.2 打印出网卡的流量
------------

![](https://img-blog.csdnimg.cn/3de3456cf67c42b3802a821682aaa675.png)

1.3 打印出根分区的可用量
--------------

![](https://img-blog.csdnimg.cn/ca3e412e2dab43cab21991e1c280439b.png)

  **打印出 1~200 之间所有能被 7 整除并且包含数字 7 的整数数字**

![](https://img-blog.csdnimg.cn/c4236f594cc04374b28fbf3a19ed5f43.png)

2. BEGIN、END 模块
---------------

逐行执行开始之前执行什么任务，结束之后再执行什么任务，用 BEGIN、END

*   BEGIN 一般用来做初始化操作，仅在读取数据记录之前执行一次
*   END 一般用来做汇总操作，仅在读取完数据记录之后执行一次

![](https://img-blog.csdnimg.cn/6271e7980923434d866165a75f3919ed.png)

![](https://img-blog.csdnimg.cn/74fecde32aa442589ddb04ce69926c37.png)![](https://img-blog.csdnimg.cn/7905461f2c954afd97f6a911dbb2e053.png)

![](https://img-blog.csdnimg.cn/bfb7771160614eb3b966294b2d49c361.png)

3. 模糊匹配
-------

```
用~表示包含，!~表示不包含

```

![](https://img-blog.csdnimg.cn/954130e973f14cd8a4b9e0d41aeb3274.png)

![](https://img-blog.csdnimg.cn/fa6f9d85c3204f029e7f05704cf6fc20.png)

![](https://img-blog.csdnimg.cn/86107425f5654431ac14c238c9a986c6.png)

4. 数值与字符串比较
-----------

![](https://img-blog.csdnimg.cn/53992c27625540f8992b87247676744a.png)

5. 逻辑运算 && 和 ||
---------------

![](https://img-blog.csdnimg.cn/b9097b6d11a34d68ac48c350877566de.png)

 欲知后续，且看下回分解