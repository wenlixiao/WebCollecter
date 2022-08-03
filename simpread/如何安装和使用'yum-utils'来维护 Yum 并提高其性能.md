> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.howtoing.com](https://www.howtoing.com/linux-yum-package-management-with-yum-utils/)

> 在本指南中，我们将向您介绍 yum-utils，这是一个与 yum 集成的实用程序集合，通过多种方式扩展其本机特性，

<div style="float: left;padding-right: 20px;padding-top: 5px;"> <script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script> <ins class="adsbygoogle" style="display:inline-block;width:336px;height:280px" data-ad-client="ca-pub-7163569408396951" data-ad-slot="7352319254"></ins> <script> //$(document).ready(function(){(adsbygoogle = window.adsbygoogle || []).push({})}); </script> </div>

不管 Fedora 的开始采用 [DNF 作为新的包管理器](https://www.howtoing.com/dnf-commands-for-fedora-rpm-package-management/)和默认的包管理库，它不会取代旧的好的 [yum 包管理](https://www.howtoing.com/20-linux-yum-yellowdog-updater-modified-commands-for-package-mangement/)的其他副产品的发行版（如 **Red Hat 企业版 **Linux（RHEL）****和 **CentOS）**好，直到它已被证明是可靠的，Yum 和更坚实的（根据 **Fedora 项目的 wiki，**为 2015 年 11 月 15 日的**，DNF** 仍处于测试的状态）。 因此，你**的 yum 管理**技能将竭诚为您服务以及为仍相当一段时间。

[![](https://www.howtoing.com/wp-content/uploads/2015/12/Yum-Management-using-Yum-Utils.png)](https://www.howtoing.com/wp-content/uploads/2015/12/Yum-Management-using-Yum-Utils.png)

使用'yum-utils'来维护 YUM 并提高其性能

出于这个原因，在本指南中，我们将向您介绍 **Yum-utils 的** ，那与 Yum 整合以多种方式扩大其机功能实用程序的集合，从而使其功能更强大，使用更方便。

### 在 RHEL / CentOS 中安装 yum-utils

**Yum-utils 的**是包含在基本回购（这是默认启用）在任何基于 Fedora 的分布，使安装它是因为这样做很容易：

```
# yum update && yum install yum-utils

```

所有由 **Yum-utils 的**提供的实用程序与主包，我们将在下一节中自动安装。

### 探索实用程序由 yum-utils 包提供

**Yum-utils 的**提供的工具列在其手册页：

```
# man yum-utils

```

以下是我们认为你有兴趣在这些 **Yum** 公用事业 10：

#### 1. 调试软件包

**debuginfo 软装 <软件包名称>** 要求安装调试的 **debuginfo 软**包（和它们的依赖） `<package name>`在崩溃的情况下，或者在开发使用某个软件包的应用程序。

为了调试一个包（或任何其他可执行程序），我们还需要安装 [GDB（GNU 调试器）](https://www.howtoing.com/debug-source-code-in-linux-using-gdb/) ，并用它在调试模式下启动程序。

例如：

```
# gdb $(which postfix)

```

上面的命令将启动一个 **gdb 的外壳** ，我们可以输入操作来执行。 例如， **运行** （如下面的图像中）将启动该程序，而 **BT（**未示出）将显示栈跟踪（也称为**回溯** ）的程序，这将提供的函数调用导致一个列表程序执行的某一点（使用这些信息，开发人员和系统管理员都可以知道在崩溃的情况下出了什么问题）。

其他可用的动作及其预期结果列于**男人 GDB。**

[![](https://www.howtoing.com/wp-content/uploads/2015/12/Debug-a-Package-in-Linux.png)](https://www.howtoing.com/wp-content/uploads/2015/12/Debug-a-Package-in-Linux.png)

在 Linux 中调试软件包

#### 2. 查找已安装软件包的存储库

下面的命令显示该库中已安装的软件包`<package 1>` `<package 2>` ... `<package n>`是从安装：

```
# find-repos-of-install httpd postfix dovecot

```

[![](https://www.howtoing.com/wp-content/uploads/2015/12/Find-Repository-of-Installed-Packages.png)](https://www.howtoing.com/wp-content/uploads/2015/12/Find-Repository-of-Installed-Packages.png)

在 Linux 中查找已安装软件包的存储库

如果不带参数运行， **发现 - 回购 - 的安装**将返回当前已安装的软件包的完整列表。

#### 3. 删除重复或孤立的包

**包清理**管理（从比当前配置的存储库之外的来源安装的程序）包清理，重复，孤儿软件包和其他依赖不一致，包括删除，如下例所示老的内核：

```
# package-cleanup --orphans
# package-cleanup --oldkernels

```

[![](https://www.howtoing.com/wp-content/uploads/2015/12/Remove-Duplicate-or-Orphaned-Packages.png)](https://www.howtoing.com/wp-content/uploads/2015/12/Remove-Duplicate-or-Orphaned-Packages.png)

在 Linux 中删除重复或孤立的包

你不必担心最后一个命令损坏你的内核。 它只会影响不再需要的旧内核包（当前运行的版本之前的版本）。

#### 4. 找出包依赖列表

**回购图**返回在所有可从配置的仓库中的包**点**格式全包的依赖列表。 另外， `repo-graph`可如果与使用存储库返回相同的信息`--repoid=<repo>`选项。

例如，让我们查看更新存储库中每个软件包的依赖关系：

```
# repo-graph --repoid=updates | less

```

[![](https://www.howtoing.com/wp-content/uploads/2015/12/Find-Out-Package-Dependency-List.png)](https://www.howtoing.com/wp-content/uploads/2015/12/Find-Out-Package-Dependency-List.png)

查找软件包依赖性列表

在上面的命令，我们发送**回购图**的输出少，方便的可视化，但你可以或者重定向到一个本地文件供以后检查：

```
# repo-graph --repoid=updates > updates-dependencies.txt

```

在这两种情况下，我们可以看到**，iputils** 软件包依赖于 **systemd** 和 **OpenSSL - 库** 。

#### 5. 检查未解决的依赖关系的列表

**repoclosure** 读取配置的存储库的元数据，检查列入其中，并为每个包未解决的依赖性的显示列表包的依赖关系：

```
# repoclosure

```

[![](https://www.howtoing.com/wp-content/uploads/2015/12/Display-List-of-Unresolved-Dependencies.png)](https://www.howtoing.com/wp-content/uploads/2015/12/Display-List-of-Unresolved-Dependencies.png)

显示未解析的依赖关系的列表

#### 6. 如何检查目录中的最新或最旧的软件包

**repomanage** 查询用 rpm 包的目录，并在目录中返回最新或最早的软件包列表。 这个工具可以派上用场，如果您有您储存不同的程序的几个**.rpm 的**包目录。

当不带参数执行**，repomanage** 返回最新的软件包。 如果与运行`--old`标志，它将返回最早的包：

```
# ls -l
# cd rpms
# ls -l rpms
# repomanage rpms

```

[![](https://www.howtoing.com/wp-content/uploads/2015/12/Check-Newest-Oldest-RPM-Pacakges-in-Directory.png)](https://www.howtoing.com/wp-content/uploads/2015/12/Check-Newest-Oldest-RPM-Pacakges-in-Directory.png)

检查目录中最新的最旧 RPM 软件包

请注意，改变 rpm 包的名称将不会影响如何 **repomanage** 的作品。

#### 7. 查询 Yum 存储库以获取有关软件包的信息

**repoquery** 查询 Yum 库，并得到有关包的其他信息，无论是安装或没有（相关性，包含的文件包中，更多）。

例如， [HTOP（Linux 的过程监控）](https://www.howtoing.com/install-htop-linux-process-monitoring-for-rhel-centos-fedora/)当前未安装此系统上，你可以看到如下：

```
# which htop
# rpm -qa | grep htop

```

[![](https://www.howtoing.com/wp-content/uploads/2015/12/Query-RPM-Package.png)](https://www.howtoing.com/wp-content/uploads/2015/12/Query-RPM-Package.png)

查询 RPM 包

现在假设我们要列出 **HTOP** 的相关性，与包含在默认安装的文件一起。 为此，请分别执行以下两个命令：

```
# repoquery --requires htop
# repoquery --list htop

```

[![](https://www.howtoing.com/wp-content/uploads/2015/12/List-Dependencies-of-RPM-Package.png)](https://www.howtoing.com/wp-content/uploads/2015/12/List-Dependencies-of-RPM-Package.png)

列出 RPM 软件包的依赖关系

#### 8. 将所有已安装的 RPM 软件包转储到 Zip 文件中

**Yum 调试转储**让你甩了你已经安装的所有软件包的完整列表，任何储存库，重要的配置和系统信息到一个压缩文件中所有可用的软件包。

如果你想调试已经发生的问题，这可以派上用场。 对于我们的方便， **Yum 调试转储**名称的文件作为 **yum_debug_dump- <主机名> - < 时间 > .txt.gz，**这使我们能够跟踪随时间的变化。

```
# yum-debug-dump

```

[![](https://www.howtoing.com/wp-content/uploads/2015/12/Dump-Installed-RPM-Packages-to-File.png)](https://www.howtoing.com/wp-content/uploads/2015/12/Dump-Installed-RPM-Packages-to-File.png)

将已安装的 RPM 软件包转储到文件

与任何压缩的文本文件，我们可以使用 **zless** 命令查看其内容：

```
# zless yum_debug_dump-mail.linuxnewz.com-2015-11-27_08:34:01.txt.gz

```

[![](https://www.howtoing.com/wp-content/uploads/2015/12/View-Content-of-Zipped-Text-File.png)](https://www.howtoing.com/wp-content/uploads/2015/12/View-Content-of-Zipped-Text-File.png)

查看压缩文本文件的内容

如果您需要恢复 **Yum 调试转储**提供的配置信息，您可以用 **yum 调试，恢复**这样做：

```
# yum-debug-restore yum_debug_dump-mail.linuxnewz.com-2015-11-27_08:34:01.txt.gz

```

[![](https://www.howtoing.com/wp-content/uploads/2015/12/Restore-Yum-Dump-File.png)](https://www.howtoing.com/wp-content/uploads/2015/12/Restore-Yum-Dump-File.png)

恢复 Yum 转储文件

#### 9. 从 Yum 存储库下载源 RPM

从库 **yumdownloader** 下载源 RPM 文件，包括他们的依赖。 用于创建要从具有受限 Internet 访问的其他计算机访问的网络存储库。

**Yumdownloader** 允许你不仅下载二进制的 RPM 也是那些来源（如果与使用`--source`选项）。

例如，让我们创建一个名为 **HTOP - 文件** ，我们将存储安装使用 rpm 程序所需要的 RPM（S）。 要做到这一点，我们需要使用`--resolve`与 yumdownloader 一起开关：

```
# mkdir htop-files
# cd htop-files
# yumdownloader --resolve htop
# rpm -Uvh 

```

[![](https://www.howtoing.com/wp-content/uploads/2015/12/Downloading-RPMs-from-Yum-Repositories.png)](https://www.howtoing.com/wp-content/uploads/2015/12/Downloading-RPMs-from-Yum-Repositories.png)

从 Yum 存储库下载 RPM

#### 10. 将远程 Yum 存储库同步到本地目录

**reposync** 密切相关 **yumdownloader（**事实上，他们支持几乎相同的选项），但提供了一个相当大的优势。 而不是下载二进制或源 RPM 文件，它将远程存储库同步到本地目录。

让我们来同步知名 [EPEL 软件库](https://www.howtoing.com/how-to-enable-epel-repository-for-rhel-centos-6-5/)到一个名为 **EPEL 本地**当前工作目录中的子目录：

```
# man reposync
# mkdir epel-local
# reposync --repoid=epel --download_path=epel-local

```

[![](https://www.howtoing.com/wp-content/uploads/2015/12/Sync-EPEL-Repository-to-Directory.png)](https://www.howtoing.com/wp-content/uploads/2015/12/Sync-EPEL-Repository-to-Directory.png)

将 EPEL 存储库同步到目录

请注意，这个过程将需要为它正在下载 **8867** 包相当长的一段：

[![](https://www.howtoing.com/wp-content/uploads/2015/12/Synchronize-Remote-Yum-Repository.png)](https://www.howtoing.com/wp-content/uploads/2015/12/Synchronize-Remote-Yum-Repository.png)

同步远程 Yum 存储库

一旦同步完成，让我们来检查我们的新创建的使用 EPEL 软件库的镜子使用的磁盘空间量[杜命令](https://www.howtoing.com/check-linux-disk-usage-of-files-and-directories/) ：

```
# du -sch epel-local/*

```

[![](https://www.howtoing.com/wp-content/uploads/2015/12/Check-Yum-Repository-Disk-Space.png)](https://www.howtoing.com/wp-content/uploads/2015/12/Check-Yum-Repository-Disk-Space.png)

检查 Yum 存储库磁盘空间

现在，如果你想保持这种 **EPEL** 镜像或用它来安装，而不是使用一台远程包是给你的。 在第一种情况下，请记住，您将需要相应地修改 **/etc/yum.repos.d/epel.repo。**

#### 11. 修复未完成或中止的 Yum 交易

**Yum 完成事务**是赶上一个系统上未完成或中止 Yum 交易，并尝试完成他们**的 yum-utils 的**计划的一部分。

例如，当我们更新通过 **yum** 包管理器的 Linux 服务器有时会抛出其内容如下的警告信息：

> 还有未完成的交易。 您可以考虑首先运行 yum-complete-transaction 来完成它们。

要解决这样的警告消息，并解决这些问题， **Yum 完成事务**命令进入画面，完成未完成的事务，它发现在**交易的所有 *** 并可以发现**交易完成 *** 文件的不完整或中止 Yum 交易 **/ 无功 / lib 中 / Yum** 目录。

运行 **Yum 完成事务**命令完成，未完成交易的 yum：

```
# yum-complete-transaction --cleanup-only

```

现在 yum 命令将运行没有不完整的事务警告。

```
# yum update

```

**注意** ：这个技巧是我们的忠实读者**先生**的一个建议 **托马斯**在评论部分[在这里](https://www.howtoing.com/linux-yum-package-management-with-yum-utils/comment-page-1/#comment-718108) 。

### 概要

在这篇文章中，我们已经介绍了一些通过 **Yum-utils 的**提供的最有用的工具。 有关完整列表，你可以参考手册页（ `man yum-utils` ）。

此外，这些工具有一个单独的手册页（见**男子 reposync，**例如），这是文件的主要来源应该是指，如果你想更多地了解他们。

如果你花一分钟来检查**的 yum-utils 的**的手册页，也许你会发现另一种工具，你想我们更深入地覆盖在一个单独的文章。 如果是这样，或者如果您对本文有任何问题，意见或建议，请随时通过使用下面的评论表向我们发送备注，让我们知道哪一个。