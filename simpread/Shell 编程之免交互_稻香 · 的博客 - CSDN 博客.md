> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_67479400/article/details/125124335)

**目录**

[一. Here Document 免交互](#t0)

[1. 免交互定义](#t1)

[2. 语法格式](#t2)

[3. 应用示例](#t3)

[3.1 示例 1](#t4)

[3.2 示例 2](#t5)

[3.2 示例 3](#t6)

[4. Here Document 变量设定](#t7)

[4.1 示例 1](#t8)

[4.2 示例 2](#t9)

[5. 多行注释](#t10)

[二. expect](#t11)

[1. 定义](#t12)

[2. expect 安装](#t13)

[3. 相关命令](#t14)

[3.1 脚本解释器](#t15)

[3.2 spawn](#t16)

[3.3 expect](#t17)

[3.4 send](#t18)

[3.5 结束符](#t19)

[3.6 set](#t20)

[3.7 exp_continue](#t21)

[3.8 send_ user](#t22)

[3.9 接收参数](#t23)

[4. 示例](#t24)

[4.1 示例 1](#t25)

[4.2 示例 2](#t26)

[4.3 示例 3](#t27)

一. Here Document 免交互
====================

1. 免交互定义
--------

使用 I/O 重定向的方式将命令列表提供给交互式程序

标准输入的一种替代品，可以帮助脚本开发人员不必使用临时文件来构建输入信息，而是直接就地 生产出一个文件并用作命令的标准输入, Here Document 可以与非交互式程序和命令一起使用

2. 语法格式
-------

```
命令 << 标记
 
....
输入内容
......
 
标记
```

*   **标记可以使用任意的合法字符（通用的字符是 EOF）**
    
*   **结尾的标记一定要顶格写，前面不能有任何字符（包括空格）**
    
*   **结尾的标记后面也不能有任何字符（包括空格）**
    
*   **开头标记前后空格会被省略掉**
    
*   **单引号 变量双引号 —**
    

3. 应用示例
-------

### 3.1 示例 1

使用 wc -l 命令后面直接跟文件名就可以统计文件内有多少行内容，将要统计的内容置于标记 “EOF” 之间，直接将内容传给 wc -l 来统计

```
[root@xlj1 ~]#wc -l <<EOF
> 1
> 2
> 3
> EOF
3
```

### 3.2 示例 2

通常使用 read 命令接收用户的输入值时会有交互过程，在 EOF 两个标记间可以输入变量值

```
##read命令会有交互过程
[root@xlj1 ~]# read -p "请输入一个数字" ack
请输入一个数字8
[root@localhost ~]# echo $ack 
8
 
###使用免交互，则EOF之间的值会变成变量i的参数
[root@xlj1 ~]# read i <<EOF
> hello
> EOF
 
[root@xlj1 ~]#echo $i
hello
```

### 3.2 示例 3

使用 passwd 命令设置密码

```
[root@xlj1 ~]#passwd liu <<EOF
> 123123
> 123123
> EOF
```

4. **Here Document 变量设定**
-------------------------

Here Document 也支持使用变量，如果标记之间有变量被使用，会先替换变量值。如果想要将一些内容写入文件，除了常规的方法外，也可以使用 Here Document。如果写入 的内容中包含变量，在写入文件时要先将变量替换成实际值，在结合 cat 命令完成写入

### 4.1 示例 1

在写入文件时会先将变量替换成实际值，再结合 cat 命令完成写入

```
[root@xlj1 home]#vim eof1.sh 
 
#!/bin/bash
file="yxp.txt"
var="park"
cat <<EOF >$file
I am going to the $var
EOF
```

### 4.2 示例 2

整体赋值给变量输出，然后通过 echo 命令将变量值打印出来，调用变量加上双引号会更好

```
[root@xlj1 home]#vim eof1.sh
#!/bin/bash
file="yxp1.txt"
var="park"
my=$(cat <<EOF >$file
I am going to the $var
EOF
)
echo $my
```

5. 多行注释
-------

*   Bash 的默认注释是 “#”，该注释方法只支持单行注释: Here Document 的引入解决了多行注释的问题
*   “:" 代表什么都不做的空命令。中间标记区域的内容不会被执行，会被 bash 忽略掉，因此可达到批量注释的效果

```
#!/bin/bash
file="yxp2.txt"
var="park"
myvar=$(cat <<EOF >$file
I am going to the $var.
I will go to play with my friends.
I am very happy.
 
EOF
)
echo $myvar
###下面部分就被注释了不会显示
:<<EOF
echo "I am going to the $var"
echo "I am very happy."
EOF
```

二. expect
=========

1. 定义
-----

是建立在 tcl（tool command language）语言基础上的一个工具，常被用于进行自动化控制和测试，解决 shell 脚本中交互的相关问题

2. expect 安装
------------

```
rpm -q expect
rpm -q tcl
yum install -y expect
```

3. 相关命令
-------

### 3.1 脚本解释器

expect 脚本中首先引入文件, 表明使用的是哪一个 shell

```
#!/usr/bin/expect

```

### 3.2 spawn

spawn 后面通常跟一个 Linux 执行命令, 表示开启一个会话、启动进程, 并跟踪后续交互信息

```
spawn passwd root

```

### 3.3 expect

*   判断上次输入结果中是否包含指定的字符串，如果有则立即返回，否则就等待超时时间后返回
*   只能捕捉由 spawn 启动的进程的输出
*   用于接收命令执行后的输出，然后和期望的字符串匹配

### 3.4 send

```
向进程发送字符串,用于模拟用户的输入，该命令不能自动回车换行,一般要加 \r (回车) 或者 \n
 
expect "密码" {send "abc123\r"}   
 
或
 
expect "密码"
 
send "abc123\r"
 
#同一行send部分要有{}   换行send部分不需要有{}
 
expect          
"密码1"{ send "abc123\r"}
"密码2"{ send "123456\r"}
"密码3"{ send "123123\r"}
 
#expect支持多个分支,只要匹配了其中一个情况，执行相应的send语句后退出该expect语句
```

### 3.5 结束符

**expect eof**

*   表示交互结束，等待执行结束，退回到原用户，与 spawn 对应
*   比如：切换到 root 用户，expect 脚本默认的是等待 10s，当执行完命令后，默认停留 10s 后，就会自动切回原用户。

**interact**

*   执行完成后保持交互状态，把控制权交给控制台，会停留在目标终端，这个时候就可以手工操作了，interact 后的命令不起作用
*   比如 interact 会保持在终端而不会退回到原终端，比如切换到 root 用户，会一直在 root 用户状态下
*   比如： ssh 到另一服务器，会一直在目标服务器终端，而不会切回的原服务器

需要注意的是：expect eof 与 interact 只能二选一

### 3.6 set

expect 默认的超时时间是 10 秒，通过 set 命令可以设置会话超时时间，若不限制超时时间则应设置为 - 1

```
set timeout 20

```

### 3.7 exp_continue

*   exp_continue 附加于某个 expect 判断项之后，可以使该项匹配后，还能继续匹配该 expect 判断语句内的其他项
*   exp_continue 类似于控制语句中 continue 语句, 表示允许 expect 继续向下执行指令

例如，将判断交互输出中是否存在 yes/no 或 *password , 如果匹配 yes/no 则输出 yes 并再次执行判断; 如果匹配 *password 则输出 123123 并结束该段 expect 语句

操作如下：

```
 expect
"(yes/no)" {send "yes\r"; exp_ continue; }
"*password" {set timeout 300; send "abc123\r";}
```

### 3.8 send_ user

```
send_ user 表示回显命令，等同于echo

```

### 3.9 接收参数

*   expect 脚本可以接受从 bash 命令行传递的参数，使用 [lindex $argv n] 获得
*   其中 n 从 0 开始，分别表示第一个, 第二个，第三个… 参数

```
set hostname [lindex $argv 0]       #相当于 hostname=$1
 
set password [lindex $argv 1]        #相当于 password=$2
 
#expect直接执行，需要使用 expect 命令去执行脚本
```

4. 示例
-----

### 4.1 示例 1

ssh 无交互登录到远程服务器

```
[root@xlj1 home]#vim expect.sh
 
#!/usr/bin/expect                   #需要用expect自己的解释器，不要写成bash否则无法识别
spawn ssh root@192.168.119.130      #开启一个程序，这个程序是ssh远程登录
expect {                            #捕获内容，当出现password的时候，就会向程序发送密码
         "password:"
        { send "123456\r"; }
}
interact                            #交互，否则会直接退出远程服务器
 
[root@xlj1 home]#chmod +x expect.sh              #需要加执行权限
[root@xlj1 home]#./expect.sh  
spawn ssh root@192.168.119.130
root@192.168.119.130's password: 
Last login: Wed Sep 15 22:39:40 2021 from 192.168.119.115
```

### 4.2 示例 2

在对方服务器上进行一下操作后再退出可执行以下脚本

```
[root@xlj1 home]#vim expect.sh
 
#!/usr/bin/expect
spawn ssh root@192.168.119.130
expect {
         "password:"
        { send "123456\r"; }
}
expect "#"                      #当捕获到#的时候
send "ls\r"                     #执行ls命令
send "ifconfig ens33\r"         #执行ifconfig ens33命令
send "exit\r"                   #执行完exit退出登录
expect eof                      #不用进行交互，意味着结束expect程序
 
[root@xlj1 home]#chmod +x expect.sh 
[root@xlj1 home]#./expect.sh 
spawn ssh root@192.168.119.130
root@192.168.119.130's password: 
Last login: Wed Sep 15 22:55:06 2021 from 192.168.119.115
[root@xlj1 ~]#ls
[root@xlj1 ~]#ifconfig ens33
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.119.130  netmask 255.255.255.0  broadcast 192.168.119.255
        inet6 fe80::2161:befa:6ffd:c44b  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:01:e6:0a  txqueuelen 1000  (Ethernet)
    ......
[root@localhost ~]#exit
登出
Connection to 192.168.119.130 closed.
```

### 4.3 示例 3

引用位置变量

```
[root@xlj1 /home]#vim expect1.sh 
#!/usr/bin/expect
set user root
set ip [lindex $argv 0]                #设置第一个位置变量为ip
set pass [lindex $argv 1]              #设置第二个位置变量为登陆密码
spawn ssh $user@$ip
expect {
        "password:"
        { send "$pass\r"; }
}
expect "#"
send "ls\r"
send "exit\r"
expect eof
 
[root@xlj1 home]#chmod +x expect1.sh 
[root@xlj1 home]#./expect1.sh 192.168.119.130 123456
spawn ssh root@192.168.119.130
root@192.168.119.130's password: 
Last login: Wed Sep 15 22:58:16 2021 from 192.168.119.115
[root@xlj1 ~]#ls
[root@xlj1 ~]#exit
登出
Connection to 192.168.119.130 closed.
```

4.4 示例 4

创建用户并设置用户密码

```
[root@xlj1 home]#vim user.sh
 
#!/bin/bash
username=$1
useradd $username
/usr/bin/expect <<-EOF
spawn passwd $username
expect {                              #获取的内容和发送的内容不能在同一行否则执行不成功
        "新的 密码"
        { send "123456\r";exp_continue }
        "重新输入新的 密码"
        { send "123456\r"; }
}
EOF
 
[root@xlj1 home]#chmod +x user.sh 
[root@xlj1 home]#./user.sh zhux
spawn passwd gebilaowang
更改用户 zhux 的密码 。
新的 密码：
无效的密码： 密码少于 8 个字符
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新
```