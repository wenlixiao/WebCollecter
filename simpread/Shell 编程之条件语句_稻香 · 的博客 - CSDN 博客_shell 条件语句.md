> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_67479400/article/details/124434542)

**目录**

[一. 条件测试](#t0)

[1. 条件测试操作](#t1)

 [1.1 test 命令](#t2)

 [1.2 文件测试](#t3)

 [1.3 常用的测试操作符](#t4)

[2. 整数值比较](#t5)

 [2.1 常用的测试操作符](#t6)

[3. 字符串比较](#t7)

 [3.1 常用的测试操作符](#t8)[​](#t8)

[4. 逻辑测试](#t9)

 [4.1 常用的测试操作符](#t10)

[二. if 语句](#t11)

[1. 单分支结构](#t12)[​](#t12)

[2. 双分支结构](#t13)

[3. 多分枝结构](#t14)

[4. 嵌套语句](#t15)

[三. case 语句](#t16)

[1. case 语句说明](#t17)

一. 条件测试
=======

1. 条件测试操作
---------

###   1.1 test 命令

测试[表达式](https://so.csdn.net/so/search?q=%E8%A1%A8%E8%BE%BE%E5%BC%8F&spm=1001.2101.3001.7020)是否成立，若成立返回 0，否则返回其他数值

### 格式 1：test 条件表达式

### 格式 2：【条件表达式】

###   1.2 文件测试

文件测试指的是根据给定的路径名称，判断对应的是文件还是目录，或者判断文件是否可读、可写、可执行等

### 格式：【操作符 文件或目录】

###   1.3 常用的测试操作符

<table border="1" cellpadding="1" cellspacing="1"><thead><tr><th>选项</th><th>说明</th></tr></thead><tbody><tr><td>-d</td><td>测试是否为自录 (Directory)</td></tr><tr><td>-e</td><td>测试目录或文件是否存在 (Exist)</td></tr><tr><td>-f</td><td>测试是否为文件 (File)</td></tr><tr><td>-r</td><td>测试当前用户是否有权限读取 (Read)</td></tr><tr><td>-w</td><td>测试当前用户是否有权限写入 (write)</td></tr><tr><td>-x</td><td>测试是否设置有可执行 (Excute) 权限</td></tr><tr><td>-b</td><td>测试是否为设备文件</td></tr><tr><td>-c</td><td>测试是否为字符设备文件</td></tr><tr><td>-s</td><td>测试存在且文件大小为空</td></tr><tr><td>-L</td><td>测试是否为链接文件</td></tr></tbody></table>

**示例 1：**

![](https://img-blog.csdnimg.cn/40563819740f4118bd9e4e9574bc5a4e.png)

 **示例 2：1 && 前条件成立则 && 后面的 ”yes“不显示** 

 **2 && 前条件不成立则显示 && 后面的 ”yes“**![](https://img-blog.csdnimg.cn/fdd9945365654489bd8564672a66baaa.png)

2. 整数值比较
--------

### 格式：【整数 1 操作符 整数 2】

###   2.1 常用的测试操作符

<table border="1" cellpadding="1" cellspacing="1"><thead><tr><th>选项</th><th>说明</th></tr></thead><tbody><tr><td>-eq</td><td>等于 (Equal)</td></tr><tr><td>-ne</td><td>不等于 (Not Equal)</td></tr><tr><td>-gt</td><td>大于 (Greater Than)</td></tr><tr><td>-It</td><td>小于 (Lesser Than)</td></tr><tr><td>-le</td><td>小于或等于 (Lesser or Equal)</td></tr><tr><td>-ge</td><td>大于或等于 (Greater or Equal)</td></tr></tbody></table>

**示例 1：**

![](https://img-blog.csdnimg.cn/8664371db5f547469a66914462cb3cd6.png)
---------------------------------------------------------------------

 **示例 2：**查看系统内存是否大于 100M，如果大于则显示提示

![](https://img-blog.csdnimg.cn/6465899af8164c3a9b8ef5294fcfdb07.png)

3. 字符串比较
--------

### 格式 1：【 字符串 1 = 字符串 2 】

###              【 字符串 1 ！= 字符串 2 】

### 格式 2：【 -z 字符串 】

###   3.1 常用的测试操作符

<table border="1" cellpadding="1" cellspacing="1"><thead><tr><th>选项</th><th>说明</th></tr></thead><tbody><tr><td>=</td><td>字符串内容相同</td></tr><tr><td>!=</td><td>字符串内容不同，！号表示相反的意思</td></tr><tr><td>-z</td><td><p>字符串内容为空</p></td></tr></tbody></table>

**示例 1：**

![](https://img-blog.csdnimg.cn/8e437129390d4d17bdecf4cba973e447.png)
---------------------------------------------------------------------

 **示例 2：**

![](https://img-blog.csdnimg.cn/9a935e206a9f4b7ba829a686d29aea02.png)

4. 逻辑测试
-------

格式 1：【表达式 1】 操作符 【表达式 2】...
---------------------------

格式 2：命令 1 操作符 命令 2 ...
----------------------

###   4.1 常用的测试操作符

<table border="1" cellpadding="1" cellspacing="1"><thead><tr><th>选项</th><th>说明</th></tr></thead><tbody><tr><td>-a 或 &amp;&amp;</td><td>逻辑与，“而且” 的意思</td></tr><tr><td>-o 或 ||</td><td><p>逻辑或，“或者” 的意思</p></td></tr><tr><td>！</td><td>逻辑否</td></tr></tbody></table>

**示例 1：**

![](https://img-blog.csdnimg.cn/0a1132cdae3943159de7f293ce9dd81d.png)
=====================================================================

 **示例 2：**

![](https://img-blog.csdnimg.cn/4c13ad5eb3fe4a3eb4291afd29f730a1.png)

二. if 语句
========

1. 单分支结构
--------

### ![](https://img-blog.csdnimg.cn/e5e2e74d39484f0584099ebe31d2197b.png)

### 结构图：![](https://img-blog.csdnimg.cn/8bec71faa98d41bdab4a5be6262c53d4.png)

**单分支只做一次判断，判断成立则输出 "OK"，不成立则不输出**

示例 1：数字大小对比，成立则输出 “OK” 不成立则不输出

**chmod +x 文件名 加权限否则无法运行**

![](https://img-blog.csdnimg.cn/7c3d85762c204f4780eefcd35318c55f.png)
---------------------------------------------------------------------

 示例 2：判断文件是否存在，如果不存在则创建该文件

![](https://img-blog.csdnimg.cn/31b7cedd2dfd4519b21b9b035a9907a4.png)

![](https://img-blog.csdnimg.cn/38a094ee73c74a78b59d42c6f4b819f3.png)

2. 双分支结构
--------

### ![](https://img-blog.csdnimg.cn/ef19064711874b839f9993ab7d1d352c.png)

### 结构图：![](https://img-blog.csdnimg.cn/8563fc4dccac407399c25025078c0b12.png)

**示例 1：判断指定的 IP 地址是否开启进行 ping 通，能 ping 通则给出 “UP” 若不能 ping 通则给出 “down”**

      **      /dev/null 是黑洞，用来存放垃圾文件**

 **-c 是 ping 的次数，-i 是 ping 的间隔时间**

![](https://img-blog.csdnimg.cn/11564634083e472485ded85da1449c69.png)

![](https://img-blog.csdnimg.cn/9da97ba9ce284043aa643e8cf3a7ad64.png)

 示例 2：创建用户并设置密码

![](https://img-blog.csdnimg.cn/2cbe6a1ac5e547b3b1c76d68ab37e1de.png)

![](https://img-blog.csdnimg.cn/9b3824a7a11648e4b52aeb4e10007435.png)

3. 多分枝结构
--------

### ![](https://img-blog.csdnimg.cn/512cb3dc51684acfa020e0c08f41da91.png)

### 结构图：![](https://img-blog.csdnimg.cn/f7b27a75c674433987af3134a9e3f795.png)

示例 1：判断 / home 的文件类别，并以多种文件类别进行判断

![](https://img-blog.csdnimg.cn/b7ec299865c9434abd92509221b561a7.png)

![](https://img-blog.csdnimg.cn/d7284e2c8217465481bc70e97fb72ea9.png)
=====================================================================

 示例 2：

![](https://img-blog.csdnimg.cn/7fd5f70c38454e91b5204fce7882d011.png)

![](https://img-blog.csdnimg.cn/adb671f7e20940c49e70f6d1ab7202da.png)

4. 嵌套语句
-------

判断 httpd 服务有没有启动

if 判断是否启动

如果启动 ------ 输出已启动

如果没启动 ---- 判断是否安装 --- 如果安装 --- 启动

示例 1 检测是否有 httpd 服务，没有的话下载并启动

![](https://img-blog.csdnimg.cn/a9a238c6db904f769a158904291bb81d.png)

![](https://img-blog.csdnimg.cn/9c3dec6414234c9a9f62c5e785822c9c.png)

三. case 语句
==========

1. case 语句说明
------------

case 语句可以使脚本程序的结构更加清晰、层次分明，常用于服务的启动、重启、停止的脚本，有的服务不提供这种控制脚本，需要用 case 语句编写 case 语句

主要适用于以下情况：某个变量存在多种取值，需要对其中的每一种取值分别执行不同的命令序列。这种情况与多分支的 if 语句非常相似，只不过 if 语句需要判断多个不同的条件，而 case 语句只是判断一个变量的不同取值

![](https://img-blog.csdnimg.cn/adae04e831e74cbba7d717e1a247f906.png)

###  结构图：

![](https://img-blog.csdnimg.cn/08369883989348de9e85599a361aa8a8.png)

示例 1  判断输入的字符类型

![](https://img-blog.csdnimg.cn/0be93308aa444d83993b7af4bc8c1389.png)

![](https://img-blog.csdnimg.cn/c9f6980218154f97a56acd3bc925b4af.png) 

 示例 2  利用位置变量信息以脚本的形式开始 httpd 的服务

![](https://img-blog.csdnimg.cn/e8285c698ed94fd3a2e47a119c4b36b7.png)

![](https://img-blog.csdnimg.cn/f27e653efb2741f9b770407a97e01f5d.png)