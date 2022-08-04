> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/xueshenlaila/article/details/118895406)

K8s（[kubernetes](https://so.csdn.net/so/search?q=kubernetes&spm=1001.2101.3001.7020)）作为行业的新宠，近期都霸屏了！

云原生计算基金会（Cloud [Native](https://so.csdn.net/so/search?q=Native&spm=1001.2101.3001.7020) Computing Foundation）公布了第三次中国云原生调查报告，报告显示 49% 的受访者在生产中使用容器，72% 的受访者在生产中使用 Kubernetes（简称 K8s），公有云使用率下降至 36%，更多的企业选择混合云。

Kubernetes 作为容器管理工具在企业使用占比越来越高，世界 500 强企业也都以 K8S 作为核心技术支撑，大公司都在争夺 K8S 人才。国内外知名公司都在基于 Kubernetes 搭建智能化的容器云管理平台。高端 K8S 运维工程师、容器云工程师、微容器开发工程师等，月薪动辄都在几十 K。

2016 年底，京东业务开始从 OpenStack 切换到 Kubernetes，第一阶段迁移 20% 的业务到 kubernetes 集群，规模是 500 + 物理节点，2w+Pod 容器。2020 年，已经把 90% 业务迁移到 k8s。

京东大规模内存数据库的请求次数峰值从每秒 5.5 亿到 8 亿，再到 17 亿；网关访问峰值从每秒 200 万到 300 万，再到超 400 万；大数据实时数据处理从每分钟 30 亿到 70 亿，再上升至 110 亿条，流量压力屡创新高。京东 618 当天活跃用户数达 1.325 亿。可以说，京东 618 - 支撑万亿交易订单的就是 k8s 容器云平台。

天猫、知乎、永辉超市、诺基亚、阿里巴巴等都在基于 Kubernetes 搭建智能化的容器云管理平台，管理数万个 K8S 集群，其中最大的集群约 1 万个物理节点，每个集群会运行几十万个 pod 应用。

那么，容器在中国的使用率和引用的技术[架构](https://so.csdn.net/so/search?q=%E6%9E%B6%E6%9E%84&spm=1001.2101.3001.7020)又有哪些呢？话不多说，直接上图：  
![](https://img-blog.csdnimg.cn/20210719143451729.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1ZXNoZW5sYWlsYQ==,size_16,color_FFFFFF,t_70)  
![](https://img-blog.csdnimg.cn/20210719143508610.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1ZXNoZW5sYWlsYQ==,size_16,color_FFFFFF,t_70)  
![](https://img-blog.csdnimg.cn/20210719143528747.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1ZXNoZW5sYWlsYQ==,size_16,color_FFFFFF,t_70)  
![](https://img-blog.csdnimg.cn/20210719143553642.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1ZXNoZW5sYWlsYQ==,size_16,color_FFFFFF,t_70)  
![](https://img-blog.csdnimg.cn/20210719143603308.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1ZXNoZW5sYWlsYQ==,size_16,color_FFFFFF,t_70)  
想要提升薪资，必须学习微服务！

大数据，人工智能，高并发等相关技术。这些领域中的具体框架，如：SpringCloud ， Dubbo ，TensorFlow ，Kafka，Hadoop，Spark，ELK 等，都需要掌握 K8S 相关技术，才可以在生成环境下大规模使用，无论是运维、开发、大数据、人工智能，学习 k8s 是未来能力和职位晋升必备的技术。

那么，很多小伙伴肯定想知道，k8s 需要掌握哪些技术呢？莫慌莫慌，微容器架构师学习路线分享给你：  
![](https://img-blog.csdnimg.cn/20210719143636297.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1ZXNoZW5sYWlsYQ==,size_16,color_FFFFFF,t_70)  
![](https://img-blog.csdnimg.cn/20210719143711561.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h1ZXNoZW5sYWlsYQ==,size_16,color_FFFFFF,t_70)  
需要 K8s 学习路线图、名企架构图、及以上微容器架构师 K8s 技能树内容的同学，扫描下方二维码，回复 k8s 领取!  
![](https://img-blog.csdnimg.cn/img_convert/3126d6a111203824842871d5ebdcff3b.png#pic_center)