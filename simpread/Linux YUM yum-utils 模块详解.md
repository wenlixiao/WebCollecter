> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.shuzhiduo.com](https://www.shuzhiduo.com/A/ke5jBR3O5r/)

> yum-utils 是 yum 的工具包集合，由不同的作者开发，使 yum 使用起来更加方便和强大。

yum-utils 是 yum 的工具包集合，由不同的作者开发，使 yum 使用起来更加方便和强大。包括：debuginfo-install,find-repos-of-install, needs-restarting, package-cleanup, repoclosure, epodiff, repo-graph, repomanage, repoquery, repo-rss, reposync,: repotrack, show-installed, show-changed-rco, verifytree, yumdownloader, yum-builddep,yum-complete-transaction, yum-config-manager, yum-debug-dump, yum-debug-restore and yum-groups-manager.

yum-utils 的安装
-------------

命令：yum install yum-utils -y

```
[root@node1 ~]# yum install yum-utils -y

Loaded plugins: fastestmirror

Loading mirror speeds from cached hostfile

 * base: mirrors.aliyun.com

 * extras: mirrors.aliyun.com

 * updates: mirrors.aliyun.com

Resolving Dependencies

--> Running transaction check

---> Package yum-utils.noarch 0:1.1.31-54.el7_8 will be installed

--> Finished Dependency Resolution
 
Dependencies Resolved
 
================================================================================================================================================================================================================

 Package                                           Arch                                           Version                                                    Repository                                    Size

================================================================================================================================================================================================================

Installing:

 yum-utils                                         noarch                                         1.1.31-54.el7_8                                            base                                         122 k
 
Transaction Summary

================================================================================================================================================================================================================

Install  1 Package
 
Total download size: 122 k

Installed size: 337 k

Downloading packages:

yum-utils-1.1.31-54.el7_8.noarch.rpm                                                                                                                                                     | 122 kB  00:00:00

Running transaction check

Running transaction test

Transaction test succeeded

Running transaction

  Installing : yum-utils-1.1.31-54.el7_8.noarch                                                                                                                                                             1/1

  Verifying  : yum-utils-1.1.31-54.el7_8.noarch                                                                                                                                                             1/1
 
Installed:

  yum-utils.noarch 0:1.1.31-54.el7_8
 
Complete!

```

yum-utils 各个模块的说明
-----------------

### find-repos-of-install 模块

find-repos-of-install 列出包是从哪个 yum 仓库安装的。

```
[root@node1 ~]# find-repos-of-install zlib

Loaded plugins: fastestmirror

zlib-1.2.7-20.el7_9.x86_64 from repo updates   #来着updates仓库

[root@node1 ~]# find-repos-of-install yum-utils

Loaded plugins: fastestmirror

yum-utils-1.1.31-54.el7_8.noarch from repo base  #来着base仓库

```

### needs-restarting　模块

needs-restarting 列出正在允许的进场被更新过，需要重新启动

### package-cleanup 模块

package-cleanup 列出本地安装的，重复的或者 orphan(找不到对应的仓库源) 的包

```
[root@node1 ~]# package-cleanup --problems

Loaded plugins: fastestmirror

No Problems Found

[root@node1 ~]#

[root@node1 ~]# package-cleanup --problems

Loaded plugins: fastestmirror

No Problems Found
#mysql-community仓库源被disable了

[root@node1 ~]# package-cleanup --orphans

Loaded plugins: fastestmirror

Loading mirror speeds from cached hostfile

 * base: mirrors.aliyun.com

 * extras: mirrors.aliyun.com

 * updates: mirrors.aliyun.com

mysql-community-client-8.0.29-1.el7.x86_64

mysql-community-client-plugins-8.0.29-1.el7.x86_64

mysql-community-common-8.0.29-1.el7.x86_64

mysql-community-icu-data-files-8.0.29-1.el7.x86_64

mysql-community-libs-8.0.29-1.el7.x86_64

mysql-community-libs-compat-8.0.29-1.el7.x86_64

mysql-community-server-8.0.29-1.el7.x86_64

mysql80-community-release-el7-6.noarch

#这些包没有被其他的prm用到

[root@node1 ~]#  package-cleanup --leaves --exclude-bin

Loaded plugins: fastestmirror

compat-libcap1-1.10-7.el7.x86_64

compat-libf2c-34-3.4.6-32.el7.x86_64

compat-libgfortran-41-4.1.2-45.el7.x86_64

compat-libtiff3-3.9.4-12.el7.x86_64

libaio-devel-0.3.109-13.el7.x86_64

libpng12-1.2.50-10.el7.x86_64

libsysfs-2.1.0-16.el7.x86_64

zlib-devel-1.2.7-20.el7_9.x86_64

#检查是否有旧kenel的包

[root@node1 ~]#  package-cleanup --oldkernels

Loaded plugins: fastestmirror

No old kernels to remove

```

### repoclosure 模块

repoclosure 模块从多个 yum 仓库读取包的原信息，查询所有的依赖关系，列出无法解决依赖关系的包。

```
[root@node1 ~]# repoclosure

Reading in repository metadata - please wait....

Checking Dependencies

Repos looked at: 4

   base

   extras

   local

   updates

Num Packages in Repos: 18545

```

### repo-graph 模块

repo-graph 模块输出一个详细的包依赖关系的列表：输出很多，最好重定向到文本文件上。

```
[root@node1 ~]# repo-graph --repoid=base |more

"abrt-java-connector" [color="0.682608695652 0.782608695652 1.0"];

"abrt-java-connector" -> {

"abrt"

"glibc"

"abrt-libs"

"satyr"

"libreport"

"glib2"

"systemd-libs"

} [color="0.682608695652 0.782608695652 1.0"];
 
"keyutils-libs-devel" [color="0.526086956522 0.626086956522 1.0"];

"keyutils-libs-devel" -> {

"keyutils-libs"

} [color="0.526086956522 0.626086956522 1.0"];
 
"cracklib-python" [color="0.604347826087 0.704347826087 1.0"];

"cracklib-python" -> {

"python"

"glibc"

"zlib"

"cracklib"

} [color="0.604347826087 0.704347826087 1.0"];
 
}

```

### repomanage 模块

repomanage 在指定目录后，列出最新和最旧的包

```
[root@node1 ~]# repomanage --old ./

[root@node1 ~]# repomanage --new ./

mysql80-community-release-el7-6.noarch.rpm

rsyslog-8.2206.0-1.el8.x86_64.rpm

```

### repoquery 模块

类似于 yum info /list /provides 以及 rpm 的集合。功能强大.

#### 按包名查询

```
[root@node1 ~]# repoquery zlib

zlib-0:1.2.7-20.el7_9.i686

zlib-0:1.2.7-20.el7_9.x86_64
 
#repoquery -i 类似于rpm -qi

[root@node1 ~]# repoquery -i zlib
 
Name        : zlib

Version     : 1.2.7

Release     : 20.el7_9

Architecture: i686

Size        : 184598

Packager    : CentOS BuildSystem <http://bugs.centos.org>

Group       : System Environment/Libraries

URL         : http://www.zlib.net/

Repository  : updates

Summary     : The compression and decompression library

Source      : zlib-1.2.7-20.el7_9.src.rpm

Description :

Zlib is a general-purpose, patent-free, lossless data compression

library which is used by many different programs.
 
Name        : zlib

Version     : 1.2.7

Release     : 20.el7_9

Architecture: x86_64

Size        : 185206

Packager    : CentOS BuildSystem <http://bugs.centos.org>

Group       : System Environment/Libraries

URL         : http://www.zlib.net/

Repository  : updates

Summary     : The compression and decompression library

Source      : zlib-1.2.7-20.el7_9.src.rpm

Description :

Zlib is a general-purpose, patent-free, lossless data compression

library which is used by many different programs.
 
#类似于yum --deplist

[root@node1 ~]# repoquery -R fio

/bin/sh

/usr/bin/bash

/usr/bin/python2.7

libaio.so.1()(64bit)

libaio.so.1(LIBAIO_0.1)(64bit)

libaio.so.1(LIBAIO_0.4)(64bit)

libc.so.6(GLIBC_2.14)(64bit)

libdl.so.2()(64bit)

libdl.so.2(GLIBC_2.2.5)(64bit)

libibverbs.so.1()(64bit)

libibverbs.so.1(IBVERBS_1.0)(64bit)

libibverbs.so.1(IBVERBS_1.1)(64bit)

libm.so.6()(64bit)

libm.so.6(GLIBC_2.15)(64bit)

libm.so.6(GLIBC_2.2.5)(64bit)

libnuma.so.1()(64bit)

libnuma.so.1(libnuma_1.1)(64bit)

libnuma.so.1(libnuma_1.2)(64bit)

libpmem.so.1()(64bit)

libpmem.so.1(LIBPMEM_1.0)(64bit)

libpmemblk.so.1()(64bit)

libpmemblk.so.1(LIBPMEMBLK_1.0)(64bit)

libpthread.so.0()(64bit)

libpthread.so.0(GLIBC_2.2.5)(64bit)

libpthread.so.0(GLIBC_2.3.2)(64bit)

librados.so.2()(64bit)

librbd.so.1()(64bit)

librdmacm.so.1()(64bit)

librdmacm.so.1(RDMACM_1.0)(64bit)

librt.so.1()(64bit)

librt.so.1(GLIBC_2.2.5)(64bit)

libz.so.1()(64bit)

rtld(GNU_HASH)

```

#### 按组名的查询

```
#ga所有的组

[root@node1 ~]# repoquery -ga

additional-devel - Additional Development

anaconda-tools - Anaconda Tools

backup-client - Backup Client

backup-server - Backup Server

base - Base

client-product - CentOS Linux Client product core

computenode-product - CentOS Linux ComputeNode product core

server-product - CentOS Linux Server product core

workstation-product - CentOS Linux Workstation product core

networkmanager-submodules - Common NetworkManager submodules

compat-libraries - Compatibility Libraries

conflicts-client - Conflicts (Client)

conflicts-computenode - Conflicts (ComputeNode)

conflicts-server - Conflicts (Server)

conflicts-workstation - Conflicts (Workstation)

console-internet - Console Internet Tools

core - Core

dns-server - DNS Name Server

debugging - Debugging Tools
 
#指定某个组

[root@node1 ~]# repoquery -g 'Development Tools'

development - Development Tools
 
#查询组的信息

[root@node1 ~]# repoquery -gi 'Development Tools'

Development Tools:
 
A basic development environment.

```

### show-installed 模块

show-installed 显示已经安装的包和介绍 (貌似没什么用)

```
[root@node1 ~]# show-installed

WARNING: The following packages are installed but not in the repository:

        mysql-community-common

        mysql-community-libs

        mysql-community-client

        mysql-community-server

        mysql-community-client-plugins

        mysql80-community-release

        mysql-community-icu-data-files

        mysql-community-libs-compat
 
@compat-libraries

@core

@debugging

@development

# Others

authconfig

chrony

esc

grub2

httpd

hunspell-en

kernel

libaio-devel

lsof

mysql-community-server

mysql80-community-release

open-vm-tools

pcre-devel

scap-security-guide

wget

yum-utils

zlib-devel

# 560 package names, 101 leaves

# 4 groups, 17 leftovers, 0 excludes

# 25 lines

```

### reposync 模块

reposync 将 yum 仓库同步到本地目录。后续可以自己做 yum 仓库

```
#Sync all packages from the 'updates' repo to the repos directory:

reposync -p repos --repoid=updates
 
# Sync all packages from the 'updates' repo to the repos directory excluding x86_64 arch. Edit /etc/yum.conf adding option exclude=*.x86_64. Then:

reposync -p repos --repoid=updates

```

### repotrack 模块

repotrack 调查一个包，和他的依赖关系，并下载下来，可以作为 reposync 的补充使用。

```
[root@node1 ~]# repotrack -p fiopkgs fio

Downloading acl-2.2.51-15.el7.x86_64.rpm

Downloading audit-libs-2.8.5-4.el7.x86_64.rpm

Downloading audit-libs-2.8.5-4.el7.i686.rpm

Downloading basesystem-10.0-7.el7.centos.noarch.rpm

Downloading bash-4.2.46-35.el7_9.x86_64.rpm

Downloading bc-1.06.95-13.el7.x86_64.rpm

Downloading binutils-2.27-44.base.el7_9.1.x86_64.rpm

Downloading boost-iostreams-1.53.0-28.el7.x86_64.rpm

Downloading boost-iostreams-1.53.0-28.el7.i686.rpm

Downloading boost-random-1.53.0-28.el7.x86_64.rpm

Downloading boost-random-1.53.0-28.el7.i686.rpm

Downloading boost-system-1.53.0-28.el7.x86_64.rpm

Downloading boost-system-1.53.0-28.el7.i686.rpm

Downloading boost-thread-1.53.0-28.el7.i686.rpm

Downloading boost-thread-1.53.0-28.el7.x86_64.rpm

Downloading bzip2-libs-1.0.6-13.el7.x86_64.rpm

Downloading bzip2-libs-1.0.6-13.el7.i686.rpm

Downloading ca-certificates-2021.2.50-72.el7_9.noarch.rpm

Downloading centos-release-7-9.2009.1.el7.centos.x86_64.rpm

Downloading chkconfig-1.7.6-1.el7.x86_64.rpm

Downloading coreutils-8.22-24.el7_9.2.x86_64.rpm

Downloading cpio-2.11-28.el7.x86_64.rpm

Downloading cracklib-2.9.0-11.el7.x86_64.rpm

Downloading cracklib-2.9.0-11.el7.i686.rpm

Downloading cracklib-dicts-2.9.0-11.el7.x86_64.rpm

Downloading cryptsetup-libs-2.0.3-6.el7.x86_64.rpm

```

### verifytree 模块

verifytree 检查本地仓库是否一致。

```
[root@node1 ~]# verifytree /etc/yum.repos.d/local.repo

Loaded plugins: fastestmirror

Determining fastest mirrors

Checking repodata:

  failed to load repomd.xml.

```

### yumdownloader 模块

yumdownloader 下载 package 包到本地。同时可以下载依赖包

```
[root@node1 ~]# yumdownloader --destdir fiopkgs --resolve fio

Loaded plugins: fastestmirror

Loading mirror speeds from cached hostfile

 * base: mirrors.aliyun.com

 * extras: mirrors.aliyun.com

 * updates: mirrors.aliyun.com

--> Running transaction check

---> Package fio.x86_64 0:3.7-2.el7 will be installed

--> Processing Dependency: librdmacm.so.1(RDMACM_1.0)(64bit) for package: fio-3.7-2.el7.x86_64

--> Processing Dependency: libpmemblk.so.1(LIBPMEMBLK_1.0)(64bit) for package: fio-3.7-2.el7.x86_64

--> Processing Dependency: libpmem.so.1(LIBPMEM_1.0)(64bit) for package: fio-3.7-2.el7.x86_64

--> Processing Dependency: libibverbs.so.1(IBVERBS_1.1)(64bit) for package: fio-3.7-2.el7.x86_64

--> Processing Dependency: libibverbs.so.1(IBVERBS_1.0)(64bit) for package: fio-3.7-2.el7.x86_64

--> Processing Dependency: librdmacm.so.1()(64bit) for package: fio-3.7-2.el7.x86_64

--> Processing Dependency: librbd.so.1()(64bit) for package: fio-3.7-2.el7.x86_64

--> Processing Dependency: librados.so.2()(64bit) for package: fio-3.7-2.el7.x86_64

--> Processing Dependency: libpmemblk.so.1()(64bit) for package: fio-3.7-2.el7.x86_64

--> Processing Dependency: libpmem.so.1()(64bit) for package: fio-3.7-2.el7.x86_64

--> Processing Dependency: libibverbs.so.1()(64bit) for package: fio-3.7-2.el7.x86_64

--> Running transaction check

---> Package libibverbs.x86_64 0:22.4-6.el7_9 will be installed

--> Processing Dependency: rdma-core(x86-64) = 22.4-6.el7_9 for package: libibverbs-22.4-6.el7_9.x86_64

---> Package libpmem.x86_64 0:1.5.1-2.1.el7 will be installed

---> Package libpmemblk.x86_64 0:1.5.1-2.1.el7 will be installed

--> Processing Dependency: libndctl.so.6(LIBNDCTL_3)(64bit) for package: libpmemblk-1.5.1-2.1.el7.x86_64

--> Processing Dependency: libndctl.so.6(LIBNDCTL_14)(64bit) for package: libpmemblk-1.5.1-2.1.el7.x86_64

--> Processing Dependency: libndctl.so.6(LIBNDCTL_1)(64bit) for package: libpmemblk-1.5.1-2.1.el7.x86_64

--> Processing Dependency: libdaxctl.so.1(LIBDAXCTL_2)(64bit) for package: libpmemblk-1.5.1-2.1.el7.x86_64

--> Processing Dependency: libndctl.so.6()(64bit) for package: libpmemblk-1.5.1-2.1.el7.x86_64

--> Processing Dependency: libdaxctl.so.1()(64bit) for package: libpmemblk-1.5.1-2.1.el7.x86_64

---> Package librados2.x86_64 1:10.2.5-4.el7 will be installed

--> Processing Dependency: libboost_random-mt.so.1.53.0()(64bit) for package: 1:librados2-10.2.5-4.el7.x86_64

--> Processing Dependency: libboost_iostreams-mt.so.1.53.0()(64bit) for package: 1:librados2-10.2.5-4.el7.x86_64

---> Package librbd1.x86_64 1:10.2.5-4.el7 will be installed

---> Package librdmacm.x86_64 0:22.4-6.el7_9 will be installed

--> Running transaction check

---> Package boost-iostreams.x86_64 0:1.53.0-28.el7 will be installed

---> Package boost-random.x86_64 0:1.53.0-28.el7 will be installed

---> Package daxctl-libs.x86_64 0:65-5.el7 will be installed

---> Package ndctl-libs.x86_64 0:65-5.el7 will be installed

---> Package rdma-core.x86_64 0:22.4-6.el7_9 will be installed

--> Finished Dependency Resolution

(1/12): boost-iostreams-1.53.0-28.el7.x86_64.rpm                                                                                                                                         |  61 kB  00:00:00

(2/12): daxctl-libs-65-5.el7.x86_64.rpm                                                                                                                                                  |  27 kB  00:00:00

(3/12): boost-random-1.53.0-28.el7.x86_64.rpm                                                                                                                                            |  39 kB  00:00:00

(4/12): libpmem-1.5.1-2.1.el7.x86_64.rpm                                                                                                                                                 |  59 kB  00:00:00

(5/12): libpmemblk-1.5.1-2.1.el7.x86_64.rpm                                                                                                                                              |  80 kB  00:00:00

(6/12): fio-3.7-2.el7.x86_64.rpm                                                                                                                                                         | 467 kB  00:00:01

(7/12): libibverbs-22.4-6.el7_9.x86_64.rpm                                                                                                                                               | 269 kB  00:00:01

(8/12): librdmacm-22.4-6.el7_9.x86_64.rpm                                                                                                                                                |  64 kB  00:00:00

(9/12): librados2-10.2.5-4.el7.x86_64.rpm                                                                                                                                                | 1.8 MB  00:00:04

(10/12): ndctl-libs-65-5.el7.x86_64.rpm                                                                                                                                                  |  65 kB  00:00:00

(11/12): rdma-core-22.4-6.el7_9.x86_64.rpm                                                                                                                                               |  51 kB  00:00:00

(12/12): librbd1-10.2.5-4.el7.x86_64.rpm

```

yum-builddep 模块

yum-builddep 模块会安装待安装的包的缺失的依赖包。

```
       Download and install all the RPMs needed to build the kernel RPM:

              yumdownloader --source kernel && rpm2cpio kernel*src.rpm | cpio -i kernel.spec && \

              yum-builddep kernel.spec

```

yum-complete-transaction 模块

yum-complete-transaction 尝试完成 failed 或者中断的 yum transaction.

```
[root@node1 fiopkgs]# yum-complete-transaction

Loaded plugins: fastestmirror

Loading mirror speeds from cached hostfile

 * base: mirrors.aliyun.com

 * extras: mirrors.aliyun.com

 * updates: mirrors.aliyun.com

No unfinished transactions left.

[root@node1 fiopkgs]# yum-complete-transaction --cleanup-only

Loaded plugins: fastestmirror

Loading mirror speeds from cached hostfile

 * base: mirrors.aliyun.com

 * extras: mirrors.aliyun.com

 * updates: mirrors.aliyun.com

No unfinished transactions left.

[root@node1 fiopkgs]#

```

### yum-config-manager

yum-config-manager 用来管理 yum 主要配置选项，同时控制仓库源的开启或关闭，也可以添加新的仓库源。

```
#查看local源的配置

[root@node1 fiopkgs]# yum-config-manager local

Loaded plugins: fastestmirror

================================================================================================= repo: local ==================================================================================================

[local]

async = True

bandwidth = 0

base_persistdir = /var/lib/yum/repos/x86_64/7

baseurl = file:///mnt/cdrom

cache = 0

cachedir = /var/cache/yum/x86_64/7/local

check_config_file_age = True

compare_providers_priority = 80

cost = 1000

deltarpm_metadata_percentage = 100

deltarpm_percentage =

enabled = True

enablegroups = True

exclude =

failovermethod = priority

ftp_disable_epsv = False

gpgcadir = /var/lib/yum/repos/x86_64/7/local/gpgcadir

gpgcakey =

gpgcheck = False

gpgdir = /var/lib/yum/repos/x86_64/7/local/gpgdir

gpgkey =

hdrdir = /var/cache/yum/x86_64/7/local/headers

http_caching = all

includepkgs =

ip_resolve =

keepalive = True

keepcache = False

mddownloadpolicy = sqlite

mdpolicy = group:small

mediaid =

metadata_expire = 21600

metadata_expire_filter = read-only:present

metalink =

minrate = 0

mirrorlist =

mirrorlist_expire = 86400

name = local

old_base_cache_dir =

password =

persistdir = /var/lib/yum/repos/x86_64/7/local

pkgdir = /var/cache/yum/x86_64/7/local/packages

proxy = False

proxy_dict =

proxy_password =

proxy_username =

repo_gpgcheck = False

retries = 10

skip_if_unavailable = False

ssl_check_cert_permissions = True

sslcacert =

sslclientcert =

sslclientkey =

sslverify = True

throttle = 0

timeout = 30.0

ui_id = local

ui_repoid_vars = releasever,

   basearch

username =
 
[root@node1 fiopkgs]# yum-config-manager --enable local

Loaded plugins: fastestmirror

================================================================================================= repo: local ==================================================================================================

[local]

async = True

bandwidth = 0

base_persistdir = /var/lib/yum/repos/x86_64/7

baseurl = file:///mnt/cdrom

cache = 0

cachedir = /var/cache/yum/x86_64/7/local

check_config_file_age = True

compare_providers_priority = 80

cost = 1000

deltarpm_metadata_percentage = 100

deltarpm_percentage =

enabled = True

enablegroups = True

exclude =

failovermethod = priority

ftp_disable_epsv = False

gpgcadir = /var/lib/yum/repos/x86_64/7/local/gpgcadir

gpgcakey =

gpgcheck = False

gpgdir = /var/lib/yum/repos/x86_64/7/local/gpgdir

gpgkey =

hdrdir = /var/cache/yum/x86_64/7/local/headers

http_caching = all

includepkgs =

ip_resolve =

keepalive = True

keepcache = False

mddownloadpolicy = sqlite

mdpolicy = group:small

mediaid =

metadata_expire = 21600

metadata_expire_filter = read-only:present

metalink =

minrate = 0

mirrorlist =

mirrorlist_expire = 86400

name = local

old_base_cache_dir =

password =

persistdir = /var/lib/yum/repos/x86_64/7/local

pkgdir = /var/cache/yum/x86_64/7/local/packages

proxy = False

proxy_dict =

proxy_password =

proxy_username =

repo_gpgcheck = False

retries = 10

skip_if_unavailable = False

ssl_check_cert_permissions = True

sslcacert =

sslclientcert =

sslclientkey =

sslverify = True

throttle = 0

timeout = 30.0

ui_id = local

ui_repoid_vars = releasever,

   basearch

username =
 
[root@node1 fiopkgs]# yum-config-manager --disable local

Loaded plugins: fastestmirror

================================================================================================= repo: local ==================================================================================================

[local]

async = True

bandwidth = 0

base_persistdir = /var/lib/yum/repos/x86_64/7

baseurl = file:///mnt/cdrom

cache = 0

cachedir = /var/cache/yum/x86_64/7/local

check_config_file_age = True

compare_providers_priority = 80

cost = 1000

deltarpm_metadata_percentage = 100

deltarpm_percentage =

enabled = 0

enablegroups = True

exclude =

failovermethod = priority

ftp_disable_epsv = False

gpgcadir = /var/lib/yum/repos/x86_64/7/local/gpgcadir

gpgcakey =

gpgcheck = False

gpgdir = /var/lib/yum/repos/x86_64/7/local/gpgdir

gpgkey =

hdrdir = /var/cache/yum/x86_64/7/local/headers

http_caching = all

includepkgs =

ip_resolve =

keepalive = True

keepcache = False

mddownloadpolicy = sqlite

mdpolicy = group:small

mediaid =

metadata_expire = 21600

metadata_expire_filter = read-only:present

metalink =

minrate = 0

mirrorlist =

mirrorlist_expire = 86400

name = local

old_base_cache_dir =

password =

persistdir = /var/lib/yum/repos/x86_64/7/local

pkgdir = /var/cache/yum/x86_64/7/local/packages

proxy = False

proxy_dict =

proxy_password =

proxy_username =

repo_gpgcheck = False

retries = 10

skip_if_unavailable = False

ssl_check_cert_permissions = True

sslcacert =

sslclientcert =

sslclientkey =

sslverify = True

throttle = 0

timeout = 30.0

ui_id = local

ui_repoid_vars = releasever,

   basearch

username =
 
[root@node1 fiopkgs]# yum-config-manager --setopt=clean_requirements_on_remove=0

Loaded plugins: fastestmirror

===================================================================================================== main =====================================================================================================

[main]

alwaysprompt = True

assumeno = False

assumeyes = False

autocheck_running_kernel = True

autosavets = True

bandwidth = 0

bugtracker_url = http://bugs.centos.org/set_project.php?project_id=23&ref=http://bugs.centos.org/bug_report_page.php?category=yum

cache = 0

cachedir = /var/cache/yum/x86_64/7

check_config_file_age = True

clean_requirements_on_remove = False

color = auto

color_list_available_downgrade = dim,cyan

color_list_available_install = normal

color_list_available_reinstall = bold,underline,green

color_list_available_running_kernel = bold,underline

color_list_available_upgrade = bold,blue

color_list_installed_extra = bold,red

color_list_installed_newer = bold,yellow

color_list_installed_older = bold

color_list_installed_reinstall = normal

color_list_installed_running_kernel = bold,underline

color_search_match = bold

color_update_installed = normal

color_update_local = bold

color_update_remote = normal

commands =

debuglevel = 2

deltarpm = 2

deltarpm_metadata_percentage = 100

deltarpm_percentage = 75

depsolve_loop_limit = 100

disable_includes =

diskspacecheck = True

distroverpkg = centos-release

downloaddir =

downloadonly =

enable_group_conditionals = True

enabled = True

enablegroups = True

errorlevel = 2

exactarch = True

exactarchlist =

exclude =

exit_on_lock = False

failovermethod = priority

fssnap_abort_on_errors = any

fssnap_automatic_keep = 1

fssnap_automatic_post = False

fssnap_automatic_pre = False

fssnap_devices = !*/swap,

   !*/lv_swap

fssnap_percentage = 100

ftp_disable_epsv = False

gaftonmode = False

gpgcheck = True

group_command = objects

group_package_types = mandatory,

   default

groupremove_leaf_only = False

history_list_view = single-user-commands

history_record = True

history_record_packages = yum,

   rpm

http_caching = all

installonly_limit = 5

installonlypkgs = kernel,

   kernel-bigmem,

   installonlypkg(kernel),

   installonlypkg(kernel-module),

   installonlypkg(vm),

   kernel-enterprise,

   kernel-smp,

   kernel-debug,

   kernel-unsupported,

   kernel-source,

   kernel-devel,

   kernel-PAE,

   kernel-PAE-debug

installroot = /

ip_resolve =

keepalive = True

keepcache = False

kernelpkgnames = kernel,

   kernel-smp,

   kernel-enterprise,

   kernel-bigmem,

   kernel-BOOT,

   kernel-PAE,

   kernel-PAE-debug

loadts_ignoremissing = False

loadts_ignorenewrpm = False

loadts_ignorerpm = False

localpkg_gpgcheck = False

logfile = /var/log/yum.log

max_connections = 0

mddownloadpolicy = sqlite

mdpolicy = group:small

metadata_expire = 21600

metadata_expire_filter = read-only:present

minrate = 0

mirrorlist_expire = 86400

multilib_policy = best

obsoletes = True

override_install_langs =

overwrite_groups = False

password =

payload_gpgcheck = False

persistdir = /var/lib/yum

pluginconfpath = /etc/yum/pluginconf.d

pluginpath = /usr/share/yum-plugins,

   /usr/lib/yum-plugins

plugins = True

progess_obj =

protected_multilib = True

protected_packages = yum,

   systemd

proxy = False

proxy_password =

proxy_username =

query_install_excludes = False

recent = 7

recheck_installed_requires = True

remove_leaf_only = False

repo_gpgcheck = False

repopkgsremove_leaf_only = False

reposdir = /etc/yum/repos.d,

   /etc/yum.repos.d

reset_nice = True

retries = 10

rpm_check_debug = True

rpmverbosity = info

shell_exit_status = 0

showdupesfromrepos = False

skip_broken = False

skip_missing_names_on_install = True

skip_missing_names_on_update = True

ssl_check_cert_permissions = True

sslcacert =

sslclientcert =

sslclientkey =

sslverify = True

syslog_device = /dev/log

syslog_facility = LOG_USER

syslog_ident =

throttle = 0

timeout = 30.0

tolerant = True

tsflags =

ui_repoid_vars = releasever,

   basearch

upgrade_group_objects_upgrade = True

upgrade_requirements_on_install = False

usercache = True

username =

usr_w_check = True

```

### yum-debug-dump/yum-debug-restore 模块

yum-debug-dump 模块是用来导出当前安装的和仓库源里可安装的 rpm 包信息。dump 会输出一个文件在当前目录：yum_debug_dump-<hostname>-<time>.txt.gz。可以用 zless 命令命令查看

yun-debug-restore 导入 dump 模块输出的 txt.gz 的包到本地信息库

```
[root@node1 ~]# yum-debug-dump

Loaded plugins: fastestmirror

Loading mirror speeds from cached hostfile

 * base: mirrors.aliyun.com

 * extras: mirrors.aliyun.com

 * updates: mirrors.aliyun.com

Output written to: /root/yum_debug_dump-node1-2022-06-29_02:47:16.txt.gz
 
[root@node1 ~]# yum-debug-restore  /root/yum_debug_dump-node1-2022-06-29_02:47:16.txt.gz

Loaded plugins: fastestmirror

Reading from: /root/yum_debug_dump-node1-2022-06-29_02:47:16.txt.gz

Loaded plugins: fastestmirror

Leaving Shell

[root@node1 ~]#

```

yum-groups-manager

```
EXAMPLES

       Create a new group metadata file, with a group called yum containing all the packages that start with yum:

               yum-groups-manager --name YUM --save groups.xml 'yum*'
 
       After the above command, load the groups.xml data, work with the yum group, make the group not user visible, and remove the yum-skip-broken and yum-priorities packages from it:

               yum-groups-manager -n YUM --merge groups.xml --remove yum-skip-broken yum-priorities --not-user-visible
 
       After the above commands, add a description and a translated name to the yum group:

               yum-groups-manager -n YUM --merge groups.xml --description 'This is a group with most of the yum packages in it' --translated-name 'en:yum packages'

```

[Linux YUM yum-utils 模块详解的更多相关文章](https://www.shuzhiduo.com/R/ke5jBR3O5r/)
--------------------------------------------------------------------------

1.  [yum 和 epel 的详解](https://www.shuzhiduo.com/A/gVdnMqQp5W/)
    
    一. 概览 1. 什么是 repo 文件 repo 文件是 Fedora 中 yum 源 (软件仓库) 的配置文件, 通常一个 repo 文件定义了一个或者多个软件仓库的细节内容, 例如我们将从哪里下载需要安装或者升级的软件包, r ...
    
2.  [Kali linux 2016.2（Rolling）中的 payloads 模块详解](https://www.shuzhiduo.com/A/A2dmkGAOJe/)
    
    不多说, 直接上干货! 前期博客 Kali linux 2016.2(Rolling) 中的 Exploits 模块详解 payloads 模块, 也就是 shellcode, 就是在漏洞利用成功后所要做的事情. 在 M ...
    
3.  [yum 的 repo 文件详解、以及 epel 简介、yum 源的更换、常用 yum 命令](https://www.shuzhiduo.com/A/Gkz1w8ZZ5R/)
    
    https://www.cnblogs.com/nineep/p/6795692.html       yum 的 repo 文件详解. 以及 epel 简介. yum 源的更换 常用命令如下: yum list  ...
    
4.  [Ansible 安装部署及常用模块详解](https://www.shuzhiduo.com/A/1O5EvKDyd7/)
    
    Ansible 命令使用 Ansible 语法使用 ansible <pattern_goes_here> -m <module_name> -a <arguments> ...
    
5.  [ansible 中常用模块详解](https://www.shuzhiduo.com/A/mo5kZbOMJw/)
    
    ansible 中常用的模块详解: file 模块 ansible 内置的可以查看模块用法的命令如下: [root@docker5 ~]# ansible-doc -s file - name: Sets ...
    
6.  [Linux 计划任务 Crontab 实例详解教程](https://www.shuzhiduo.com/A/MAzA8278d9/)
    
    说明: Crontab 是 Linux 系统中在固定时间执行某一个程序的工具, 类似于 Windows 系统中的任务计划程序 下面通过详细实例来说明在 Linux 系统中如何使用 Crontab 操作系统: CentOS ...
    
7.  [Linux 文件权限与属性详解 之 ACL](https://www.shuzhiduo.com/A/A2dm92V4de/)
    
    Linux 文件权限与属性详解 之 一般权限 Linux 文件权限与属性详解 之 ACL Linux 文件权限与属性详解 之 SUID.SGID & SBIT Linux 文件权限与属性详解 之 ch ...
    
8.  [Linux 下 DNS 服务器搭建详解](https://www.shuzhiduo.com/A/n2d9x8ro5D/)
    
    Linux 下 DNS 服务器搭建详解 DNS  即 Domain Name System(域名系统) 的缩写, 它是一种将 ip 地址转换成对应的主机名或将主机名转换成与之相对应 ip 地址的一种机制. 其中通过域名解析 ...
    
9.  [【转】postgresql 9.4 在 linux 环境的安装步骤详解](https://www.shuzhiduo.com/A/mo5kxM1Edw/)
    
    本文章来为各位介绍一篇关于 postgresql 9.4 在 linux 环境的安装步骤详解, 希望文章能够对各位新手朋友带来帮助的哦. 环境说明系统: centos 6.4 64 位软件: postgresql ...
    
10.  [python 之 OS 模块详解](https://www.shuzhiduo.com/A/QW5Ypy3Jma/)
    
    python 之 OS 模块详解 ^_^, 步入第二个模块世界 ----->OS 常见函数列表 os.sep: 取代操作系统特定的路径分隔符 os.name: 指示你正在使用的工作平台. 比如对于 Windows ...
    

随机推荐
----

1.  [切分 vocab 时遇到的问题](https://www.shuzhiduo.com/A/mo5knvM3Jw/)
    
    vocab 的格式如下所示, 每个词和对应 100 维的向量: </s> 0.004003 0.004419 -0.003830 -0.003278 0.001367 0.003021 0.000 ...
    
2.  [显示 mac 隐藏文件](https://www.shuzhiduo.com/A/x9J2ODEe56/)
    
    显示 Mac 隐藏文件的命令: defaults write com.apple.finder AppleShowAllFiles -bool true 隐藏 Mac 隐藏文件的命令: defaults writ ...
    
3.  [HDU 1061 N^N (n 的 n 次方的最后一位)](https://www.shuzhiduo.com/A/A2dmExnqde/)
    
    题目意思: http://acm.hdu.edu.cn/showproblem.php?pid=1061 求 N^N 的最后一位数. 题目分析: 此题有非常多种方法, 主要是中循环节, 看自己怎么找了. 我的方 ...
    
4.  [WebKit 介绍和总结（一）](https://www.shuzhiduo.com/A/pRdBWZRnzn/)
    
    一 . WebKit 简单介绍 Webkit 是一个开放源码的浏览器引擎 (web browser engine) , 最初的代码来自 KDE 的 KHTML 和 KJS( 均开放源码 ) . 苹果公司 ...
    
5.  [java 常用 API 的总结 (1)](https://www.shuzhiduo.com/A/xl56bMlx5r/)
    
    本篇是对于这一段时间以来接触到的常用 api 的一些总结, 便于以后的查阅.... 一. 正则表达式 对于正则表达式, 我的感觉就是当我们在做某些题的时候正则表达式会省去我们很多的时间, 并且正则表达式的使用格式 ...
    
6.  [excel 打开 csv 格式的文件，数字末尾都变成零，解决方式](https://www.shuzhiduo.com/A/obzb2V3MzE/)
    
    excel 打开 csv 格式的文件, 数字末尾都变成零, 解决方式
    
7.  [Spring Extensible XML](https://www.shuzhiduo.com/A/obzb7O2bJE/)
    
    Spring 框架从 2.0 版本开始, 提供了基于 Schema 风格的 Spring XML 格式用来定义 bean 的扩展机制. 引入 Schema-based XML 是为了对 Traditional 的 XML 配置形式进行 ...
    
8.  [Pythagorean Triples 707C](https://www.shuzhiduo.com/A/Gkz1p2gndR/)
    
    Katya studies in a fifth grade. Recently her class studied right triangles and the Pythagorean theor ...
    
9.  [css 字符串转换为类 map 对象及反转](https://www.shuzhiduo.com/A/1O5EYOO357/)
    
    存储对象为啥是类 map(即:{key:val,...} 格式), 因为 Map 对象的 val 为字符时, 无法存储 '('.')' 左右括号, 我也很无奈╮(╯▽╰)╭ 解析脚本: <!DOCTYPE htm ...
    
10.  [python 编译模块为 2 禁制](https://www.shuzhiduo.com/A/mo5ky9L2zw/)
    
    编译模块为 2 禁制 yum -y install python26-setuptoolseasy_install -U setuptools# cd /usr/lib64/python2.6# easy_ ...