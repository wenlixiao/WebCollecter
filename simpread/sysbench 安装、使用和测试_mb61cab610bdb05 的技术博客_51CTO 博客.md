> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/u_15474913/5387543)

> sysbench 安装、使用和测试，sysbench 是一个开源的、模块化的、跨平台的多线程性能测试工具，可以用来进行 CPU、内存、磁盘 I/O、线程、数据库的性能测试。

© 著作权归作者所有：来自 51CTO 博客作者 zhoujy_dba 的原创作品，请联系作者获取转载授权，否则将追究法律责任  
[sysbench 安装、使用和测试](https://blog.51cto.com/u_15474913/5387543)  
[https://blog.51cto.com/u_15474913/5387543](https://blog.51cto.com/u_15474913/5387543)

**摘要：**  
      sysbench 是一个开源的、模块化的、跨平台的多线程性能测试工具，可以用来进行 CPU、内存、磁盘 I/O、线程、数据库的性能测试。目前支持的数据库有 MySQL、Oracle 和 PostgreSQL。当前功能允许测试的系统参数有：

```
file I/O performance （文件I / O性能）
scheduler performance （调度性能）
memory allocation and transfer speed （内存分配和传输速度）
POSIX threads implementation performance （POSIX线程执行绩效）
database server performance (OLTP benchmark) （数据库服务器性能）

1.
2.
3.
4.
5.

```

**安装：** 1)：Ubuntu 系统可以直接 apt，如：

```
apt-get install sysbench

1.

```

      2)：其他系统的则可以编译安装 [在 / home/zhoujy / 目录下]：（安装前先装 automake，libtool）

```
wget http://nchc.dl.sourceforge.net/project/sysbench/sysbench/0.4.12/sysbench-0.4.12.tar.gz
tar zxvf sysbench-0.4.12.tar.gz

1.
2.

```

```
进入解压目录，并且创建安装目录：
root@m2:/home/zhoujy# cd sysbench-0.4.12/
root@m2:/home/zhoujy/sysbench-0.4.12# mkdir /usr/sysbench/

1.
2.
3.

```

```
准备编译
root@m2:/home/zhoujy/sysbench-0.4.12# apt-get install automake
root@m2:/home/zhoujy/sysbench-0.4.12#apt-get install libtool
root@m2:/home/zhoujy/sysbench-0.4.12# ./autogen.sh

1.
2.
3.
4.

```

要是出现：perl: warning: Falling back to the standard locale ("C")。则需要设置 locale:

```
echo "export LC_ALL=C" >> /root/.bashrc
source /root/.bashrc

1.
2.

```

要是没有安装开发包，即 / usr/include/ 目录下面没有 mysql 文件夹。则需要执行安装 (版本为 12.04):

```
sudo apt-get install libmysqlclient-dev
sudo apt-get install libmysqld-dev
sudo apt-get install libmysqld-pic

1.
2.
3.

```

执行 configure 操作：

```
./configure --prefix=/usr/sysbench/ --with-mysql-includes=/usr/include/mysql/ --with-mysql-libs=/usr/lib/mysql/ --with-mysql

1.

```

```
说明：
--prefix=/usr/sysbench/                    ：指定sysbench的安装目录。
--with-mysql-includes=/usr/include/mysql/  ：指定安装mysql时候的includes目录。
--with-mysql-libs=/usr/lib/mysql/          ：指定装mysql时候的lib目录。
--with-mysql                               ：sysbench默认支持mysql，如果需要测试oracle或者pgsql则需要制定–with-oracle或者–with-pgsql。

1.
2.
3.
4.
5.

```

在这里需要先执行：

```
cp /usr/bin/libtool /home/zhoujy/sysbench-0.4.12/libtool

1.

```

再 make 和 make install。否者会出现 libtool 报出的 Xsysbench: command not found  错误，则表示编译文件包的 libtool 版本太低，需要替换。

**选项说明 (通用)：**

