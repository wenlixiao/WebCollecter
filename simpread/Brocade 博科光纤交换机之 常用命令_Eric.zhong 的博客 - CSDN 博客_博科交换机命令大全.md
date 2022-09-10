> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_38623994/article/details/119863352)

### 文章目录

*   *   *   [基础命令](#_2)
        *   *   [IP 地址](#IP_3)
            *   [主机名](#_22)
            *   [Domain ID](#Domain_ID_28)
            *   [DNS 域名](#DNS_41)
            *   [时间日期](#_47)
            *   [版本信息](#_62)
            *   [交换机状态查询](#_73)
        *   [核心命令](#_91)
        *   *   [Alias 别名管理](#Alias_92)
            *   [Zone 分区管理](#Zone_102)
            *   [CFG 配置管理](#CFG_113)
            *   [示例参考](#_126)
        *   [调试命令](#_140)
        *   *   [端口经常出现 up/down 异常调查](#updown_141)
            *   [我的存储压力是如何的？流量带宽调查？](#_152)
            *   [如何查看两台交换机级联？](#_167)

### 基础命令

#### IP 地址

```
ipaddrshow
	Ethernet IP Address: 10.77.77.77
	Ethernet Subnetmask: 255.255.255.0
	Fibre Channel IP Address: none
	Fibre Channel Subnetmask: none
	Gateway Address: 0.0.0.0

ipaddrset
	Ethernet IP Address [10.77.77.77]: 192.168.100.100
	Ethernet Subnetmask [0.0.0.0]: 255.255.255.0
	Fibre Channel IP Address [none]:
	Fibre Channel Subnetmask [none]:
	Gateway Address [172.17.1.1]:192.168.100.254
	Set IP address now? [y = set now, n = next reboot]: y

```

#### [主机名](https://so.csdn.net/so/search?q=%E4%B8%BB%E6%9C%BA%E5%90%8D&spm=1001.2101.3001.7020)

```
switchname

```

#### Domain ID

```
switch:admin> domainsshow
switch:admin> switchdisable 
switch:admin> configure 
	Configure... 
	Fabric Parameters (yes, y, no, n): [no] y 
	Domain (1...239): [1] 2
switch:admin> switchenable

```

> Domain ID 取值范围是 0 ~ 31.

#### DNS 域名

```
dnsconfig

```

#### 时间日期

```
# 时间格式
date "mmddHHMMyy"
# 向导式设置时区
tstimezone --interactive
# 直接式设置时区
tstimezone Asia/Shanghai
# NTP服务设置
tsclockserver "192.168.100.201;192.168.100.202"
tsclockserver
date

```

#### 版本信息

```
version
	Kernel:     2.6.34.6
	Fabric OS:  v8.2.2d
	Made on:    Fri Sep 25 20:57:19 2020
	Flash:      Sat Jul 3 10:55:28 2021
	BootProm:   3.0.32

```

#### 交换机状态查询

```
switchshow - 交换机与端口状态
hashow - CP卡状态（Brodecade 300不支持）
slotshow - SLOT状态（Brodecade 300不支持）
psshow - 电源状态
tempshow - 温度状态
fanshow - 风扇状态
firmwareshow - 微码状态
licenseshow - 授权状态
cfgshow - 配置信息
zoneshow - ZONE信息
uptime - 交换机运行时间
porterrshow - 端口错误信息
errshow - 错误信息

```

### 核心命令

#### Alias 别名管理

```
alicreate - 创建别名
alishow - 显示系统中定义的别名
aliadd - 别名中添加成员
aliremove - 别名中删除成员
alidelete - 删除别名

```

#### Zone 分区管理

```
zonecreate - 创建一个zone
zoneadd - Zone添加新成员
zoneremove - Zone删除成员
zoneobjectcopy - 复制一个zone到一个新的zong
zoneobjectrename - 重新命名一个zone
zoneobjectexpunge - 删除一个zone

```

#### CFG 配置管理

```
cfgshow - 查看zone配置
cfgsave - 保存zone配置
cfgenable - 启用zone配置
cfgclear - 删除所有zone信息
cfgsize - 查看zone数据库大小
cfgtransshow - 查看是否有未被保存的zone相关信息
cfgtransabort - 放弃未被保存的zone相关信息
cfgactvshow - 显示激活中的zone配置

```

#### 示例参考

```
alicreate "ETERNUS-CM1CA0P0", "50:00:00:e0:XX:XX:XX:30"
alicreate "ORACLERAC01-FC01", "10:00:00:10:XX:XX:XX:90"
alicreate "ORACLERAC01-FC02", "10:00:00:10:XX:XX:XX:70 "

zonecreate "ETERNUS-ORACLERAC-ZONE", "ETERNUS-CM0CA0P0; ORACLERAC01-FC01; ORACLERAC01-FC02"

cfgcreate "OracleRAC_cfg","ETERNUS-ORACLERAC-ZONE"
cfgsave
cfgenable OracleRAC_cfg

```

### 调试命令

#### 端口经常出现 up/down 异常调查

```
# 查看端口的状态和RX/TX的衰减值等(下记为端口10参考)
sfpshow 10

# 查看该端口下的报错数量(下记为端口10参考)
porterrshow 10
portstatsclear 10

```

#### 我的存储压力是如何的？流量带宽调查？

```
# 查看该端口可以查询端口带宽
portperfshow
    0      1      2      3      4      5      6      7      8      9     10     11     12     13     14     15  
================================================================================================================
 302.2k   1.7m 244.2k   3.2k   1.7m   0     75.0k 133.5k   0      0      0      0      0      0      0      0   

   16     17     18     19     20     21     22     23   Total
================================================================
   0      0      0      0      0      0      0      0      4.1m


```

#### 如何查看两台交换机级联？

```
# E-Port端口中，upstream是上游设备，downstream是下游设备
admin> switchshow 
# 博科私有协议互联
admin> islshow
  1:  6->  6 10:00:d8:1f:XX:XX:XX:XX   1 FCSw2                  sp: 16.000G bw: 16.000G QOS CR_RECOV FEC 
  2:  7->  7 10:00:d8:1f:XX:XX:XX:XX   1 FCSw2                  sp: 16.000G bw: 16.000G QOS CR_RECOV FEC 

```

> Trunk 可以有效提高交换机之间的数据传输带宽，减少了大规模 SAN 网络发生数据传输拥塞的可能。