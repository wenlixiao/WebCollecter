> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/mrjiang-test/p/13406108.html)

一、检查系统是否自带了 mariadb 数据库，如果有自带，我们就卸载

a. 检查是否有安装

```
# yum list installed |grep mariadb

```

 ![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730173102492-1947136287.png)

b. 这里系统自带了，我们把它卸载

```
# yum remove mariadb-libs.x86_64

```

二、首先我们要先去 mysql 官网下载 yum 存储库的 rpm 包 ([https://dev.mysql.com/downloads/repo/yum/](https://dev.mysql.com/downloads/repo/yum/))，下载对应自己系统的，接着把它用 FileZila 等工具上传到 Linux 操作系统上面，上传到任意目录都行，只要你记得

![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730173811303-1222005150.png)

我这里是上传到根目录

![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730174211367-1039318792.png)

 三、安装你上传到系统的 rpm 包

a. 安装 mysql 的 rpm 包

```
# rpm -ivh mysql80-community-release-el7-3.noarch.rpm

```

![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730174453007-1081533519.png)

b. 安装好了之后我们可以用下面这条命令查看 yum 仓库是否存在 mysql-server 安装包

```
# yum list |grep mysql-community-server

```

![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730180929488-539353333.png)

 四、由于官网不提供 mysql8.0 的 rpm 包，我们要装的是 mysql5.7，那么我们可以通过修改文件来指定默认安装 mysql5.7

a. 使用下面这条命令我们可以看见，mysql57 是禁用的，mysql80 是启用的 (如果你要装 8.0 可以直接忽略这一步)

```
# yum repolist all|grep mysql

```

![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730174741477-1196042907.png)

 b. 修改 yum 源中 mysql 的源文件，找到你要安装的 mysql 版本代码，enabled=1 代表启用，enabled=0 代表禁用，同时禁用其它版本 (只需要改带版本号的代码)，如果你要安装其它版本的也是一样，记得禁用其它版本，只存在一个版本可安装！

```
# vim /etc/yum.repos.d/mysql-community.repo

```

 ![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730175644722-1535099343.png)

 c. 这时候我们再来看一下是否已经启用 mysql5.7 并禁用 mysql8.0 了

```
# yum repolist all|grep mysql

```

 ![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730180345655-1254677356.png)

 五、接下来我们就开始安装 mysql5.7 了

```
# yum install -y mysql-community-server

```

 六、安装结束了之后我们要启动 mysql 并设置开机自启动

```
# systemctl start mysqld   ---启动mysql
# systemctl enable mysqld   ---设置mysql开机自启动
# service mysqld status   ---查看mysql服务状态

```

![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730182021172-813335615.png)

 七、尝试登录 mysql，安装 mysql 之后会给一个初始密码，我们可以通过下面命令来获取密码并登录

a. 获取初始密码

```
# grep 'temporary password' /var/log/mysqld.log

```

![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730182538149-1654384368.png)

b. 使用这个初始密码登录 mysql

```
# mysql -u root -p    ---输入这条命令后回车，粘贴你的初始密码后再回车

```

![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730182715107-501389852.png)

ps: 登录之后不改初始密码我们没法使用数据库，会出现下图的情况

![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730183506722-1706362417.png)

c. 接下来我们改密码，mysql 默认密码策略要满足大小写 + 数字 + 特殊符号，如果你接受这个策略，那么直接使用下面命令，改完记得刷新权限

```
改密码命令：
# ALTER USER username@hostname IDENTIFIED BY "new password";
例子：
# ALTER USER root@localhost IDENTIFIED BY "Mysql12345678.";
刷新权限：
FLUSH PRIVILEGES;

```

 那么我们不能接受这个策略呢？可以使用下面的方法改密码策略并修改密码：

```
修改密码总长度：
# SET GLOBAL validate_password_length=1;
修改密码强度策略：
# SET GLOBAL validate_password_policy=0;

```

 ![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730185321123-461494973.png)

 修改了密码策略之后我们再进行修改密码，记得刷新权限，接下来就可以使用改的简单密码进行登录了

```
改密码命令：
# ALTER USER username@hostname IDENTIFIED BY "new password";
例子：
# ALTER USER root@localhost IDENTIFIED BY "12345";
刷新权限：
FLUSH PRIVILEGES;

```

![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730185556528-1799352278.png)

八、现在的 mysql 还不能被远程访问，接下来我们设置远程访问

a. 登录 mysql, 使用下面命令进行设置，我这里是直接把 root 设置了所有人可以访问，这个是不安全的，通常做法是新建一个 mysql 用户去设置，方法在这里不多赘述，自行百度

```
列出数据库的库：
# show databases;
使用mysql库：
# use mysql;
查询user和host：
# select user,host from user;
把root用户改为远程登录用户，%表示任何地址都能访问：
# update user set host="%" where user="root";

```

 ![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730190905588-1026327390.png)

设置完远程登录我们就可以用 Windows 的数据库连接软件进行远程访问了

![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730201521890-1828070024.png)

 b. 接下来我们设置字符集

先使用下面命令查看字符集：

```
# show variables like '%character%';

```

 ![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730191547519-1398375320.png)

我们可以发现有几个不是 utf8，那么接下来我们来设置为 utf8：

先使用 \q 命令退出 mysql，接下来使用下面命令并添加相应内容

```
编辑my.cnf文件
# vim /etc/my.cnf
#添加如下内容在文件末尾
 
character-set-server=utf8
 
[client]
 
default-character-set=utf8
 
[mysql]
 
default-character-set=utf8

```

 ![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730195401153-2037790545.png)

 接下来使用下面命令重启 mysql，再登录 mysql，查看字符集是否已经修改

```
# systemctl restart mysqld   ---重启mysql
# mysql -uroot -p   ---登录mysql
#show variables like '%character%';   ---查看字符集

```

 ![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730195909092-1421978635.png)

 九、到这里安装就结束了，接下来我们做一个收尾工作，卸载掉 mysql 的. noarch

先使用下面命令查看是否存在这个包：

```
# yum list installed mysql80-community-release

```

 ![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730201135378-1907464696.png)

 这个就是我们的卸载对象，使用下面命令进行卸载：

```
# yum remove -y mysql80-community-release.noarch

```

![](https://img2020.cnblogs.com/blog/1996853/202007/1996853-20200730201323403-1560600668.png)