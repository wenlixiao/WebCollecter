> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MjM5NTk0MTM1Mw==&mid=2650653148&idx=1&sn=7c807ce05ac31de51ebe59bf8f5b78c6&chksm=bef9bf5a898e364c5b51f523679c9a6d09d69c7de4c7e2372821fd8c6de6961bbcabcbf9ee9c&scene=21#wechat_redirect)

附：

[Linux 系统巡检报告 / AIX 系统巡检报告](http://mp.weixin.qq.com/s?__biz=MjM5NTk0MTM1Mw==&mid=2650633230&idx=1&sn=59296aac68fab8eec0fdd8cd87e515e4&chksm=bef90a88898e839e498328ac0914c6e4447a7a1074e35328867b81274829cbf1c62de63d082c&scene=21#wechat_redirect)  

[Oracle 数据库巡检模板](http://mp.weixin.qq.com/s?__biz=MjM5NTk0MTM1Mw==&mid=2650632499&idx=1&sn=13e7512e42a0361a0489c5c17aceab0a&chksm=bef90fb5898e86a3e376c7d3b14ef3bb32c40e55ffbca0891e829d610a3c809ce4878b647d00&scene=21#wechat_redirect)   

[VMware 系统巡检表模板](http://mp.weixin.qq.com/s?__biz=MjM5NTk0MTM1Mw==&mid=2650630628&idx=1&sn=29756f63f8cfc53e36f30fefe6259d85&chksm=bef91762898e9e74e0de6e3454c515736213fa08f22e8ef3e8e95f816b725f2b4336d706b964&scene=21#wechat_redirect)  

（以上点击标题可阅读↑↑↑）

IT 巡检内容、方法大全

目 录  

1.  概述 

2.  巡检维度 

3.  巡检内容  

4.  巡检方法

5.  常用命令、常见问题和解决方法

6.  附录 1 词汇表

7.  附录 2 参考资料

**1. 概述**

**1.1 范围定义**

对 IT 系统巡检的逻辑组成，通过对范围定义的与 IT 系统相关的维度的评估，定位当前 IT 系统的健康状况，指导建立改进方案与方针。

**1.2 内容说明**

对 IT 系统巡检的具体评估指标， 用于支持对范围所定义的维度评估结论， 提供具体的数据支持；用于给客户提供巡检类报告的数据提供数据支持。

**2. 巡检维度**

对 IT 系统巡检的评估维度主要包括以下五个方面：

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXaDHM86MXtvpVCv5drTibGwu0YJgQnSXK15L5HI3BoZnwib2jA3nJ7grw/640?wx_fmt=png)  

一个完备的 IT 系统建设应该包括上述所有相关解决方案， 而客户应用系统中在这几方面体现了不同的完备程度。由于用户行业与业务特点，对这些范围的侧重程度不同， 因此我们在评估特定行业用户的 IT 系统之初， 要充分考虑这种行业因素，所得出的结论也是对特点行业用户有指导意义的评估结果。

**2.1 基础设施状况**

IT 基础设施包括系统软件平台和硬件基础设平台。

系统软件平台主要包括操作系统、数据库、中间件。

硬件基础设平台主要包括网络通讯平台和服务器系统平台以及存储系统平台。

对基础设施状况的评估内容包括：

  IT 系统运维环境状况

  IT 系统硬件运行状况

  IT 系统软件平台运行状况

  IT 系统链路状况

**2.2 容量状况**

由于 IT 系统的业务和服务需求可能每天都在发生变化, 信息系统有时会遇到带宽和存储能力不足的问题。要与 IT 系统当前和将来的业务需求相符意味着必须经常地测定容量。容量规划是一种性能价格比很高的手段，可以根据以往的性能统计数字预知潜在的资源短缺情况。

正确的对当前 IT 系统的容量状况做出评估， 是掌握和预测系统当前和未来可用程度的一个重要标志之一，进而也以此为依据做出合理的容量规划。

对容量状况的评估主要包括：

  网络带宽负载状况

  存储的容量状况

  主机系统负载情况

  业务系统所能承载的吞吐量

  软件平台参数配置适用度。

**2.3 性能状况**

IT 系统所提供的业务的性能，是当前业界评价 IT 系统实施成功与否的主要标准之一。

通常对 IT 系统性能状况评估的对象为具体的业务功能模块， 但并不是针对所有的业务功能模块，对这些模块的选取一般遵循以下原则：

  系统日常运行中，使用频率高的功能模块；

  系统日常运行中，业务容易产生相对大并发量的功能模块；

  涉及到的大数据量表操作的功能模块；

  用户反映性能问题突出的模块。

通过选取具有代表性的功能模块，进行性能评测，得出当前系统的性能状况，而这种巡检的环境需要接近真实环境才具有说服力。而本 IT 系统预防性巡检活动通常是在真实的生产环境下完成，因此需要采取适合现场环境的性能评估手段来完成。

对 IT 业务系统的性能评估主要包括以下三个方面：

  业务系统的响应性能状况

  业务系统的稳定性性能状况

  业务容量性能状况

业务系统的响应性能指的是在正常业务并发负载下，以响应时间为主要关注点的业务模块操作的执行时间，通常单位为秒；

业务系统的稳定性性能的主要关注点则是在长时间较大负载压力下，业务系统能够正常完成业务操作的程度；

业务容量性能状况指的是当前业务系统负载承受能力，目的是了解系统的业务压力可承受的范围，以便在峰值到来之前做出应对措施，通常关注的性能指标为并发量和业务的吞吐量。

**2.4 信息安全**

这里把信息安全定义为信息系统数据不会被非法用户在未经授权的情况下取得或破坏。信息安全所涉及的技术与业务层面很广，以下是对其简要分类：

**1. 物理安全**

保护信息系统的机房环境、设备、设施、媒体和信息免遭自然灾害、环境事故、人为物理操作失误、各种以物理手段进行的违法犯罪行为导致的破坏、丢失。

**2. 网络系统安全**

网络防护安全是数中心据安全的重要组成部分。网络安全模式要求数据中心首先分析自己的网络系统，并从中找出不同业务、数据和安全策略的分界线，在这些分界线上构建 IT 系统安全等级不同的安全域。

在安全域划分的基础上，通过采用入侵检测、漏洞扫描、病毒防治、防火墙、网络隔离、安全虚拟专网（VPN）等成熟技术，利用物理环境保护、边界保护、系统加固、节点数据保护、数据传输保护等手段，通过对网络和系统安全防护的统一设计和统一配置，实现 IT 系统全系统高效、可靠的网络安全防护。

**3. 操作系统安全**

操作系统提供若干种基本的机制和能力来支持信息系统和应用程序安全，如身份鉴别、访问控制、审计等等。目前主流的商用操作系统主要有 UNIX、LINUX 和 Windows 平台。由于商用的普遍性特点，这些系统都存在许多安全弱点，甚至包括结构上的安全隐患， 比如超级管理员 / 系统管理员的不受控制的权限、 缓冲区溢出攻击、病毒感染等。

操作系统的安全是上层应用安全的基础。提高操作系统本身的安全等级尤为关键，除了及时打 Patch 外，还要采用如下的加强措施：

  身份鉴别机制：实施强认证方法，比如口令、数字证书等；

  访问控制机制：实施细粒度的用户访问控制、细化访问权限等；

  数据保密性：对关键信息、数据要严加保密；

  完整性：防止数据系统被恶意代码比如病毒破坏，对关键信息进行数字签名技术保护；

  系统的可用性：不能访问的数据等于不存在， 不能工作的业务进程也毫无用处。

因此操作系统要加强应对攻击的能力，比如防病毒、防缓冲区溢出攻击等；

  审计：审计是一种有效的保护措施，它可以在一定程度上阻止对信息系统的威胁，并对系统检测、故障恢复方面发挥重要作用。

**4. 数据库安全**

数据库安全性问题应包括两个部分：一、数据库数据的安全。它应能确保当数据库系统 DownTime 时， 当数据库数据存储媒体被破坏时以及当数据库用户误操作时，数据库数据信息不至于丢失；二、数据库系统不被非法用户侵入。它应尽可能地堵住潜在的各种漏洞，防止非法用户利用它们侵入数据库系统。

**5. 数据的传输安全**

为保证业务数据在传输过程的真实可靠，需要有一种机制来验证活动中各方的真实身份。安全认证是维持业务信息传输正常进行的保证， 它涉及到安全管理、加密处理、 PKI 及认证管理等重要问题。应用安全认证系统采用国际通用的 PKI 技术、X.509 证书标准和 X.500 信息发布标准等技术标准可以安全发放证书，进行安全认证。当然，认证机制还需要法律法规支持。安全认证需要的法律问题包括信用立法、电子签名法、电子交易法、认证管理法律等。

**6. 应用身份鉴定**

由于传统的身份认证多采用静态的用户名 / 口令身份认证机制， 客户端发起认证请求， 由服务器端进行认证并响应认证结果。用户名 / 口令这种身份认证机制的优点是使用简单方便，但是由于没有全面的安全性方面的考虑，所以这种机制存在诸多的安全隐患。可以采用：双因子认证和 CA 认证两种解决方案。

**7. 应用授权管理**

权限管理系统是 IT 系统信息安全基础设施的重要组成部分，是 ICDC 信息系统授权管理体系的核心。它将授权管理和访问控制决策机制从具体的应用系统中剥离出来，采用基于角色的访问控制（RBAC，Role Based Access Controls）技术，通过分级的、自上而下的权限管理职能的划分和委派，建立统一的特权管理基础设施（PMI，Privilege Management Infrastructure） ，在统一的授权管理策略的指导下实现分布式的权限管理。

权限管理系统能够按照统一的策略实现层次化的信息资源结构和关系的描述和管理，提供统一的、基于角色和用户组的授权管理，对授权管理和访问控制决策策略进行统一的描述、 管理和实施， 提供基于属性证书和 LDAP 的策略和授权信息发布功能，构建高效的决策信息库和决策信息库的更新、同步机制，面向各类应用系统提供统一的访问控制决策计算和决策服务。建立统一的权限管理系统，不仅能够解决面向单独业务系统或软件平台设计的权限管理机制带来的权限定义和划分不统一、各访问控制点安全策略不一致、管理操作冗余、管理复杂等问题， 还能够提高授权的可管理性， 降低授权管理的复杂度和管理成本，方便应用系统的开发，提高整个系统的安全性和可用性。

**8. 应用访问控制**

访问控制是 IT 系统安全防范和保护的主要核心策略， 它的主要任务是保证信息资源不被非法使用和访问。访问控制规定了主体对客体访问的限制，并在身份识别的基础上，根据身份对提出资源访问的请求加以控制。它是对信息系统资源进行保护的重要措施，也是计算机系统最重要和最基础的安全机制。根据控制手段和具体目的的不同， 数据中心的访问控制技术包括以下几个方面：入网访问控制、网络权限控制、目录级安全控制、属性安全控制等，只有各种安全策略相互配合才能真正起到保护作用。

**9. 应用审计追踪**

IT 系统的安全审计提供对用户访问系统过程中所执行操作进行记录的功能，将用户在系统中发生的相关操作（如：系统登陆 / 退出、系统操作）记录到数据库中，以确保在需要的时候，对用户历史访问系统的操作进行追溯。

通常审计跟踪与日志恢复可结合起来使用，日记恢复处理可以很容易地为审计跟踪提供审计信息。如果将审计功能与告警功能结合起来，就可以在违反安全规则的事件发生时，或在威胁安全的重要操作进行时，及时向安检员发出告警信息，以便迅速采取相应对策，避免损失扩大。审计记录应包括以下信息：事件发生的时间和地点；引发事件的用户；事件的类型；事件成功与否。

在 IT 系统中，审计可以是独立工作的不相关的组件的集合，可以是相互关联运作的组件的集合。审计范围包括操作系统和各种应用程序。

**10. 安全管理与策略**

IT 系统安全管理系统应包括管理策略、管理组织保障、管理法规制度以及管理技术保障等内容。

IT 系统安全是一个动态不断调整的过程，它随着 IT 系统业务应用和基础设施的不断发展而不断改变，例如 IT 系统信息系统各个信息网络、信息安全部件的具体设置规则，包括特定系统（设备）的口令管理策略、特定防火墙的过滤规则、特定认证系统中的认证规则、特定访问控制系统中的主体访问控制表、安全标签等。为了保证 IT 系统信息安全，及时进行安全策略调整是必要。管理组织保障，实现对人员、系统、安全设备、物理环境和系统运行的安全管理。另外，IT 系统安全策略应遵照相关行业的法律、规定。

管理技术保障是 IT 系统安全运行管理的技术保证。

**2.5 业务连续性**

连续性是指一个数据中心类应用为了维持其生存， 一旦发生突发事件或灾难后，在其所规定的时间内必须恢复关键业务功能的强制性要求，这就需要预先发现可能会影响企业关键业务能力和过程的所有事件， 采取相应的预防和处理策略，以保证企业在事件发生时业务不被中断。通过业务连续性计划保证数据中心业务的不间断能力，即在灾难、意外发生的情况下，无论是数据中心组织结构、业务操作和 IT 系统，都可以以适当的备用方式继续业务运作。

严格的说，业务持续计划的建立和实施过程，实际上是涉及数据中心运营，因此也涉及到项目管理的方方面面。通过多年的实践，根据自身实践经验并参照国际灾难恢复协会（DRI）与业务连续性协会（BCI）的标准，总结出业务持续计划的模型，经过长时间的验证，该业务持续计划模型能够给数据中心带来有效及彻底的业务持续管理。

灾难恢复的技术实现和级别——

容灾按级别可分为数据容灾和应用容灾两部分：

数据容灾：在异地建立一个数据拷贝，这个拷贝在本地生产系统的 “数据系统” 出现不可恢复的 “物理故障” 时，提供可用的数据。

应用容灾：在异地提供一个完整的应用和数据系统拷贝（不一定要求同当量），这个拷贝在本地生产系统出现不可恢复的 “物理故障” 时，提供即时可用的生产系统。

**1. 平台安全性**

平台完整性解决 ICDC 内部业务平台和接入平台的高可靠性问题。主要包括服务器、存储和网络层面的技术。

平台完整性涉及的技术主要包括：服务器、存储器、及相应网络连接的部件级可靠性技术；平台的集群技术；Application Server 的高可靠技术；数据库的高可靠技术。

**2. 备份和恢复完整性**

备份和恢复完整性实现 IT 系统内部对业务数据平台的保护。包括服务器和存储层相关技术。

备份完整性涉及的技术主要包括基于磁带、光盘等离线介质的备份技术（或称定点拷贝） ；以及基于在线存储介质（磁盘）进行的生产数据快照技术。

实现备份完整性目标，首先需要映射业务种类所需要的数据集。即根据容灾备份系统的需求，明确哪些业务状态数据需要备份，事实上，需要提供最完善备份的是稳定的业务状态数据， 而处理流程当中的中间临时数据的备份需求较低。

另外，在备份完整性的实施过程中，应该区分备份数据和存档数据。备份数据是为满足容灾备份的要求，具有较短的时效性，备份数据会根据一定的备份频度被反复覆盖。存档数据则按照业务或法规的要求，有较长的时效性，并具有不断累积的特性。

在绝大多数数据中心应用场合， 备份是经常性的工作， 恢复是十分偶然的操作，因此， 恢复往往是难以经过充分巡检、 优化的容灾备份技术 --- 这就更加要求恢复操作具有明确的可预见性。

**3. 信息完整性**

信息完整性实现对业务数据平台的跨 ICDC 生产中心的保护， 实现信息完整性技术是将业务连续性扩展到容灾阶段的一个十分关键的步骤。

信息完整性技术将生产中心的业务状态数据完整地复制到备份中心。

实现信息完整性可以采用同步或异步复制技术。

**4. 处理完整性**

处理完整性即对业务支撑系统平台的完整的、跨越生产中心的保护。

实现处理完整性， 需要比较复杂的系统集成工作， 包括详细的系统设计和规划。

目前的大多数关键业务及其关联子业务系统的容灾的级别要求为处理完整性。

实现处理完整性的关键在于以下三个要素：

  对数据平台的保护－远程数据复制技术（即信息完整性）和对业务平台的保护－服务器、数据库等冗余及切换技术以及应用软件切换技术的集成

  对接入平台的保护和切换－外部接口的冗余和切换

  系统的监控和切换

**5. 业务连续性管理**

业务连续性管理是 IT 信息安全政策的宏观管理文件， 该规范清楚说明业务连续性计划对于保障信息安全所采取态度、监管责任以及信念。

业务连续性管理规范包含《灾难恢复预案》 、 《业务连续性计划》等文件。这些规范从宏观层面，涵盖了灾难备份建设所涉及的内容，其目的是要保护信息安全。根据这些规范，建立业务连续性计划、灾难恢复预案，其中主要包括：灾难应急小组的组织架构和人员职责， 应急队伍、 联络清单及各类应急处理流程，普及教育及人员培训计划和演习计划等，并报主管部门备案。

主管部门要对各单位灾难恢复预案进行全面审核，评估灾难恢复预案的完整性和可操作性，配合建立规范的管理制度和操作文档。

定期进行灾难演习与应急培训。

**3. 巡检内容**

上一节完成了对 IT 系统巡检的关注方面的分析说明， 这一节开始介绍具体体现这些关注方面的指标，在实际检查过程中，可以根据客户的需要选取特定的指标参数，作为评估目标系统的数据支持内容。

**3.1 系统整体架构**

以下内容作为基本 IT 系统信息被首先调查记录，供分析参考使用。

  IT 系统架构拓扑图

  网络设备配置

--- 设备型号, IOS 版本, 模块型号和数量, 用途

  存储系统配置

--- 设备型号, IO 带宽, Cache 容量，磁盘数量，接入模式，存储容量，LUN 配

置，所属应用

  主机系统配置

--- 设备型号，CPU 配置（类型，主频，数量） ，内存容量，网卡配置（数量，

速率） ，内置硬盘配置（数量，容量，Raid） ，所属应用

  数据库软件

--- 产品名称，版本号，所属应用

  中间件软件

--- 产品名称，版本号，JDK 版本，所属应用

  应用系统

--- 产品名称，版本号，架构平台，系统架构类型

**3.2 机房环境**

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXMgmL6EIj9Ciao4tDHBBX7qQhbfZIeRZVj5OgtJunLspmaCXFPUGHl6A/640?wx_fmt=png)  

以上的条件可以现场观察和询问用户完成。  

**3.3 网络系统**

网络设备

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXGUk7N4r4K0Dkho8FpdKRgR9IEvTMILVVNiadv5unTcA6tskE4KgHIFQ/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXo6jRCqnDezuXkT3PkCuKicZicXKzexYm9CTRUCa4blEuEEG6ubIicuHicg/640?wx_fmt=png)  