```
root@db2:~# sysbench 
Missing required command argument.
Usage: #使用方法
  sysbench [general-options]... --test=<test-name> [test-options]... command

General options: #通用选项
  --num-threads=N            number of threads to use [1] #创建测试线程的数目。默认为1.
  --max-requests=N           limit for total number of requests [10000] #请求的最大数目。默认为10000，0代表不限制。
  --max-time=N               limit for total execution time in seconds [0] #最大执行时间，单位是s。默认是0,不限制。
  --forced-shutdown=STRING   amount of time to wait after --max-time before forcing shutdown [off] #超过max-time强制中断。默认是off。
  --thread-stack-size=SIZE   size of stack per thread [32K] #每个线程的堆栈大小。默认是32K。
  --init-rng=[on|off]        initialize random number generator [off] #在测试开始时是否初始化随机数发生器。默认是off。
  --test=STRING              test to run #指定测试项目名称。
  --debug=[on|off]           print more debugging info [off] #是否显示更多的调试信息。默认是off。
  --validate=[on|off]        perform validation checks where possible [off] #在可能情况下执行验证检查。默认是off。
  --help=[on|off]            print help and exit #帮助信息。
  --version=[on|off]         print version and exit #版本信息。

Compiled-in tests: #测试项目
  fileio - File I/O test #IO
  cpu - CPU performance test #CPU
  memory - Memory functions speed test #内存
  threads - Threads subsystem performance test #线程
  mutex - Mutex performance test #互斥性能测试
  oltp - OLTP test # 数据库，事务处理

Commands: prepare：测试前准备工作； run：正式测试 cleanup：测试后删掉测试数据 help version

See 'sysbench --test=<name> help' for a list of options for each test. #查看每个测试项目的更多选项列表

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

```

**更多选项：**  
1):sysbench  --test=fileio help

![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e6802c87083.gif)![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e75b3964944.gif)View Code

```
root@db2:~# sysbench  --test=fileio help
sysbench 0.4.12:  multi-threaded system evaluation benchmark

fileio options:
  --file-num=N   创建测试文件的数量。默认是128
  --file-block-size=N  测试时文件块的大小。默认是16384(16K)
  --file-total-size=SIZE   测试文件的总大小。默认是2G
  --file-test-mode=STRING  文件测试模式{seqwr(顺序写), seqrewr(顺序读写), seqrd(顺序读), rndrd(随机读), rndwr(随机写), rndrw(随机读写)}
  --file-io-mode=STRING   文件操作模式{sync(同步),async(异步),fastmmap(快速map映射),slowmmap(慢map映射)}。默认是sync
  --file-extra-flags=STRING   使用额外的标志来打开文件{sync,dsync,direct} 。默认为空
  --file-fsync-freq=N   执行fsync()的频率。(0 – 不使用fsync())。默认是100
  --file-fsync-all=[on|off] 每执行完一次写操作就执行一次fsync。默认是off
  --file-fsync-end=[on|off] 在测试结束时才执行fsync。默认是on
  --file-fsync-mode=STRING  使用哪种方法进行同步{fsync, fdatasync}。默认是fsync
  --file-merged-requests=N   如果可以，合并最多的IO请求数(0 – 表示不合并)。默认是0
  --file-rw-ratio=N     测试时的读写比例。默认是1.5

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

```

2):sysbench  --test=cpu help

![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e6802c87083.gif)![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e75b3964944.gif)View Code

```
--cpu-max-prime=N  最大质数发生器数量。默认是10000

1.

```

3):sysbench  --test=memory help

![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e6802c87083.gif)![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e75b3964944.gif)View Code

```
root@db2:~# sysbench  --test=memory help
sysbench 0.4.12:  multi-threaded system evaluation benchmark

memory options:
  --memory-block-size=SIZE  测试时内存块大小。默认是1K
  --memory-total-size=SIZE    传输数据的总大小。默认是100G
  --memory-scope=STRING    内存访问范围{global,local}。默认是global
  --memory-hugetlb=[on|off]  从HugeTLB池内存分配。默认是off
  --memory-oper=STRING     内存操作类型。{read, write, none} 默认是write
  --memory-access-mode=STRING存储器存取方式{seq,rnd} 默认是seq

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

```

4):sysbench  --test=threads help

![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e6802c87083.gif)![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e75b3964944.gif)View Code

```
sysbench 0.4.12:  multi-threaded system evaluation benchmark

threads options:
  --thread-yields=N   每个请求产生多少个线程。默认是1000
  --thread-locks=N    每个线程的锁的数量。默认是8

1.
2.
3.
4.
5.

```

5):sysbench  --test=mutex help

![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e6802c87083.gif)![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e75b3964944.gif)View Code

