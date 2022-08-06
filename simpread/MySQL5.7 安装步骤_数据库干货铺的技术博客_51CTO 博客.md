> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/u_11583583/4877712)

> MySQL5.7 安装步骤，MySQL5.7 在性能、复制、安全等方面较之前版本有很大的提升，因此新建数据库时建议使用 MySQL5.7 版本

© 著作权归作者所有：来自 51CTO 博客作者 juncai_51 的原创作品，请联系作者获取转载授权，否则将追究法律责任  
[MySQL5.7 安装步骤](https://blog.51cto.com/u_11583583/4877712)  
[https://blog.51cto.com/u_11583583/4877712](https://blog.51cto.com/u_11583583/4877712)

本次进行 MySQL5.7 版本的安装，关于 MySQL 版本选择、官网下载地址、相关系统配置等操作可以参照之前的博文, 本文就不在赘述咯。可以参考历史文章处理，下面直奔主题，进行相关安装工作。

总体安装步骤概述如下：

*   *     
        
*   操作系统等相关配置设置
*   安装依赖包
*   创建用户
*   修改配置文件、创建相关数据目录、日志目录等并授权
*   运行安装命令，启动数据库
*   配置环境变量、服务等
    *     
        

因为我之前已经安装了 MySQL5.6，本次继续在同一台机器上安装一个 MySQL5.7 版本的实例，相关配置及依赖包等操作之前已处理完毕，

现在就接着后续操作进行了。如果首次安装，需先进行配置。

**一、准备工作**

安装必要依赖包

> yum install -y libaio

 创建用户

> groupadd mysql
> 
> useradd -r -g mysql mysql

上传安装包至 /usr/local  , 如上传至其他路径，以下操作过程中注意修改路径，或者在 / usr/local 目录创建软连接

解压

> tar -zxvf mysql-5.7.22-linux-glibc2.12-x86_64.tar.gz

![](https://s2.51cto.com/images/blog/202112/27142344_61c95bf061db699450.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

 创建软连接

> ln -s mysql-5.7.22-linux-glibc2.12-x86_64  /usr/local/mysql5.7

![](https://s2.51cto.com/images/blog/202112/27142344_61c95bf0adb0c9082.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=) 

修改目录权限

> cd  /usr/local
> 
> chown -R mysql:root mysql5.7

![](https://s2.51cto.com/images/blog/202112/27142344_61c95bf0cfd2e18225.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**二、创建相关数据目录、日志目录、配置文件等并授权**

  创建相关目录

> mkdir -p  /data/mysql/mysql3307/{data,logs,etc,tmp}

 配置文件

> vi  /data/mysql/mysql3307/etc/my3307.cnf

可以参考如下配置文件，注意根据自己的环境修改相关配置

```
[mysqld]

server-id = 463307
port = 3307
user = mysql
character_set_server=utf8mb4
skip_name_resolve = 1
max_connections = 800
max_connect_errors = 1000
datadir = /data/mysql/mysql3307/data      
transaction_isolation = READ-COMMITTED
explicit_defaults_for_timestamp = 1
join_buffer_size = 512M
tmp_table_size = 512M
tmpdir = /data/mysql/mysql3307/tmp
pid-file = /data/mysql/mysql3307/tmp/mysqld3307.pid
socket = /data/mysql/mysql3307/tmp/mysql3307.sock
max_allowed_packet = 128M
sql_mode = "STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER"
interactive_timeout = 1800
wait_timeout = 1800
read_buffer_size = 16M
read_rnd_buffer_size = 32M
sort_buffer_size = 32M
language = /usr/local/mysql5.7/share/english  

log_error =/data/mysql/mysql3307/logs/mysqld3307.log
slow_query_log = 1
slow_query_log_file = /data/mysql/mysql3307/logs/slow.log
log_queries_not_using_indexes = 1
log_slow_admin_statements = 1
log_slow_slave_statements = 1
log_throttle_queries_not_using_indexes = 10
expire_logs_days = 3
long_query_time = 2
min_examined_row_limit = 100

master_info_repository = TABLE
relay_log_info_repository = TABLE
log_bin = /data/mysql/mysql3307/logs/mysql_binlog
sync_binlog = 1
gtid_mode = on
enforce_gtid_consistency = 1
log_slave_updates
binlog_format = row
relay_log = /data/mysql/mysql3307/logs/relay.log
relay_log_recovery = 1
binlog_gtid_simple_recovery = 1
slave_skip_errors = ddl_exist_errors

innodb_page_size = 8192
innodb_buffer_pool_size = 1G    
innodb_buffer_pool_instances = 8
innodb_buffer_pool_load_at_startup = 1
innodb_buffer_pool_dump_at_shutdown = 1
innodb_lru_scan_depth = 2000
innodb_lock_wait_timeout = 5
innodb_io_capacity = 4000
innodb_io_capacity_max = 8000
innodb_flush_method = O_DIRECT
innodb_file_format = Barracuda
innodb_file_format_max = Barracuda
innodb_log_group_home_dir = /data/mysql/mysql3307/logs/  
innodb_undo_directory =  /data/mysql/mysql3307/logs/      
innodb_undo_logs = 128
innodb_undo_tablespaces = 3
innodb_flush_neighbors = 1
innodb_log_file_size = 512M               
innodb_log_buffer_size = 16777216
innodb_purge_threads = 4
innodb_large_prefix = 1
innodb_thread_concurrency = 64
innodb_print_all_deadlocks = 1
innodb_strict_mode = 1
innodb_sort_buffer_size = 67108864

plugin_dir=/usr/local/mysql5.7/lib/plugin      
plugin_load = "rpl_semi_sync_master=semisync_master.so;rpl_semi_sync_slave=semisync_slave.so"
loose_rpl_semi_sync_master_enabled = 1
loose_rpl_semi_sync_slave_enabled = 1
loose_rpl_semi_sync_master_timeout = 5000

[mysqld-5.7]
innodb_buffer_pool_dump_pct = 40
innodb_page_cleaners = 4
innodb_undo_log_truncate = 1
innodb_max_undo_log_size = 1G
innodb_purge_rseg_truncate_frequency = 128
binlog_gtid_simple_recovery=1
log_timestamps=system
transaction_write_set_extraction=MURMUR32
show_compatibility_56=on

1.
2.
3.
4.
5.
6.
7.
8.
9.
10.
11.
12.
13.
14.
15.
16.
17.
18.
19.
20.
21.
22.
23.
24.
25.
26.
27.
28.
29.
30.
31.
32.
33.
34.
35.
36.
37.
38.
39.
40.
41.
42.
43.
44.
45.
46.
47.
48.
49.
50.
51.
52.
53.
54.
55.
56.
57.
58.
59.
60.
61.
62.
63.
64.
65.
66.
67.
68.
69.
70.
71.
72.
73.
74.
75.
76.
77.
78.
79.
80.
81.
82.
83.
84.
85.
86.
87.
88.
89.
90.
91.
92.
93.
94.
95.
96.
97.
98.
99.
100.
101.
102.
103.
104.
105.
106.
107.
108.
109.
110.
111.
112.
113.

```

授权

> chown -R mysql:mysql /data/mysql/mysql3307

![](https://s2.51cto.com/images/blog/202112/27142344_61c95bf0ef17399679.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

 **三、 安装及启动**

**1. 指定本实例的配置文件进行安装**

# 建议写全路径运行命令进行安装

> /usr/local/mysql5.7/bin/mysqld  --defaults-file=/data/mysql/mysql3307/etc/my3307.cnf --initialize --user=mysql

注意：mysql5.7 安装后默认会产生一个临时密码，存放于错误日志里，可以查看错误日志进行查看

# 查看日志

> vim /data/mysql/mysql3307/logs/mysqld3307.log

此时可以看到临时密码，并查看是否有错误产生

![](https://s2.51cto.com/images/blog/202112/27142345_61c95bf1955fa84493.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

> **如果不想产生临时密码，可以运行如下命令进行操作，建议使用 initialize 产生随机密码（下面的语句只是科普使用，无需执行）**​
> 
> /usr/local/mysql5.7/bin/mysqld  --defaults-file=/data/mysql/mysql3307/etc/my3307.cnf --initialize-insecure  --user=mysql  
> 
> 即：使用 --initialize 生成随机密码，使用 --initialize-insecure 生成空密码。

注意：如果安装过程中出现错误，需要将 datadir 及 logs 下对应的文件删除，并将错误原因查出解决后再运行安装命令，否则将无法再次安装。

**2. 启动 MySQL 实例**

#启动数据库，并在后台运行（执行下面命令时多次回车，专为小白而备注）

/usr/local/mysql5.7/bin/mysqld_safe  --defaults-file=/data/mysql/mysql3307/etc/my3307.cnf &         

![](https://s2.51cto.com/images/blog/202112/27142345_61c95bf1ba40c38725.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

# 查看进程

ps -ef|grep mysql

![](https://s2.51cto.com/images/blog/202112/27142345_61c95bf1e40ce4454.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

# 登录 mysql: 此时需要用到临时密码（存放于错误日志中）

/usr/local/mysql5.7/bin/mysql -uroot -p'c,/yqCh:&9pS' -P3307 --socket=/data/mysql/mysql3307/tmp/mysql3307.sock  ## 标红的为临时密码

![](https://s2.51cto.com/images/blog/202112/27142346_61c95bf21824643961.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

# 进入后需先修改密码才能进行其他操作，如下所示

![](https://s2.51cto.com/images/blog/202112/27142346_61c95bf23658379724.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

**3. 停止数据库**

> shutdown 

  MySQL 5.7 可以在 sql 里运行 shutdown 命令关闭数据库（mysql5.6 需要用 mysqladmin 进行）

![](https://s2.51cto.com/images/blog/202112/27142346_61c95bf27af9a2336.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

# mysqladmin 方式关闭 （mysql5.6 采用此方法关闭）

> /usr/local/mysql5.7/bin/mysqladmin  -uroot -p123456  --port=3307  --socket=/data/mysql/mysql3307/tmp/mysql3307.sock  shutdown

![](https://s2.51cto.com/images/blog/202112/27142346_61c95bf29a4c635083.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

##  至此 mysql5.6  mysql5.7 版本的安装及启动停止就已完成了， MySQL8.0 的安装与 MySQL5.7 类似，但是新特性较多，后续也会进行找机会再举例说明。

以上步骤中如有疑问或错误 欢迎指正。