防火墙

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXIY1icEicFZDJZMBU9tY9WuvQkk1c1amS9nkPMicxMKu1OpDXtD7kMTzWA/640?wx_fmt=png)  

IPS

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXbgcKbKrdo2GSwpn8fNursyYXfPZVRrVfXe7t4rybsJPMcKAicechsSA/640?wx_fmt=png)  

IDS

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXwXpNdhGBMA22w8rgJn7ePcEiakrTPCIibnehCXvfnoqarMEguaHMUyWg/640?wx_fmt=png)  

VPN

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXmXqFUxBeheyRvY2WCgicm9EXRoRw6VQGHArubwUCLAemv43JNNlxyMA/640?wx_fmt=png)  

**3.4 存储系统**

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXMiaQWfPLCT2NtHXz4A128Zao0IZAkEeTdicAZusw4tBcUnVgIicCfLXDw/640?wx_fmt=png)  

**3.5 主机系统**

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXQCYYTq6ZAibibNL24YibiboDWnHN8pQN45J0zemibacu2YbdkPooj4xdK0w/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXyjaADj2AQcc3o9JNkIysA3LbDX0dHicFA5xeYOoZPy0JFP6XcKrVYzg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXg33AsUlhpib5lChAVcbW29WxfAgXh3fzOMVibeZr89XicwxZZ3cROlSYQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXFJiajTT8T8xeJ9hEUo6LicAqTI6VQW8ibnjoh8EPkRadUr3qP4Wc2FVpQ/640?wx_fmt=png)  