```
root@db2:~# sysbench  --test=mutex help
sysbench 0.4.12:  multi-threaded system evaluation benchmark

mutex options:

  --mutex-num=N    数组互斥的总大小。默认是4096
  --mutex-locks=N    每个线程互斥锁的数量。默认是50000
  --mutex-loops=N    内部互斥锁的空循环数量。默认是10000

1.
2.
3.
4.
5.
6.
7.
8.

```

**6): sysbench --test=oltp help**

```
root@db2:~# sysbench --test=oltp help
sysbench 0.4.12:  multi-threaded system evaluation benchmark

oltp options:
  --oltp-test-mode=STRING    执行模式{simple,complex(advanced transactional),nontrx(non-transactional),sp}。默认是complex
  --oltp-reconnect-mode=STRING 重新连接模式{session(不使用重新连接。每个线程断开只在测试结束),transaction(在每次事务结束后重新连接),query(在每个SQL语句执行完重新连接),random(对于每个事务随机选择以上重新连接模式)}。默认是session
  --oltp-sp-name=STRING   存储过程的名称。默认为空
  --oltp-read-only=[on|off]  只读模式。Update，delete，insert语句不可执行。默认是off
  --oltp-skip-trx=[on|off]   省略begin/commit语句。默认是off
  --oltp-range-size=N      查询范围。默认是100
  --oltp-point-selects=N          number of point selects [10]
  --oltp-simple-ranges=N          number of simple ranges [1]
  --oltp-sum-ranges=N             number of sum ranges [1]
  --oltp-order-ranges=N           number of ordered ranges [1]
  --oltp-distinct-ranges=N        number of distinct ranges [1]
  --oltp-index-updates=N          number of index update [1]
  --oltp-non-index-updates=N      number of non-index updates [1]
  --oltp-nontrx-mode=STRING   查询类型对于非事务执行模式{select, update_key, update_nokey, insert, delete} [select]
  --oltp-auto-inc=[on|off]      AUTO_INCREMENT是否开启。默认是on
  --oltp-connect-delay=N     在多少微秒后连接数据库。默认是10000
  --oltp-user-delay-min=N    每个请求最短等待时间。单位是ms。默认是0
  --oltp-user-delay-max=N    每个请求最长等待时间。单位是ms。默认是0
  --oltp-table-name=STRING  测试时使用到的表名。默认是sbtest
  --oltp-table-size=N         测试表的记录数。默认是10000
  --oltp-dist-type=STRING    分布的随机数{uniform(均匀分布),Gaussian(高斯分布),special(空间分布)}。默认是special
  --oltp-dist-iter=N    产生数的迭代次数。默认是12
  --oltp-dist-pct=N    值的百分比被视为'special' (for special distribution)。默认是1
  --oltp-dist-res=N    ‘special’的百分比值。默认是75

General database options:
  --db-driver=STRING  数据库类型，指定数据库驱动程序('help' to get list of available drivers)
  --db-ps-mode=STRING 数据库预处理模式{auto, disable} [auto]
Compiled-in database drivers:
    mysql - MySQL driver
mysql options:
  --mysql-host=[LIST,...]       MySQL server host [localhost]
  --mysql-port=N                MySQL server port [3306]
  --mysql-socket=STRING         MySQL socket
  --mysql-user=STRING           MySQL user [sbtest]
  --mysql-password=STRING       MySQL password []
  --mysql-db=STRING             MySQL database name [sbtest]
  --mysql-table-engine=STRING   storage engine to use for the test table {myisam,innodb,bdb,heap,ndbcluster,federated} [innodb]
  --mysql-engine-trx=STRING     whether storage engine used is transactional or not {yes,no,auto} [auto]
  --mysql-ssl=[on|off]          use SSL connections, if available in the client library [off]
  --myisam-max-rows=N           max-rows parameter for MyISAM tables [1000000]
  --mysql-create-options=STRING additional options passed to CREATE TABLE []

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

```

**测试:**  
1) 测试 CPU: sysbench --test=cpu --cpu-max-prime=2000 run

![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e6802c87083.gif)![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e75b3964944.gif)View Code

