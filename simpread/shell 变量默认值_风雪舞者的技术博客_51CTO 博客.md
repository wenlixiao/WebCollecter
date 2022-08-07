> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.51cto.com](https://blog.51cto.com/happytree007/1900383?abTest=51cto)

> shell 变量默认值，一、环境 & nbsp;   ubuntu14.04x86_64    二、变量默认值 & nbsp;  2......

© 著作权归作者所有：来自 51CTO 博客作者 happytree007 的原创作品，请联系作者获取转载授权，否则将追究法律责任  
[shell 变量默认值](https://blog.51cto.com/happytree007/1900383)  
[https://blog.51cto.com/happytree007/1900383](https://blog.51cto.com/happytree007/1900383)

**一、环境**

    ubuntu14.04 x86_64

**二、变量默认值**

    2.1     **${vari:-defaultValue}**

    当 var 没有定义时，此时使用 defaultValue, 而 vari 依然为空，没有改变值

     eg:  
      在终端上操作

    2.2    **${vari:=defaultValue}**

     当 vari 没有定义时，此时使用 defaultValue, 同时 vari 也被赋值为 defaultValue

      eg:

    在终端上操作

    2.3 **${vari:?value}**

         当 vari 没有定义时，或者定义了值为空，将在终端报错并且退出，用于检查是否定义以及是否为空

        eg.

        2.4 **${vari:+value}**

          当 vari 定义并且不为空，将用 value 替换 vari 的值，否则什么也不做， 与 ${vari:-value} 相反

         eg:

 **三、应用**

    3.1 Makefile

             eg: linux 内核中其中一个 Makefile 中的     

    3.2 函数默认参数  

        和 c++ 的默认参数异曲同工之妙

        default_parameters.cpp

    #g++ default_parameters.cpp

    # ./a.out  
    1, 2, 3  
    10, 2, 3  
    10, 99.9, 3  
    20, 33.3, 9.3

    default_parameters.sh

    #bash default_parameters.sh

    10, 10.2, zhangsan  
    23, 10.2, zhangsan  
    23, 12.0, zhangsan  
    34, 23.0, lisi        

  
        这样就可以让函数呈现多态性  

举报文章

请选择举报类型

内容侵权 涉嫌营销 内容抄袭 违法信息 其他

上传截图

格式支持 JPEG/PNG/JPG，图片不超过 1.9M

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

已经收到您得举报信息，我们会尽快审核

相关文章
----