**3.6 数据库系统**

**3.6.1 Oracle 数据库**

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXJiapSmaaFcCmszyRBDwWv20rJH8JjWTU3jic4RVuW7ZVZ8RwFZENwkAA/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXCogONRvROiadggS2wx1mAmkFpB14evN8wJg51kdv1eOVW02Ojghea5g/640?wx_fmt=png)  

**3.6.2 DB2 数据库**

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXAxFKiaM1nsjbrWhGxXvaiadW1p0mBHSK9zxiaBGpibzXlKcr7DTibPPIX5w/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXksaeKTKlzgV33ib3tNEuCMU0YaNXO9b7FMdUT37xFichfbRnK6SibBrfA/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXrrKe5KMpYIRv0QsibMibiclePNfWF6ibZbMRI3pTK0ckMREGib0foF6wSUQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXicqPbUuicIeurqAGicCAXjWPVKFqlWMgvEMib6nJqlkGtp71AhkPSFpH7w/640?wx_fmt=png)  

**3.7 中间件系统**

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXJFGAuWyPtTH8YKFJbeMHJu7p5vEsyUWiaAWw0Wd9ESljxktynicVaeSQ/640?wx_fmt=png)  

**3.8 应用系统**

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXdibX646cAicHVAoC0KqqkqddSiaKy5R88m6vwMDEk7qEZMWvhhEibywXrA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXxSny89ibfBojxhiberbia7QdGks0yciaqqhxRfynmNiauZxhE3wu5LwUO5Q/640?wx_fmt=png)  

