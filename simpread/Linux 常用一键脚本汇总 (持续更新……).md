> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [aoarasi.com](https://aoarasi.com/archives/onekey)

0. 系统优化脚本 (初始化)
---------------

```
bash -c "$(curl -L s.aaa.al/init.sh)"


```

1. 获取公网地址
---------

2. 多合一脚本
--------

```
wget -O box.sh https://raw.githubusercontent.com/BlueSkyXN/SKY-BOX/main/box.sh && chmod +x box.sh && clear && ./box.sh


```

3.IO 测试 + 测速
------------

```
wget -qO- https://raw.githubusercontent.com/oooldking/script/master/superbench.sh | bash


```

4. 回程路由脚本一
----------

```
wget -qO- git.io/besttrace | bash


```

5. 回程路由脚本二
----------

```
curl https://raw.githubusercontent.com/zhucaidan/mtr_trace/main/mtr_trace.sh|bash


```

6. 流媒体解锁查看
----------

```
bash -c "$(curl -L mcnb.top/netflix.sh)"


```

7.BBR
-----

```
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh 


```