```
root@db2:~# sysbench --test=cpu --cpu-max-prime=2000 run
sysbench 0.4.12:  multi-threaded system evaluation benchmark

Running the test with following options:
Number of threads: 1

Doing CPU performance benchmark

Threads started!
Done.

Maximum prime number checked in CPU test: 2000

Test execution summary:
    total time:                          3.7155s
    total number of events:              10000
    total time taken by event execution: 3.7041
    per-request statistics:
         min:                                  0.36ms
         avg:                                  0.37ms
         max:                                  2.53ms
         approx.  95 percentile:               0.37ms

Threads fairness:
    events (avg/stddev):           10000.0000/0.00
    execution time (avg/stddev):   3.7041/0.00

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

```

2) 测试线程：sysbench  --test=threads --num-threads=500 --thread-yields=100 --thread-locks=4 run

![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e6802c87083.gif)![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e75b3964944.gif)View Code

```
root@db2:~# sysbench  --test=threads --num-threads=500 --thread-yields=100 --thread-locks=4 run
sysbench 0.4.12:  multi-threaded system evaluation benchmark

Running the test with following options:
Number of threads: 500

Doing thread subsystem performance test
Thread yields per test: 100 Locks used: 4
Threads started!
Done.

Test execution summary:
    total time:                          1.0644s
    total number of events:              10000
    total time taken by event execution: 501.3952
    per-request statistics:
         min:                                  0.05ms
         avg:                                 50.14ms
         max:                                587.05ms
         approx.  95 percentile:             190.28ms

Threads fairness:
    events (avg/stddev):           20.0000/4.72
    execution time (avg/stddev):   1.0028/0.01

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

```

3) 测试 IO：--num-threads 开启的线程     --file-total-size 总的文件大小  
1，prepare 阶段，生成需要的测试文件，完成后会在当前目录下生成很多小文件。  
sysbench --test=fileio --num-threads=16 --file-total-size=2G --file-test-mode=rndrw prepare      
2，run 阶段  
sysbench --test=fileio --num-threads=20 --file-total-size=2G --file-test-mode=rndrw run  
3，清理测试时生成的文件  
sysbench --test=fileio --num-threads=20 --file-total-size=2G --file-test-mode=rndrw cleanup

![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e6802c87083.gif)![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e75b3964944.gif)View Code

```
root@db2:~/io# sysbench --test=fileio --num-threads=16 --file-total-size=2G --file-test-mode=rndrw prepare
sysbench 0.4.12:  multi-threaded system evaluation benchmark

128 files, 16384Kb each, 2048Mb total
Creating files for the test...
root@db2:~/io# sysbench --test=fileio --num-threads=20 --file-total-size=2G --file-test-mode=rndrw run
sysbench 0.4.12:  multi-threaded system evaluation benchmark

Running the test with following options:
Number of threads: 20

Extra file open flags: 0
128 files, 16Mb each
2Gb total file size
Block size 16Kb
Number of random requests for random IO: 10000
Read/Write ratio for combined random IO test: 1.50
Periodic FSYNC enabled, calling fsync() each 100 requests.
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing random r/w test
Threads started!
Done.

Operations performed:  6010 Read, 3997 Write, 12803 Other = 22810 Total
Read 93.906Mb  Written 62.453Mb  Total transferred 156.36Mb  (6.6668Mb/sec)
  426.68 Requests/sec executed

Test execution summary:
    total time:                          23.4534s
    total number of events:              10007
    total time taken by event execution: 111.5569
    per-request statistics:
         min:                                  0.01ms
         avg:                                 11.15ms
         max:                                496.18ms
         approx.  95 percentile:              53.05ms

Threads fairness:
    events (avg/stddev):           500.3500/37.50
    execution time (avg/stddev):   5.5778/0.21

root@db2:~/io# sysbench --test=fileio --num-threads=20 --file-total-size=2G --file-test-mode=rndrw cleanup
sysbench 0.4.12:  multi-threaded system evaluation benchmark

Removing test files...

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

```

4) 测试内存：sysbench --test=memory --memory-block-size=8k --memory-total-size=1G run

![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e6802c87083.gif)![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e75b3964944.gif)View Code

```
sysbench 0.4.12:  multi-threaded system evaluation benchmark
Operations performed: 1310720 (396525.32 ops/sec)
10240.00 MB transferred (3097.85 MB/sec)
Test execution summary:
total time:                          3.3055s
total number of events:              1310720
total time taken by event execution: 205.0560
per-request statistics:
min:                                  0.00ms
avg:                                  0.16ms
max:                               1066.04ms
approx.  95 percentile:               0.02ms
Threads fairness:
events (avg/stddev):           13107.2000/3870.38
execution time (avg/stddev):   2.0506/0.28

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

```