**3.9 备份与恢复系统**

备份与恢复系统是 IT 系统中重要的容灾措施，IT 系统应该根据自身业务特点选取以下备份与恢复方案。

**1. 备份系统**

设备系统备份：

部件的冗余

--- 包括网络设备，主机设备，存储设备内部部件的冗余，保证在设备本身避免单点故障。

设备的冗余

--- 网络层设备冗余包括交换设备的 HA 和线路冗余， 交换设备的 HA 可以实现故障发生时自动切换。

--- 主机层设备冗余可以采用冷备与热备两种方式， 热备即主机集群， 实现故障发生时自动切换。

--- 存储层的设备冗余指阵列间的镜像和异地复制方案。

数据系统备份：

系统级归档备份

--- 一般采用磁带备份方式，备份设备可选取磁带机或磁带库

--- 制定备份策略，可以按一段时间周期，将完全备份、增量备份和差分备份组合使用制定备份策略。

--- 系统级归档备份的备份数据与在线生产数据存在备份间隔差异， 对数据库数据采用这种备份时应将数据库设置为归档模式，来消除这种差异，保证数据的完整性。

存储级数据备份

--- 本地镜像

--- 同城容灾镜像

--- 异地数据传输，分为同步和异步模式。

应用系统备份：

应用系统备份基于网络备份，主机系统备份和数据备份的整合，方案中涉及以下因素：

