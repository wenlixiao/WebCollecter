> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_67479400/article/details/125031007)

**目录**

[一、Sed](#t0)

[1. Sed 概述](#t1)

[2. Sed 工作流程](#t2)

[3. 基本用法](#t3)

[4. 常用选项](#t4)

[5. Sed 命令的常用参数](#t5)

[二. Sed 命令使用](#t6)

[1. 输出指定行](#t7)

[1.1 使用 Sed 命令输出指定行](#t8)

[1.2 结合正则表达式输出指定行](#t9)

[2. 插入符合条件的行](#t10)

[3. 删除符合条件的行](#t11)

[4. 替换符合条件的文本](#t12)

[4.1 字符串替换](#t13)

[5. 迁移符合条件的文本](#t14)

[6. 使用脚本编辑文件](#t15)

一、Sed
=====

1. Sed 概述
---------

sed 编辑器时一种流编辑器，流编辑器会在编辑器处理数据之前基于预先提供的一组规则来编辑数据流

sed 编辑器可以根据命令来处理数据流中的数据，这些命令要么从命令行中输入，要存储在一个命令文本文件中

2. Sed 工作流程
-----------

sed 的工作流程主要包括读取、执行和显示三个过程：

*   读取：sed 从输入流（文件、管道、标准输入）中读取一行内容并存储到临时的缓冲区中（又称模式空间，pattern space）。
*   执行：默认情况下，所有的 sed 命令都在模式空间中顺序地执行，除非指定了行的地址，否则 sed 命令 将会在所有的行上依次执行。
*   显示：发送修改后的内容到输出流。在发送数据后，模式空间将会被清空

默认情况下所有的 sed 命令都是在模式空间内执行的，因此输入的文件并不会发生任何变化，除非是用重定向存储输出

3. 基本用法
-------

```
sed -e '操作' 文件1 文件2
 
sed -n -e '操作' 文件1 文件2 
 
sed -f 脚本文件 文件1 文件2 
 
sed -i -e '操作' 文件1 文件2
```

4. 常用选项
-------

```
-e 或 - -expression=∶ 多点编辑
 
-f 或- -file=∶表示用指定的脚本文件来处理输入的文本文件。
 
-h 或- -help∶显示帮助。
 
-n∶ 不输出模式空间内容到屏幕，即不自动打印，加p，又恢复自动打印
 
-i∶ 备份文件文件并原处编辑
 
-r：使用扩展正则表达式
```

5. Sed 命令的常用参数
--------------

<table border="1" cellpadding="1" cellspacing="1"><thead><tr><th>选项</th><th>说明</th></tr></thead><tbody><tr><td><strong>s</strong></td><td>替换，替换指定字符</td></tr><tr><td><strong>d</strong></td><td>删除，删除选定的行</td></tr><tr><td><strong>a</strong></td><td>增加，在当前行下面增加一行指定内容</td></tr><tr><td><strong>i</strong></td><td>插入，在选定行上面插入一行指定内容</td></tr><tr><td><strong>c</strong></td><td>替换，将选定行替换为指定内容</td></tr><tr><td><strong>Y</strong></td><td>字符转换，转换前后的字符长度必须相同</td></tr><tr><td><strong>P</strong></td><td>打印，如果同时指定行，表示打印指定行; 如果不指定行，则表示打印所有内容; 如果有非打印字符，则以 AscII 码输出。其通常与_n" 选项一起使用</td></tr><tr><td><strong>=</strong></td><td>打印行号</td></tr><tr><td><p><strong>l (小写 L)</strong></p></td><td>打印数据流中的文本和不可打印的 ASCII 字符（比如结束符 s、制表符 \ t）</td></tr></tbody></table>

<table border="1" cellpadding="1" cellspacing="1"><thead><tr><th>选项</th><th>参数</th></tr></thead><tbody><tr><td>H</td><td>复制到剪贴板</td></tr><tr><td>g、G</td><td>将剪贴板中的数据覆盖 / 追加至指定行</td></tr><tr><td>w</td><td>保存为文件</td></tr><tr><td>r</td><td>读取指定文件</td></tr><tr><td>a</td><td>追加指定内容。具体操作方法如下所示</td></tr><tr><td>I,i</td><td>忽略大小写</td></tr></tbody></table>

二. Sed 命令使用
===========

1. 输出指定行
--------

### 1.1 使用 Sed 命令输出指定行

```
sed -n 'p' passwd.txt         ##输出所有内容,效果等同于 cat test.txt
 
sed -n '3p' passwd.txt	      ##输出第 3 行
 
sed -n '3,5p' passwd.txt	  ##输出 3~5 行
 
sed -n 'p;n' passwd.txt	      ##输出所有奇数行,n 表示读入下一行资料
 
sed -n 'n;p' passwd.txt	      ##输出所有偶数行,n 表示读入下一行资料
```

![](https://img-blog.csdnimg.cn/6e5a995fa0a647b1ba55fc7b473e4dba.png)

```
sed -n '1,5{p;n}' passwd.txt    ##输出第 1~5 行之间的奇数行(第 1、3、5 行)
 
sed -n '10,${n;p}' passwd.txt	##输出第 10 行至文件尾之间的偶数行
 
sed -n '2,+3p' passwd.txt       ##从第2行开始，连续3行进行输出，即输出2~5行
```

![](https://img-blog.csdnimg.cn/0988e735671040f5a20248ba30df074b.png)

### 1.2 结合[正则表达式](https://so.csdn.net/so/search?q=%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F&spm=1001.2101.3001.7020)输出指定行

sed 命令结合正则表达式时，格式略有不同，正则表达式以 “/” 包围

如果遇到特殊符号的情况，扩展正则还需要转义字符 “\”

![](https://img-blog.csdnimg.cn/a7c06224eff34ebfbae5ad786fc90c43.png)

```
sed -n '/the/p' text.txt       	//输出包含the 的行
 
sed -n '2,/the/p' text.txt	    //输出从第 4 行至第一个包含 the 的行
 
sed -n '/the/=' text.txt        //输出包含the 的行所在的行号,等号(=)用来输出行号
 
sed -n '/^PI/p' text.txt	    //输出以PI 开头的行
 
sed -n '/[0-9]$/p' text.txt     //输出以数字结尾的行
 
sed -n '/\<wood\>/p' text.txt   //输出包含单词wood 的行
```

![](https://img-blog.csdnimg.cn/e15deb02e61c4ffc85c9624fcfd93848.png)

2. 插入符合条件的行
-----------

使用插入时，如果添加多行数据，除最后一行外，每行末尾都需要用 “\n” 符号表示数据未完结，换行

```
sed '/the/i 稻香' text.txt    //在含有the行的前面一行添加稻香
 
sed '/the/a 七里香' text.txt  //在含有the行的下一行添加七里香
 
sed '3a晴天' text.txt         //在第3行之后插入晴天（字符）
```

**i 表示在当前行的前面一行添加**

![](https://img-blog.csdnimg.cn/6d2f708b3be54080b0e0f74060b1d2e8.png)

**a 表示在当前行的后面一行添加**

![](https://img-blog.csdnimg.cn/e02fab1bbc9f402685fbb923d72d5041.png)

![](https://img-blog.csdnimg.cn/f6e97837475e46f89ab583ff87da2876.png)

3. 删除符合条件的行
-----------

nl 命令用于计算文件的行数，结合该命令可以更加直观地查看到命令执行的结果

```
nl text.txt | sed '3d'        //删除第3行
 
nl text.txt | sed '3,5d'      //删除第3~5行
 
nl text.txt |sed '/cross/d'   //删除包含 cross 的行,原本的第 8 行被删除
 
nl text.txt |sed '/cross/!d'  //删除不包含 cross 的行,用!符号表示取反操作
 
sed '/^[a-z]/d' text.txt      //删除以小写字母开头的行
 
sed '/\.$/d' text.txt         //删除以.结尾的行
```

![](https://img-blog.csdnimg.cn/a3ccbcdd7c054788b9d215a2d071231d.png)

![](https://img-blog.csdnimg.cn/47b3a9365c454f76882472612ffb2310.png)

![](https://img-blog.csdnimg.cn/3cf992c5f6bb4a9a8fefef8e1773dd2e.png)

4. 替换符合条件的文本
------------

### 4.1 [字符串](https://so.csdn.net/so/search?q=%E5%AD%97%E7%AC%A6%E4%B8%B2&spm=1001.2101.3001.7020)替换

```
sed 's/the/THE/' text.txt	   //将每行中的第一个the 替换为 THE
 
sed 's/l/L/2' text.txt	       //将每行中的第 2 个 l 替换为 L
 
sed 's/the/THE/g' text.txt	   //将文件中的所有the 替换为 THE
 
sed 's/o//g' text.txt    	   //将文件中的所有o 删除(替换为空串)
 
sed 's/^/#/' text.txt	       //在每行行首插入#号
 
sed '/the/s/^/#/' text.txt	   //在包含the 的每行行首插入#号
 
sed '3,5s/the/THE/g' text.txt  //将第 3~5 行中的所有 the 替换为 THE
 
sed '/the/s/o/O/g' text.txt	   //将包含the 的所有行中的 o 都替换为 O
```

![](https://img-blog.csdnimg.cn/328e94919f8948d084f0bbb77c872f6f.png)

![](https://img-blog.csdnimg.cn/dbc1433c3f204b1ab44bf15c93b2fde7.png)

![](https://img-blog.csdnimg.cn/128d150a906b472289078d898a536f84.png)

![](https://img-blog.csdnimg.cn/243ab0a6eec841ca8bbe3c4be57ca4f6.png)

4.2 先备份再修改数据

![](https://img-blog.csdnimg.cn/ed5d6ef385044841bbe014a714e20398.png)

5. 迁移符合条件的文本
------------

```
sed '/the/{H;d};$G' text.txt	     //将包含the 的行迁移至文件末尾,{;}用于多个操作
 
sed '1,5{H;d};14G' text.txt	         //将第 1~5 行内容转移至第 17 行后
 
sed '/the/w out.file' text.txt	     //将包含the 的行另存为文件 out.file
 
sed '/the/r /etc/hostname' text.txt	 //将文件/etc/hostname 的内容添加到包含 the 的每行以后
 
sed '3aNew' text.txt	             //在第 3 行后插入一个新行,内容为New
 
sed '/the/aNew' text.txt	         //在包含the 的每行后插入一个新行,内容为 New
 
sed '3aNew1\nNew2' text.txt	         //在第 3 行后插入多行内容,中间的\n 表示换行
```

![](https://img-blog.csdnimg.cn/d3b2dfe46dcb48038a1c97e9fa8309e1.png)

![](https://img-blog.csdnimg.cn/cfda73755d9949448d54d3cfc9f01cbe.png)

![](https://img-blog.csdnimg.cn/b458748890284a2697f69348a2655ea6.png)

![](https://img-blog.csdnimg.cn/3b66bc7b45a2403f8e9058569b78b2bd.png)

![](https://img-blog.csdnimg.cn/c9e15e05f4fc404f98155e1ffe079821.png)

![](https://img-blog.csdnimg.cn/df22a2a765054d719512f4979307613c.png)

6. 使用脚本编辑文件
-----------

使用 sed 脚本将多个编辑指令存放到文件中（每行一条编辑指令），通过 “-f” 选项来调用

例如执行以下命令即可将第 1~5 行内容转移至第 14 行后，等同于 sed ‘1,5{H;d};16G’ text.txt

![](https://img-blog.csdnimg.cn/b82ce38efd1c471c9ee63d6c310f4fa1.png)