5) 测试 mutex：sysbench –test=mutex –num-threads=100 –mutex-num=1000 –mutex-locks=100000 –mutex-loops=10000 run

![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e6802c87083.gif)![](https://s2.51cto.com/images/blog/202206/15094726_62a93a2e75b3964944.gif)View Code

```
Test execution summary:
total time:                          12.5606s
total number of events:              100
total time taken by event execution: 1164.4236
per-request statistics:
min:                               9551.87ms
avg:                              11644.24ms
max:                              12525.32ms
approx.  95 percentile:           12326.25ms
Threads fairness:
events (avg/stddev):           1.0000/0.00
execution time (avg/stddev):   11.6442/0.59

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

```

**6) 测试 OLTP：**  
1，prepare 阶段，生成需要的测试表  
sysbench --test=oltp --mysql-table-engine=innodb --mysql-host=192.168.X.X --mysql-db=test --oltp-table-size=500000 --mysql-user=root --mysql-password=123456 prepare  
2，run 阶段  
sysbench --num-threads=16 --test=oltp --mysql-table-engine=innodb --mysql-host=192.168.x.x --mysql-db=test --oltp-table-size=500000 --mysql-user=root --mysql-password=123456 run  
3，清理测试时生成的测试表  
sysbench --num-threads=16 --test=oltp --mysql-table-engine=innodb --mysql-host=192.168.x.x --mysql-db=test --oltp-table-size=500000 --mysql-user=root --mysql-password=123456 cleanup

```
root@db2:~# sysbench --test=oltp --mysql-table-engine=innodb --mysql-host=192.168.x.x --mysql-db=rep_test --oltp-table-size=1000000 --mysql-user=root --mysql-password=123456 prepare
sysbench 0.4.12:  multi-threaded system evaluation benchmark

No DB drivers specified, using mysql
Creating table 'sbtest'...
Creating 1000000 records in table 'sbtest'...
root@db2:~# sysbench --num-threads=16 --test=oltp --mysql-table-engine=innodb --mysql-host=192.168.x.x --mysql-db=rep_test --oltp-table-size=1000000 --mysql-user=root --mysql-password=123456 run    
sysbench 0.4.12:  multi-threaded system evaluation benchmark

No DB drivers specified, using mysql
Running the test with following options:
Number of threads: 16

Doing OLTP test.
Running mixed OLTP test
Using Special distribution (12 iterations,  1 pct of values are returned in 75 pct cases)
Using "BEGIN" for starting transactions
Using auto_inc on the id column
Maximum number of requests for OLTP test is limited to 10000
Threads started!
Done.

OLTP test statistics:
    queries performed:
        read:                            140000
        write:                           50000
        other:                           20000
        total:                           210000

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

```

```
#—-事务数总计，每秒的事务处理量
    transactions:                        10000  (356.98 per sec.)
    deadlocks:                           0      (0.00 per sec.)
    read/write requests:                 190000 (6782.56 per sec.)
    other operations:                    20000  (713.95 per sec.)

Test execution summary:
    total time:                          28.0130s
    total number of events:              10000
    total time taken by event execution: 447.7731
    per-request statistics:
         min:                                  3.91ms
         avg:                                 44.78ms
         max:                                207.61ms
         approx.  95 percentile:              76.48ms

Threads fairness:
    events (avg/stddev):           625.0000/22.96
    execution time (avg/stddev):   27.9858/0.01

root@db2:~# sysbench --num-threads=16 --test=oltp --mysql-table-engine=innodb --mysql-host=192.168.x.x --mysql-db=rep_test --oltp-table-size=1000000 --mysql-user=root --mysql-password=123456 cleanup
sysbench 0.4.12:  multi-threaded system evaluation benchmark

No DB drivers specified, using mysql
Dropping table 'sbtest'...
Done.

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

```

测试表信息：