本地应用系统备份，远程应用系统备份

手动应用切换，自动应用切换

应用系统备份是备份方案中级别最高的备份形式，而其中自动应用切换的远程系统备份方案则是最高级备份方案，保证应用的完整性。

**2. 恢复系统**

备份系统完成 IT 系统的容灾保证的一般工作， 恢复的成功与否是衡量备份方案有效的唯一标志。

备份是多次重复工作，而恢复操作则较少发生，这种情况下，验证备份有效性就尤为重要。通过制定以下策略与措施，保证恢复策略的有效性：

*   制定恢复应急预案
    
*   制定恢复流程
    
*   定期进行巡检、培训与演习
    

**4. 巡检方法**

对照巡检计划的安排，对主机系统进行硬件、操作系统进行功能及性能检查。

注意：系统中所使用的每台主机都要单独列表检查。

**4.1 IBM 主机**

巡检对象：XX 系统 XX 服务器 (HOSTNAME)

巡检目的：检查 XX 系统 XX 服务器的状态

巡检平台：XX 系统主机，超级用户

前提条件：线路通畅

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXEZopuz3L5NZicacRFIQC9ZVdVJmj9XlFMY9iaWLXFBagaq8I6RACWDTA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXsesUibw7oY3zvrUAPEBj6L2ib7iaTXvOgut2OlnQD8oibRzkl8rjBiarPcQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXyg48qs6UXQTC2utRStIRqppCFsvvAEEbKCicgwHZYv7tJtcQzwMlc7g/640?wx_fmt=png)  

**4.2 IBM HACMP Cluster**

巡检对象：XX 项目双机系统

巡检目的：XX 系统双机热备功能正常

巡检平台：XX 系统主机，超级用户

前提条件：线路通畅

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCX8qq4KgkprgoWUyKojpK9icZHQC41T32qhYGOIx9ic6eDIHxtDYZ0yOmQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXzwvYGk514hVjZmUEjjEicWNaMoOEia5ZhJLic8bPTa8KTaJ3zPFsy0UBQ/640?wx_fmt=png)

**4.3 HP 主机**

巡检对象：XX 系统 XX 服务器 (HOSTNAME)

巡检目的：检查 XX 系统 XX 服务器的状态

巡检平台：XX 系统主机，超级用户

前提条件：线路通畅

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXg7Btm1T0LFMTiaCt3MgtUJtaib9dl9jkBiaNxj44zmLp2H5ktB4XpiaKuQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXfwB0n5tY925cyuTWqSISw6osrvuf4jjSiccquy5AfaSRLF6tjN1Mt8g/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXe8ZW6vWtvQGUsH9CWwtagM8gCxlwQbcro0198xB7rge2zQ1fkcFRMA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXNmM0QVdcY7cbjL81S7AQVYDP3kuQIZvMCf2tfz5iaIsgcXuh9I1ickicw/640?wx_fmt=png)  

**4.4 HP MC/ServiceGuard Cluster**

巡检对象：XX 项目双机系统

巡检目的：XX 系统双机热备功能正常

