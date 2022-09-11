> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/u_1716127/3219292)

> cut 命令详解，1.cut 作用 cut 命令可以从一个文本文件或者文本流中提取文本列。

© 著作权归作者所有：来自 51CTO 博客作者 chaoren399 的原创作品，请联系作者获取转载授权，否则将追究法律责任

1.cut 作用
--------

cut 命令可以从一个文本文件或者文本流中提取文本列。

2.cut 语法
--------

 cut -d'分隔字符' -f fields  (用于有特定分隔字符)  
 cut -c 字符区间  (用于排列整齐的信息)  
选项与参数：  
　　-d ：后面接分隔字符。与 -f 一起使用；  
　　-f ：依据 -d 的分隔字符将一段信息分割成为数段，用 -f 取出第几段的意思；  
　　-c ：以字符 (characters) 的单位取出固定字符区间；

3.cut 例子
--------

### 3.1 将 PATH 变量取出，找出第五个路径

```
/zzy/zookeeper-standlone/bin:/zzy/hbase-0.98.8-hadoop2/bin:/zzy/hadoop-2.6.0/bin:/zzy/hadoop-2.6.0/sbin:
/usr/local/java/jdk1.7.0_79/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
1.
2.

```

命令:

```
echo $PATH | cut -d ':' -f 5    (-d ：后面接分隔字符。与 -f 一起使用；)
1.

```

 结果:

![](https://s2.51cto.com/images/blog/202107/29/f8f9de7668f7b1ef479551f2b04c1049.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 3.2 将 PATH 变量取出, 找出第三和第五个路径

```
echo $PATH | cut -d ':' -f 3,5
1.

```

![](https://s2.51cto.com/images/blog/202107/29/ab3aac08f77ec2ddd24d8910a96fc4da.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 3.3 将 PATH 变量取出，找出第三到最后一个路径

```
echo $PATH | cut -d ':' -f 3-
1.

```

![](https://s2.51cto.com/images/blog/202107/29/297eebbf9e5d0a58c570f460239f3825.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 3.4 将 PATH 变量取出，找出第一到第三个路径

```
echo $PATH | cut -d ':' -f 1-3
1.

```

### 3.5 将 PATH 变量取出，找出第一到第三，还有第五个路径

```
echo $PATH | cut -d ':' -f 1-3,5
1.

```

4. 实用例子: 只显示 / etc/passwd 的用户和 shell
------------------------------------

```
cat /etc/passwd | cut -d ':' -f 1,7 
1.

```

![](https://s2.51cto.com/images/blog/202107/29/191883cf1b1dce745ff68aaaac260130.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

岁月里，寒暑交替。人世间，北来南往。铭心的，云烟的。都付往事，不念，不问。

举报文章

请选择举报类型

内容侵权 涉嫌营销 内容抄袭 违法信息 其他

上传截图

格式支持 JPEG/PNG/JPG，图片不超过 1.9M

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

已经收到您得举报信息，我们会尽快审核

**相关文章**