```
mysql> desc sbtest;
+-------+------------------+------+-----+---------+----------------+
| Field | Type             | Null | Key | Default | Extra          |
+-------+------------------+------+-----+---------+----------------+
| id    | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| k     | int(10) unsigned | NO   | MUL | 0       |                |
| c     | char(120)        | NO   |     |         |                |
| pad   | char(60)         | NO   |     |         |                |
+-------+------------------+------+-----+---------+----------------+
4 rows in set (0.01 sec)

mysql> show create table sbtest\G;
*************************** 1. row ***************************
       Table: sbtest
Create Table: CREATE TABLE `sbtest` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `k` int(10) unsigned NOT NULL DEFAULT '0',
  `c` char(120) NOT NULL DEFAULT '',
  `pad` char(60) NOT NULL DEFAULT '',
  PRIMARY KEY (`id`),
  KEY `k` (`k`)
) ENGINE=InnoDB AUTO_INCREMENT=1000001 DEFAULT CHARSET=utf8
1 row in set (0.01 sec)

ERROR: 
No query specified

mysql> select count(*) from sbtest;
+----------+
| count(*) |
+----------+
|  1000000 |
+----------+
1 row in set (0.30 sec)
mysql> desc sbtest;
ERROR 1146 (42S02): Table 'rep_test.sbtest' doesn't exist

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

```

可以用这个测试：

```
sysbench --num-threads=4 --test=oltp --oltp-reconnect-mode=random --mysql-table-engine=innodb --mysql-host=192.168.200.201 --mysql-db=rep_test --ol
tp-table-size=500000 --mysql-user=zjy --mysql-password=1234#

1.
2.

```

**################################### 更新 2016-07-27**

今天使用 sysbench 的时候，发现 0.5 版本已经没有 --test=oltp 的参数，现在就更新说明下 0.5 版本的 sysbench 如何使用 oltp 的测试。

**① 安装：通过 apt-get install sysbench**

```
root@t22:~# sysbench --test=oltp help
sysbench 0.5:  multi-threaded system evaluation benchmark

PANIC: unprotected error in call to Lua API (cannot open oltp: No such file or directory)

1.
2.
3.
4.

```

sysbench 0.5 相比 0.4 版本有一些变化，包括 oltp 测试结合了 lua 脚本。通过 **dpkg -L sysbench** 查看相关文件：

```
root@t22:~# dpkg -L sysbench
/.
/usr
/usr/bin
/usr/bin/sysbench
/usr/share
/usr/share/doc
/usr/share/doc/sysbench
/usr/share/doc/sysbench/copyright
/usr/share/doc/sysbench/TODO
/usr/share/doc/sysbench/changelog.Debian.gz
/usr/share/doc/sysbench/README
/usr/share/doc/sysbench/manual.html
/usr/share/doc/sysbench/tests
/usr/share/doc/sysbench/tests/db
/usr/share/doc/sysbench/tests/db/oltp_simple.lua
/usr/share/doc/sysbench/tests/db/insert.lua
/usr/share/doc/sysbench/tests/db/select_random_ranges.lua
/usr/share/doc/sysbench/tests/db/oltp.lua
/usr/share/doc/sysbench/tests/db/common.lua
/usr/share/doc/sysbench/tests/db/select_random_points.lua
/usr/share/doc/sysbench/tests/db/delete.lua
/usr/share/doc/sysbench/tests/db/update_non_index.lua
/usr/share/doc/sysbench/tests/db/update_index.lua
/usr/share/doc/sysbench/tests/db/parallel_prepare.lua
/usr/share/doc/sysbench/tests/db/select.lua

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

```

**②：参数介绍**

```
--test=tests/db/oltp.lua 表示调用 tests/db/oltp.lua 脚本进行 oltp 模式测试
--mysql-table-engine=innodb 表示选择测试表的存储引擎
--oltp_tables_count=10 表示会生成 10 个测试表
--oltp-table-size=100000 表示每个测试表填充数据量为 100000
--rand-init=on 表示每个测试表都是用随机数据来填充的
如果在本机，也可以使用 –mysql-socket 指定 socket 文件来连接。加载测试数据时长视数据量而定，若过程比较久需要稍加耐心等待。

1.
2.
3.
4.
5.
6.

```

```
--num-threads=8 表示发起 8个并发连接
--oltp-read-only=off 表示不要进行只读测试，也就是会采用读写混合模式测试
--report-interval=10 表示每10秒输出一次测试进度报告
--rand-type=uniform 表示随机类型为固定模式，其他几个可选随机模式：uniform(固定),gaussian(高斯),special(特定的),pareto(帕累托)
--max-time=120 表示最大执行时长为 120秒
--max-requests=0 表示总请求数为 0，因为上面已经定义了总执行时长，所以总请求数可以设定为 0；也可以只设定总请求数，不设定最大执行时长
--percentile=99 表示设定采样比例，默认是 95%，即丢弃1%的长请求，在剩余的99%里取最大值

1.
2.
3.
4.
5.
6.
7.

```