巡检平台：XX 系统主机，超级用户

前提条件：线路通畅

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXI2V0AQ2vReXIA4OW04PZP7ImlOuAaae7eVD1dZmWQsvvek7631U8gw/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXZqolkUWaULvull8M0eAA7SWAOH4HxZDibibMibDogHTDvQiaII05wial7IA/640?wx_fmt=png)  

**4.5 SUN 主机**

巡检对象：XX 系统 XX 服务器 (HOSTNAME)

巡检目的：检查 XX 系统 XX 服务器的状态

巡检平台：XX 系统主机，超级用户

前提条件：线路通畅

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXjMpjvBG4erZVZrhwXAC2tGur3hmRb4SKQcO5CrAdQPFJhRELNgfP8w/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXCRKsrOW4icTibMtjX18cvF6XO7ITO5R9yPcicbtLvbh3uhzpqw9pThQ6A/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXuuh1xHExqJCqWTfiaxc6ApKt1sAMjdtcialrWFzswEKkOjR77y1R3KfA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCX0KKXP34v755DmBMbynJDFv0UaXa0FgbCoWtAdX9azKiaQ9LOHbvTwyg/640?wx_fmt=png)  

**4.6 VCS Cluster**

巡检对象：XX 系统 XX 服务器 (HOSTNAME)

巡检目的：检查 XX 系统 XX 服务器的状态

巡检平台：XX 系统主机，超级用户

前提条件：线路通畅

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXa23KRha7icCicc4MYM1dfUJuImjP4tY6micrYLsxeYicicN06scvueZvoHA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXz9pLjcbBmbyicM06LLGqIQTOm6InaAcyw1I18Bfia63oQrftdG66rV1Q/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXsSa8PJuHiajew1R1Cy6uaVGD9nG0jYtHlQOmRq4fY6d21voFibPPRhFw/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXuxaFk4cBgia72HN1fnqLO8XibPtq5x5dWcsGewkUNXeOdu4XS1jkzjXA/640?wx_fmt=png)  

**4.7 网络部分**

对照巡检计划的安排，对网络设备进行硬件、操作系统进行功能及性能巡检。

注意：系统中所使用的每台网络设备都要单独列表巡检。

**4.7.1 XX 网络设备**

巡检对象：XX 系统网络设备 (NAME)

巡检目的：XX 系统网络设备的系统状态

巡检平台：XX 系统网络设备，超级用户

前提条件：线路通畅

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXjPRhcpZDrPzPnfvu2k852wYmEGBStmibbibGGZX9JuGfPK2JSchXia6dA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXiaz3FVTHv4CePNzv8KSoQqeZicPX2wlL7Pia1a359r1LsXHkricc5L25Nw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXPDb3h7DeRDM30tIssDAAdvxIsNNyScIw9IgNuSp1ZPaD40etXnztYw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXXicwHI2TliaTtzlLKM5dOzFkx5yVuibfKs094LZQyhRE7bYa9VTicunR8A/640?wx_fmt=png)  

**4.7.2 XX 网络设备**

巡检对象：XX 系统网络设备 (NAME)

巡检目的：XX 系统网络设备的系统状态

巡检平台：XX 系统网络设备，超级用户

前提条件：线路通畅

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXAUfca7JkWicickkZXNNszqFLAalSlgmQkEeicFfYITQBFXZqLsaqwFT7Q/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXnFicibyJ4iaN8bFXrLOmCEX29bQkE6mZeXeokNtEE9qdO6RzhkKmMamug/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXo68RA2uNIcajicQibpb3OACmUf3LrDbEbqmatDZGRqibAO5dFnKHslMicA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXJibfD9TuWUfCXqqtHPZ3urOSdJBWeaiar3icIBpeXNcWzYQ2NxkAE9C3A/640?wx_fmt=png)  

**5. 常用命令、常见问题和解决方法**

**5.1 机房环境**

对机房的基础设施配备应该按照标准实施， 不符合标准的项目应该尽可能整改，添加应有设施。对 UPS 的维护应该定期进行检测，巡检其供电的有效时间，一旦发现电池老化应尽快更换。

**5.2 网络系统**

网络设备

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXtPiaQQMsiaBpZxg6bSTwdtfOaq9b5eRT0wyJkOYFzicLNBiaZ3s8vTquEg/640?wx_fmt=png)  

Cisco 系统的一些巡检常用命令列表：

总体的信息收集  show tech

查看 ios 版本等信息 show version

查看 log  show log

查看设备的时钟  show clock

查看接口状态  show ip int bri

查看设备路由情况 show ip route

查看 ios 软件包  show flash (或 show bootflash /show disk0)

防火墙

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCX73iaAgYukqhD5OV9ibFqjsRvia8EzfnSOKXrOuASKVaMoFibDdBRkpDBEw/640?wx_fmt=png)  

IPS

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXvibzOvAkL057p5OWERUpZib4O6O9CSVwqEs8xaaZ6W7EaHTd8BOD7Ggg/640?wx_fmt=png)

IDS  

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCX7icNweJ5FD5H2cHAVGicZjCn8ibzPWCCiczjDr7ETqLLdpgFNnlcOmKJNQ/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXQXsFfAVGJBP6Nrgqib0M0oAUkuHaP1X9HOGxvdaJXDDNgjBTjw0oiamA/640?wx_fmt=png)  

