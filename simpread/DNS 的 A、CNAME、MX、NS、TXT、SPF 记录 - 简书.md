> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/b483300378af)

前言
==

最近工作过程中需要设定邮件服务器，其中涉及到 dns 服务器的设定。  
整理并且记录自己的理解。

种类
==

A、CNAME、MX、NS、TXT、SPF  
下面挨个介绍一下。

### A 记录 / AAAA 记录

##### IPv4：

*   示例：ns1.exmaple.com. IN A 198.51.100.2
*   解释：【domain】 IN A 【IP 地址】

##### IPv6：

*   示例：ns1.exmaple.com. IN AAAA 8fe0::8f61:ac8:30cd:a16e
*   解释：【domain】 IN AAAA 【IP 地址】  
    ※IN 的意思是「Internet」，不是 IN/OUT 的「IN」。

##### 干什么用呢？

我们在浏览器输入域名后，需要向 DNS 服务器请求，找到这个域名对应的服务器 IP。上面示例就是这么一条记录。  
虽然域名和 IP 都可以变更，但是相比来说域名变更更加简单和随意。所以当网站更换自己域名的时候，就需要修改这条记录。

### CNAME

*   示例：sub.example.com. IN CNAME hoge.example.com.
*   解释：【別名】 IN CNAME 【原名】

##### 干什么用呢？

给某一个 domain 起多个名字。  
类似于，jd.com,jd360.com,jingdong.com 虽然是不同名字的域名，但是可以指向同一个原名 jd.com。可以让企业的对外展示更加灵活。  
举例：  
jd360.com IN CNAME jd.com  
jingdong.com IN CNAME jd.com  
jd.com IN A 123.123.123.123 （这条是 A 记录例子）

### MX 记录

*   MX 记录（Mail Exchange）：邮件路由记录  
    在 DNS 上设定，用于将邮箱地址 @符号后的域名指向邮件服务器。
*   示例：example.com. IN MX 10 mail.example.com.
*   解释：【domain】 IN MX 【优先度】 【邮件服务器】

##### 干什么用呢？

当发信侧服务器给受信侧发邮件时，首先会要求 DNS 服务器解析受信侧邮箱地址中 @后面部分的域名对应的 MX 记录（DNS 的写法可以理解成 [example.com](https://links.jianshu.com/go?to=http%3A%2F%2Fexample.com) 的 A 记录下面，有一行上面示例的 MX 记录，当然邮箱服务器也有对应的 A 记录）。  
这样，邮件就直接发到对应的 MX 记录的 A 记录里的 IP 了。  
例子：给 [test@exmaple.com](https://links.jianshu.com/go?to=mailto%3Atest%40exmaple.com) 发邮件的话，  
DNS 会返回给发信侧 198.51.100.3 这个 IP

> exmaple.com. IN A 198.51.100.2  
> example.com. IN MX 10 mail.example.com.  
> mail.example.com. IN A 198.51.100.3

※如果是普通用户通过【exmaple.com】浏览主页，那么 DNS 继续返回 198.51.100.2 。这个其实也需要 DNS 判断请求服务器是邮件服务器还是普通的访问。

### NS 记录

*   指定域名解析服务器。
*   示例：example.com. IN NS ns1.example.com.
*   解释：【domain】 IN NS 【DNS 服务器】

##### 干什么用呢？

指定该域名由哪个 DNS 服务器来进行解析。

### TXT 记录

*   示例：ns1.exmaple.com. IN TXT "联系电话：XXXX"
*   解释：【domain】 IN TXT 【任意字符串】

##### 干什么用呢？

一般指某个主机名或域名的说明，或者联系方式，或者标注提醒等等。

### SPF 记录

SPF 记录是 TXT 记录的一个运用。后面的备注需要按照指定的格式才能有效。

*   示例：exmaple.com. IN TXT "v=spf1 ip4:198.51.100.1 ~all"
*   解释：【domain】 IN TXT 【送信侧邮件服务器确认规则】

##### 干什么用呢？

从发信侧服务器设定到 DNS 上的这条记录中，读取信息，判断发信侧是否合法。  
如果不符合规则，那么按照约定的规则处理掉。  
跟 MX 记录正好相反。  
MX：我是收件服务器，你找我时，请参考我设定到 DNS 服务器上的 MX 记录。  
SPF：我是发信服务器，你接受邮件时，请参考我设定到 DNS 服务器上 SPF 规则。如果不是我发的信，你可以删掉或者接收。

##### SPF 记录规则

1.  格式：  
    版本 空格 定义 空格 定义 （空格 定义的循环）  
    跟着例子看的话，比较好理解。  
    example.com. IN SPF "v=spf1 ip4:192.0.2.1 -all"
    
    *   v=spf1 是版本。只出现一次。
    *   ip4:192.0.2.1 第一个定义
    *   -all 第二个定义
2.  定义的格式。
    
    *   种类  
        | all | ip4 | ip6 | a | mx | ptr | exists | include|
    *   前缀  
        "+" Pass（通过）  
        "-" Fail（拒绝）  
        "~" Soft Fail（软拒绝）  
        "?" Neutral（中立）
3.  定义测试  
    测试时，将从前往后依次测试每个定义。  
    如果一个定义命中了要查询的 IP 地址，则由相应定义的前缀决定怎么处理。默认的前缀为 +。  
    如果测试完所有的 定义也没有命中，则结果为 Neutral。  
    结果及处理方法一览
    

<table><thead><tr><th>结果</th><th>说明</th><th>服务器处理办法</th></tr></thead><tbody><tr><td>Pass</td><td>发件 IP 是合法的</td><td>接受来信</td></tr><tr><td>Fail</td><td>发件 IP 是非法的</td><td>退信</td></tr><tr><td>Soft Fail</td><td>发件 IP 非法，但是不采取强硬措施</td><td>接受来信，但是做标记</td></tr><tr><td>Neutral</td><td>SPF 记录中没有关于发件 IP 是否合法的信息</td><td>接受来信</td></tr><tr><td>None</td><td>服务器没有设定 SPF 记录</td><td>接受来信</td></tr><tr><td>PermError</td><td>发生了严重错误（例如 SPF 记录语法错误）</td><td>没有规定</td></tr><tr><td>TempError</td><td>发生了临时错误（例如 DNS 查询失败）</td><td>接受或拒绝</td></tr></tbody></table>

后记
==

有一个问题，调查后更新。  
设定好 SPF 的记录之后怎么快速测试呢？  
总不能每次都等 DNS 更新完再测试吧？

20210914 更新：  
SPF 测试方法很多，google 一下的话可以找到很多网站。  
我常用的是 [https://mxtoolbox.com/spf.aspx](https://links.jianshu.com/go?to=https%3A%2F%2Fmxtoolbox.com%2Fspf.aspx)