具体的说明可以看​ [​sysbench 安装、使用、结果解读​](http://imysql.com/2014/10/17/sysbench-full-user-manual.shtml)​​和​ [​使用 sysbench 对 mysql 压力测试​](http://seanlook.com/2016/03/28/mysql-sysbench/)​。

**③：使用说明**

**初始化数据：prepare。**在本地数据库的 dba_test 库中，初始化三张表（sbtest1、sbtest2、sbtest3），存储引擎是 innodb，每张表 50 万数据。

```
sysbench --test=/usr/share/doc/sysbench/tests/db/oltp.lua --mysql-table-engine=innodb --mysql-host=127.0.0.1 --mysql-db=dba_test --oltp-table-size=500000 --oltp_tables_count=3 --rand-init=on --mysql-user=zjy --mysql-password=zjy  prepare

1.

```

因为是本机测试，所以也可以用也可以使用 --mysql-socket 指定 socket 文件来连接。

**测试：run。**模拟对 3 个表并发 OLTP 测试，每个表 50 万行记录，持续压测时间为 5 分钟。

```
sysbench --test=/usr/share/doc/sysbench/tests/db/oltp.lua --mysql-table-engine=innodb --mysql-host=127.0.0.1 --mysql-db=dba_test --num-threads=8 --oltp-table-size=500000 --oltp_tables_count=3 --oltp-read-only=off --report-interval=10 --rand-type=uniform --max-time=600 --max-requests=0 --percentile=99 --mysql-user=zjy --mysql-password=zjy run

1.

```

上面跑完会出一个结果：

```
#每10秒钟报告一次测试结果，tps、每秒读、每秒写、95%以上的响应时长统计
[  10s] threads: 8, tps: 66.60, reads: 943.67, writes: 269.62, response time: 431.72ms (95%), errors: 0.00, reconnects:  0.00
[  20s] threads: 8, tps: 34.30, reads: 480.20, writes: 137.20, response time: 598.28ms (95%), errors: 0.00, reconnects:  0.00
[  30s] threads: 8, tps: 36.60, reads: 512.40, writes: 146.40, response time: 494.87ms (95%), errors: 0.00, reconnects:  0.00

OLTP test statistics:
    queries performed:
        read:                            941248   #读总数
        write:                           268928   #写总数
        other:                           134464   #其他操作总数(SELECT、INSERT、UPDATE、DELETE之外的操作，例如COMMIT等)
        total:                           1344640  #全部总数
    transactions:                        67232  (112.04 per sec.)  #总事务数(每秒事务数)
    read/write requests:                 1210176 (2016.73 per sec.) #读写总数(每秒读写次数)
    other operations:                    134464 (224.08 per sec.) #其他操作总数(每秒其他操作次数)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          600.0698s   #总耗时
    total number of events:              67232  #共发生多少事务数
    total time taken by event execution: 4799.8569s # 所有事务耗时相加(不考虑并行因素)
    response time:
         min:                                  2.09ms   #最小耗时
         avg:                                 71.39ms  #平均耗时
         max:                                839.32ms #最大耗时
         approx.  95 percentile:             309.40ms 超过95%平均耗时

Threads fairness:
    events (avg/stddev):           8404.0000/17.56
    execution time (avg/stddev):   599.9821/0.02

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

```

**清理数据：cleanup**

```
sysbench --test=/usr/share/doc/sysbench/tests/db/oltp.lua --mysql-table-engine=innodb --mysql-host=127.0.0.1 --mysql-db=dba_test --num-threads=8 --oltp-table-size=500000 --oltp_tables_count=3 --oltp-read-only=off --report-interval=10 --rand-type=uniform --max-time=600 --max-requests=0 --mysql-user=zjy --mysql-password=zjy  cleanup

1.

```

更多 0.5 版本的 sysbench 可以看​ [​sysbench 安装、使用、结果解读​](http://imysql.com/2014/10/17/sysbench-full-user-manual.shtml)​​和​ [​使用 sysbench 对 mysql 压力测试​](http://seanlook.com/2016/03/28/mysql-sysbench/)​。

~~~~~~~~~~~~~~~ 万物之中, 希望至美 ~~~~~~~~~~~~~~~