VPN

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXHMcCJ9GfUteFNgV1thlhwqCsp6D638BsDvEPV4ASFrql5gpeRj1AcA/640?wx_fmt=png)  

**5.3 存储系统**

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXdAnZ4Sf4oBv4iaWrOcqypKm8HiaC7RECChVOBjibjwxCPNdfwnoicOyhtQ/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXpesNTOqHbJJmAz7Hia7PKPwLOgJvJyOKvUYGrsTcC5PNmibs13wWich6Q/640?wx_fmt=png)  

Sun T3 阵列的常用命令列表：

系统状态  sys stat

系统配置  sys list

系统部件状态  fru stat

系统部件列表  fru list

卷的列表和状态  vol list，vol stat

SUN StorEdge 3000 系列阵列 cli 命令列表：

显示阵列全部配置 show configuration

查看设备网络状态 show network-parameters

组件状态命令

show battery-status

show enclosure-status

show frus

查看磁盘信息  show disks

查看逻辑设备卷等

show logical-drives

show luns

查看分区状态

show lun-maps

show partitions

show logical-volumes

显示 firmware 版本

show ses-devices

show deses-devices

**5.4 主机系统**

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCX9yJLVKn6TwgI0jYDXHcNLbsF1wqebyiagpyelzBzzRExGc2nibzqcxbQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCX8EuvsMCpSg1iaJnSDRaMibpz5L1hheViaxlbUlAa252P8WogVyylrJusA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXsEmRoEWQ6dIiaDG3R9bWv31mYV2IjMl8EBb0NnRC1KGXROWKG4JSVaQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXNzK9gwK7JXuAZ43sAvE8w5OBruibgQWC27FCphehSdulLu37TkgG45A/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCX9dEurd75gFt8OicpEOW8UyPRp6hnjDDpNuCvvLDfg3jZ0bkEsrx0Vtg/640?wx_fmt=png)  

**5.4.1 sun solaris 主机命令**

查看系统运行状况设备运行状况

tform/sun4u/sbin/prtdiag –v

查看系统日志

grep WARN /var/adm/messages*

grep error /adm/messages*

grep panic /adm/messages*

查看网络状态路由配置

ifconfig –a

netstat –rn

磁盘和分区使用情况

df –k

format

disksuit

metastat,metadb

volume manager

vxprint –ht

CPU

psrinfo

sar 1 10

vmstat

prstat

系统补丁  uname –a

进程情况  ps –ef

磁盘 IO 状况有无错误

iostat –En

iostat -xn 3

**5.4.2 IBM AIX 主机命令**

查看系统运行状况设备运行状况

prtconf

lscfg –pvv

查看系统日志

errpt

errpt -a|more

errpt -a -j 日志号

查看网络状态路由配置

ifconfig –a

netstat –rn

磁盘和分区使用情况

df –k

lsdev -Ccdisk

lsvg –o

lsvg –l 磁盘组

lsps -a

CPU

lsdev -Ccprocessor

系统补丁 

进程情况  ps –ef

磁盘 IO 状况有无错误  

iostat –En

iostat -xn 3

**5.4.3 HP-UX 主机命令**

查看系统运行状况设备运行状况 

查看系统日志

vi /var/adm/syslog/syslog.log

列出 I/O 卡的相关信息  ioscan -fn

查看网络状态路由配置

lanscan

netstat –rn

磁盘和分区使用情况 

bdf

vgdisplay -v vgxx

lvdisplay -v LVxx

ioscan -funC disk

pvdisplay -v /dev/dsk/c*t*d*

CPU

系统 ID OS 版本  uname -a

进程情况  ps –ef

磁盘 IO 状况有无错误  iostat –En

**5.5 数据库系统**

**5.5.1 Oracle 数据库**

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXNWY7RslL76j2d3ECDZvLL7u08xkKPcxSe7NRshMsMbJgw45hX6tnmw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCX1c1b8zOMIfIUL0Ut7HTiaNjibMzp6wBz9wvqAOwCh1ibvFG6pZM4QQagQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCX1Fvjr8Rv9gqyv7If9eFjJ15oO7LAoZ0WOBdmKWwxSoR2Rgn9QXaHSg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXqB5BOX3c8TU0wicNBKWroz8MLbibLCf9s3icxv9TOGCqEuEFpubbiaBATQ/640?wx_fmt=png)  

Oracle 命令列表：

数据库 alert 日志信息――检查日志中是否有错误信息提示。

初始化参数 ―― show parameter;

检查控制文件状态―― select * from v$controlfile;

检查联机日志文件状态―― select * from v$logfile;

检查数据文件状态―― select * from v$datafile;

检查表空间使用率――

select  b.file_id  "File  ID",b.tablespace_name

"TabSP_Name",b.bytes/1024/1024 "Size(M)",

(b.bytes-sum(nvl(a.bytes,0))) "Used",sum(nvl(a.bytes,0)) "Free",

sum(nvl(a.bytes,0))/(b.bytes)*100 "Free Per%"

from dba_free_space a,dba_data_files b

where a.file_id=b.file_id

group by b.tablespace_name,b.file_id,b.bytes

order by b.file_id;

检查回滚段使用情况――

SELECT SEGMENT_NAME,OWNER,TABLESPACE_NAME,SEGMENT_ID,FILE_ID,STATUS

