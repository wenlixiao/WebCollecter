> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/huanyue9987/p/15843998.html)

非商业转载，来自简书 - 虞大胆的叽叽喳喳 - 杰作的：[在 Python 中安装包的三种方法 - 简书](https://www.jianshu.com/p/c6055e8873ee "在Python中安装包的三种方法 - 简书")

最近一段时间都在学习 Python3（如果你想部署 Python3 的开发环境，可参考《是时候配置一个 Python3 的开发环境了》），乘此机会重新回顾了 Python2 的相关知识，在 Python 中，如果想引入第三方包和库，可以通过工具安装，那么这些安装工具背后做了什么是我非常关心的，这篇本文解释了相关知识：

*   Python 有多少种类型的包（Python 历史实在太悠久了）。
*   Python 包安装工具有哪些。
*   安装第三方包后，生成了哪些文件。
*   如何将 Python 代码打包成一个包（要基于 Python 包管理工具），该主题不是本文重点。

下面这张图简单解释了包之间的关系：

![](https://img-blog.csdnimg.cn/ee717034c44140889ed477bdf383a733.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5qCR5LiL5rC05pyI,size_18,color_FFFFFF,t_70,g_se,x_16)

结构关系

*   开发者开发包需要遵循标准，然后发布到 Pypi 中，下一篇文章会描述。
*   包使用者可以使用多种工具从 Pypi 中下载包（本文的重点）。
*   Pypi 包含 Meta 信息和源代码仓库。

在我学习 PHP 的时候，没有一种很好的包安装工具（现在可以使用 Composer），而 Python 在标准化方面做的更好。

再一次**申明**：

*   本文没有在 Python 3 环境下测试。
*   本文知识点可能陈旧，比如 Python Pypi 官方已做了很大改变。

### python 有多种类型的包工具

*   Distutils：Python 标准的包管理工具，扩展性不够。
*   Setuptools：比 Distutils 提供了更多的功能，虽然不是官方的，但却是事实上的标准。
*   Distribute：是 Setuptools 的一个分支，目前已经退出了历史舞台。
*   Distutils2：又一个被废弃的标准。

这些工具本来就是 Python 的一个包，如果开发者想编写、发布一个包，必须基于这些包进行开发。

### 三种类型的包

*   tar.gz
*   egg
*   whell

开发者可以基于 Distutils 或 Setuptools 生成这三种类型的包。

### 安装包

（1）源码安装

可以手动下载第三方包，然后手动安装。

```
# wget "https://files.pythonhosted.org/packages/96/66/43e6df87373557553be2b4343db27d008c6dcefa110ccff38cba1459ca07/ywdblogmath-0.1.tar.gz"
# tar xvf ywdblogmath-0.1.tar.gz 
# cd ywdblogmath-0.1/ 
# python setup.py install 

```

安装或更新文件如下：

*   /usr/local/lib/python2.7/dist-packages/easy-install.pth
*   /usr/local/lib/python2.7/dist-packages/ywdblogmath-0.1-py2.7.egg

某些被安装的包可能包含 C 代码，所以需要 gcc 这样的工具编译。

（2）easy_install

如果想要使用 easy_install 安装第三方包，需要先安装 setuptools，如果本机没有安装，可以采用源码方式安装，比如：

```
https://pypi.org/project/setuptools/ 下载 .zip 包
# python setup.py  install

```

easy_install 支持从 Pypi（tar.gz 或 egg 包）、URL、本地目录安装软件包：

```
# 从 Pypi 安装最新的包，可能是 tar.gz 或 egg 包
$ easy_install  ywdblogmath 

# 安装 tar.gz 类型的包
$ easy_install "https://files.pythonhosted.org/packages/96/66/43e6df87373557553be2b4343db27d008c6dcefa110ccff38cba1459ca07/ywdblogmath-0.1.tar.gz" 

# 安装 egg 包
# easy_install "https://files.pythonhosted.org/packages/b0/fe/1fef363672c1e179de61ff1519aed6a3d68200b4cad0536b6d96b08cc5e9/ywdblogmath-0.3-py2.7.egg" 

# 本地目录包含 ywdblogmath 的源码 
$ easy_install /root/python

```

如果安装的是一个 tar.gz 或 egg 的压缩包，安装后会出现相关文件，如下：

*   /usr/local/lib/python2.7/dist-packages/easy-install.pth（文件更新）
*   /usr/local/lib/python2.7/dist-packages/ywdblogmath-0.1-py2.7.egg（文件新增）

特别说明：

*   easy_install 只能安装包，不能卸载包。
*   easy_install 不能安装 wheel 格式的包（pip 可以，但 pip 不能安装 egg 格式的包）

（3）pip

如果本机没有安装 pip，可以使用 easy_install 工具安装。

pip 常用命令：

```
# pip list 
# pip install ywdblogmath 
# pip install ywdblogmath -U 
# pip show ywdblogmath 

```

安装 tar.gz 包：

```
$ pip install "https://files.pythonhosted.org/packages/96/66/43e6df87373557553be2b4343db27d008c6dcefa110ccff38cba1459ca07/ywdblogmath-0.1.tar.gz"  


```

安装后会出现相关文件，如下：

*   /usr/local/lib/python2.7/dist-packages/ywdblogmath（新增目录）
*   /usr/local/lib/python2.7/dist-packages/ywdblogmath-0.1-py2.7.egg-info（新增文件）

安装 wheel 包：

```
$ pip install "https://files.pythonhosted.org/packages/5f/ca/6624a4b42be2df78f51043d2282944e78dc939066a3da07dfdb949cd6d3e/ywdblogmath-0.4-py2-none-any.whl"


```

安装后会出现相关文件，如下：

*   /usr/local/lib/python2.7/dist-packages/ywdblogmath （新增目录）
*   /usr/local/lib/python2.7/dist-packages/ywdblogmath-0.4.dist-info（新增文件）