FROM DBA_ROLLBACK_SEGS;

检查用户状态――

select

username,account_status,default_tablespace,temporary_tablespace,crea

ted from dba_users;

是否存在失效对象――

select owner， object_name,object_type from dba_objects where status =

‘INVALID’;

是否有异常等待事例 ――

select event,sum(decode(wait_Time,0,0,1)) "Prev",

sum(decode(wait_Time,0,1,0)) "Curr",count(*) "Tot"

from v$session_Wait group by event order by 4;

检测连接数情况 ――

SELECT status,count(*) "count" FROM v$session GROUP BY status;

用户使用情况 ―― 向客户了解使用过程是否有问题。

**5.5.2 DB2 数据库**

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXqib2CROStjYLarGZz2QqFB0FQcLQGVv3cP7xFknyXe0jia9tkHrlC6bg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXBkNNdicNFbAF7nWLHicBncAkPx0YR2qXmLmUCHehd9Qg5jRbXCj6H2Bw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXkaKHTgvCUGdLQsPGhlVHuqfMmJ1A9tIhLO8TLibk6XkA0T99hoMDVhA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXoyOuEm8tkGiaD6VAQBalc6O4TBGGoJggicuAJFicGIQSz2Q12mZMhCEng/640?wx_fmt=png)  

**5.6 中间件系统**

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCX6fcGsEqySTTt37ANlvHKaiauL9N6nDcBlYRysUjicziaYFCdaHDmDd73g/640?wx_fmt=png)  

**5.7 应用系统**

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXjsUzdMWdTo7pkr5L37bNFNiafrRqpfeLFLxiaWMv6OsMWcKicmo8ZqL8g/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXjDy1POeNQXGMxC9ZrfGIIVYAXqBpPF3uFOuetLpNXia3Ky0mXyxK6Wg/640?wx_fmt=png)  

**6. 附录 1 词汇表**

列出本巡检方案中专门术语的定义、英文缩写词的原词组和意义、项目组内达成一致意见的专用词汇，同时要求继承全部的先前过程中定义过的词汇。

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXlFL7kDYPEYeGW7JoOejj7d5CltDBicQs8VicwqedXhX4Cn2av0SGrBng/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXTRouq5GFDImuCzaava2SoIQoiaNicefRlqnCp4O6LmEfzaCYfL2FvP5w/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCX4xficBQbTz0C4ZrFJIN9dSr4s3Ef1tcbgjJYUibAg2fWh91DabjY3ZPw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXOxH6UGAmTcYgvPiag3Tf1wkwO69E267PejibKD92YnsM8cEBiaGQgbqzQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCX7ic6KibEDf9aY2ZkurrcsXZI915LBCGGYnnqEiaxQ8xKVIXIFuCaWFqyg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXnbrk4JK6MhBAhz0knrWk0yNcU1ibcVtIErkxHibGYtctaCDrk5DUuydw/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCX9ZQX2WuQZa1cSFiaG34s4Wy5BqmbjI8icbpbUFsibllbmSrFrbGG6Gliaw/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToI8icEcmeY8ctRsRRNwRRicZCXoDpwW8tqHtqLO7fecJWJgwmYiba4eDx9VS74cg3j05MAA6sHUD73Zicw/640?wx_fmt=png)  

备注中注明该词汇的来源，或有其他更详细的解释的文档位置；以及对该词汇的其他叫法。

**7. 附录 2 参考资料**

本方案同时查阅了以下 Internet 网址上的技术标准及信息。

IBM e-Server p-Series 信息中心：

http://publib16.boulder.ibm.com/pseries/en_US/infocenter/base

IBM Redbooks 网站：

http://www.redbooks.ibm.com

HP 公司网站：

http://www.hp.com

SUN 公司网站：

http://www.sun.com

CISCO 公司网站：

http://www.cisco.com

EMC2 公司网站：

http://www.emc2.com

> 由社区会员上传分享，点击阅读原文可下载原文档

 资料推荐：

[VMware 资料大全 | 周末送资料](http://mp.weixin.qq.com/s?__biz=MjM5NTk0MTM1Mw==&mid=2650651724&idx=1&sn=b7dcd884c4d8c269e9144f4f03d1fde3&chksm=bef9c2ca898e4bdc60a6281c5011105265d28c7f603988e3427ca99ff9b6686a078b7e9de289&scene=21#wechat_redirect)  

**下载 twt 社区客户端 APP**

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToIicEhGeibB7bXWzcpZw2W1MtYUE1nEjaJ9GUdRLWxKpriaKibjWLXeTMtgKOUib4dLibBWojKDpX5hIMkxQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/icdBCpcEToIicEhGeibB7bXWzcpZw2W1MtY4XZqa0GbU2yZxwDk8sqkNpnTBPlXPicSUlltBa8ODoeX6T6ftVicOdqg/640?wx_fmt=png)

长按识别二维码即可下载

或到应用商店搜索 “twt”

长按二维码关注公众号

**![](https://mmbiz.qpic.cn/mmbiz_jpg/icdBCpcEToI8sSChzbkLyCCuqNd1gwh4XY2JjYj0yicYY5cNjR2f44zRcISdax4mfAOAzLqrJSSQ5Dz8DV7Jx3uQ/640?wx_fmt=jpeg)**

* 本公众号所发布内容仅代表作者观点，不代表社区立场