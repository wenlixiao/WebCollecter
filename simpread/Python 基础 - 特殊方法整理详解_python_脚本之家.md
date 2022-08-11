> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jb51.net](https://www.jb51.net/article/224482.htm)

> python 中特殊方法（魔术方法）是被 python 解释器调用的，我们自己不需要调用它们，我们统一使用内置函数来使用。本篇文章将对其详细介绍，感兴趣的小伙伴可以参考下面文章的具体内容

python 中特殊方法（魔术方法）是被 python 解释器调用的，我们自己不需要调用它们，我们统一使用内置函数来使用。本篇文章将对其详细介绍，感兴趣的小伙伴可以参考下面文章的具体内容

1、概述
----

python 中特殊方法（魔术方法）是被 python 解释器调用的，我们自己不需要调用它们，我们统一使用内置函数来使用。例如：特殊方法`__len__()`实现后，我们只需使用`len()`方法即可；也有一些特殊方法的调用是隐式的，例如：`for i in x`: 背后其实用的是内置函数 iter(x)。

下面将介绍一些常用特殊方法和实现。通过实现一个类来说明

2、常用特殊方法及实现
-----------

### 2.1 _len__()

一般返回数量，使用`len()`方法调用。在`__len__()`内部也可使用`len()`函数

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p></td><td><p><code>class</code> <code>Zarten():</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__init__(</code><code>self</code><code>, age):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.age </code><code>=</code> <code>age</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.brother </code><code>=</code> <code>[</code><code>'zarten_1'</code><code>, </code><code>'zarten_2'</code><code>]</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__len__(</code><code>self</code><code>):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code> <code>len</code><code>(</code><code>self</code><code>.brother)</code></p><p><code>z </code><code>=</code> <code>Zarten(</code><code>18</code><code>)</code></p><p><code>print</code><code>(</code><code>len</code><code>(z))</code></p></td></tr></tbody></table>

### 2.2 __str__()

对象的字符串表现形式，与`__repr__()`基本一样，**微小差别在于：**

*   `__str__()`用于给终端用户看的，而`__repr__()`用于给开发者看的，用于调试和记录日志等。
*   在命令行下，实现`__str_()后`，直接输入对象名称会显示对象内存地址；而实现`__repr__()`后，跟`print`（对象）效果一样。
*   若这 2 个都实现，会调用`__str_()，`一般在类中至少实现`__repr__()`

![](https://pic3.zhimg.com/80/v2-1670e4576044d479bc9ff78d97df6796_720w.jpg)

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p></td><td><p><code>class</code> <code>Zarten():</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__repr__(</code><code>self</code><code>):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code> <code>'my name is Zarten_1'</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__str__(</code><code>self</code><code>):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code> <code>'my name is Zarten_2'</code></p><p><code>z </code><code>=</code> <code>Zarten()</code></p><p><code>print</code><code>(z)</code></p></td></tr></tbody></table>

### 2.3 __iter__()

返回一个可迭代对象，一般跟`__next__()`一起使用

**判断一个对象是否是：**可迭代对象 `from collections import Iterable`

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p></td><td><p><code>from</code> <code>collections </code><code>import</code> <code>Iterable</code></p><p><code>class</code> <code>Zarten():</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__init__(</code><code>self</code><code>, brother_num):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.brother_num </code><code>=</code> <code>brother_num</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.count </code><code>=</code> <code>0</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__iter__(</code><code>self</code><code>):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code> <code>self</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__next__(</code><code>self</code><code>):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>if</code> <code>self</code><code>.count &gt;</code><code>=</code> <code>self</code><code>.brother_num:</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>raise</code> <code>StopIteration</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>else</code><code>:</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.count </code><code>+</code><code>=</code> <code>1</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code> <code>'zarten_'</code> <code>+</code> <code>str</code><code>(</code><code>self</code><code>.count)</code></p><p><code>zarten </code><code>=</code> <code>Zarten(</code><code>5</code><code>)</code></p><p><code>print</code><code>(</code><code>'is iterable:'</code><code>, </code><code>isinstance</code><code>(zarten, Iterable))</code></p><p><code>for</code> <code>i </code><code>in</code> <code>zarten:</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>print</code><code>(i)</code></p></td></tr></tbody></table>

### 2.4 __getitem__()

此特殊方法返回数据，也可以替代`__iter_()`和`__next__()`方法，也可支持切片

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p></td><td><p><code>class</code> <code>Zarten():</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__init__(</code><code>self</code><code>):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.brother </code><code>=</code> <code>[</code><code>'zarten_1'</code><code>,</code><code>'zarten_2'</code><code>,</code><code>'zarten_3'</code><code>,</code><code>'zarten_4'</code><code>,</code><code>'zarten_5'</code><code>,]</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__getitem__(</code><code>self</code><code>, item):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code> <code>self</code><code>.brother[item]</code></p><p><code>zarten </code><code>=</code> <code>Zarten()</code></p><p><code>print</code><code>(zarten[</code><code>2</code><code>])</code></p><p><code>print</code><code>(zarten[</code><code>1</code><code>:</code><code>3</code><code>])</code></p><p><code>for</code> <code>i </code><code>in</code> <code>zarten:</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>print</code><code>(i)</code></p></td></tr></tbody></table>

### 2.5 __new__()

`__new__()`用来构造一个类的实例，第一个参数是`cls`，一般情况下不会使用。而`__init__()`用来初始化实例，所以`__new__()`比`__init___()`先执行。

若`__new__()`不返回，则不会有任何对象创建`，__init___()`也不会执行；

![](https://img.jbzj.com/file_images/article/202104/2021416141955635.png)

若`__new__()`返回别的类的实例，则`__init___()`也不会执行；

**用途：**可使用`__new___()`实现单例模式

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p></td><td><p><code>class</code> <code>Zarten():</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__new__(</code><code>cls</code><code>, </code><code>*</code><code>args, </code><code>*</code><code>*</code><code>kwargs):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>print</code><code>(</code><code>'__new__'</code><code>)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code> <code>super</code><code>().__new__(</code><code>cls</code><code>)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__init__(</code><code>self</code><code>, name, age):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>print</code><code>(</code><code>'__init__'</code><code>)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.name </code><code>=</code> <code>name</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.age </code><code>=</code> <code>age</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__repr__(</code><code>self</code><code>):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code> <code>'name: %s&nbsp; age:%d'</code> <code>%</code> <code>(</code><code>self</code><code>.name,</code><code>self</code><code>.age)</code></p><p><code>zarten </code><code>=</code> <code>Zarten(</code><code>'zarten'</code><code>, </code><code>18</code><code>)</code></p><p><code>print</code><code>(zarten)</code></p></td></tr></tbody></table>

![](https://pic3.zhimg.com/80/v2-34965d9030f70e98897c07b07df2f8ba_720w.jpg)

### 2.6 使用__new__() 实现单例模式

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p><p>17</p><p>18</p><p>19</p><p>20</p><p>21</p><p>22</p></td><td><p><code>class</code> <code>Zarten():</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>_singleton </code><code>=</code> <code>None</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__new__(</code><code>cls</code><code>, </code><code>*</code><code>args, </code><code>*</code><code>*</code><code>kwargs):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>print</code><code>(</code><code>'__new__'</code><code>)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>if</code> <code>not</code> <code>cls</code><code>._singleton:</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>cls</code><code>._singleton </code><code>=</code> <code>super</code><code>().__new__(</code><code>cls</code><code>)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code> <code>cls</code><code>._singleton</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__init__(</code><code>self</code><code>, name, age):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>print</code><code>(</code><code>'__init__'</code><code>)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.name </code><code>=</code> <code>name</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.age </code><code>=</code> <code>age</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__repr__(</code><code>self</code><code>):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code> <code>'name: %s&nbsp; age:%d'</code> <code>%</code> <code>(</code><code>self</code><code>.name,</code><code>self</code><code>.age)</code></p><p><code>zarten </code><code>=</code> <code>Zarten(</code><code>'zarten'</code><code>, </code><code>18</code><code>)</code></p><p><code>zarten_1 </code><code>=</code> <code>Zarten(</code><code>'zarten_1'</code><code>, </code><code>19</code><code>)</code></p><p><code>print</code><code>(zarten)</code></p><p><code>print</code><code>(zarten_1)</code></p><p><code>print</code><code>(zarten_1 </code><code>=</code><code>=</code> <code>zarten)</code></p></td></tr></tbody></table>

![](https://pic4.zhimg.com/80/v2-4e42f4ca269ed1abd483c6c55a7bef7f_720w.jpg)

### 2.7 __call__()

实现后对象可变成可调用对象，此对象可以像函数一样调用，例如：自定义函数，内置函数，类都是可调用对象，可用`callable()`判断是否是可调用对象

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p></td><td><p><code>class</code> <code>Zarten():</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__init__(</code><code>self</code><code>, name, age):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.name </code><code>=</code> <code>name</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.age </code><code>=</code> <code>age</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__call__(</code><code>self</code><code>):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>print</code><code>(</code><code>'name:%s&nbsp; age:%d'</code> <code>%</code> <code>(</code><code>self</code><code>.name, </code><code>self</code><code>.age))</code></p><p><code>z </code><code>=</code> <code>Zarten(</code><code>'zarten'</code><code>, </code><code>18</code><code>)</code></p><p><code>print</code><code>(</code><code>callable</code><code>(z))</code></p><p><code>z()</code></p></td></tr></tbody></table>

### 2.8__enter__()

一个上下文管理器的类，必须要实现这 2 个特殊方法`：__enter_()`和`__exit__()` 使用`with`语句来调用。

使用`__enter__()返`回对象，使用__exit__() 关闭对象

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p></td><td><p><code>class</code> <code>Zarten():</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__init__(</code><code>self</code><code>, file_name, method):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.file_obj </code><code>=</code> <code>open</code><code>(file_name, method)</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__enter__(</code><code>self</code><code>):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code> <code>self</code><code>.file_obj</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__exit__(</code><code>self</code><code>, exc_type, exc_val, exc_tb):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.file_obj.close()</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>print</code><code>(</code><code>'closed'</code><code>)</code></p><p><code>with Zarten(</code><code>'e:\\test.txt'</code><code>, </code><code>'r'</code><code>) as f:</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>r </code><code>=</code> <code>f.read()</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>print</code><code>(r)</code></p></td></tr></tbody></table>

### 2.9 __add__()

加法运算符重载以及`__radd__()`反向运算符重载

当对象作加法时，首先会在 “+” 左边对象查找`__add__()，`若没找到则在 “+” 右边查找`__radd__()`

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p><p>11</p><p>12</p><p>13</p><p>14</p><p>15</p><p>16</p></td><td><p><code>class</code> <code>Zarten():</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__init__(</code><code>self</code><code>, age):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.age </code><code>=</code> <code>age</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__add__(</code><code>self</code><code>, other):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code> <code>self</code><code>.age </code><code>+</code> <code>other</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__radd__(</code><code>self</code><code>, other):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>return</code>&nbsp; <code>self</code><code>.age </code><code>+</code> <code>other</code></p><p><code>z </code><code>=</code> <code>Zarten(</code><code>18</code><code>)</code></p><p><code>print</code><code>(z </code><code>+</code> <code>10</code><code>)</code></p><p><code>print</code><code>(</code><code>20</code> <code>+</code> <code>z)</code></p></td></tr></tbody></table>

### 2.10 __del__()

**对象生命周期结束时调用，相当于析构函数**

<table><tbody><tr><td><p>1</p><p>2</p><p>3</p><p>4</p><p>5</p><p>6</p><p>7</p><p>8</p><p>9</p><p>10</p></td><td><p><code>class</code> <code>Zarten():</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__init__(</code><code>self</code><code>, age):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>self</code><code>.age </code><code>=</code> <code>age</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;</code><code>def</code> <code>__del__(</code><code>self</code><code>):</code></p><p><code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code>print</code><code>(</code><code>'__del__'</code><code>)</code></p><p><code>z </code><code>=</code> <code>Zarten(</code><code>18</code><code>)</code></p></td></tr></tbody></table>

**特殊（魔术）方法汇总一览表**  

![](https://pic4.zhimg.com/80/v2-f38f4a447196a3a1e1af08dcea6c78e3_720w.jpg)

到此这篇关于 Python 基础 - 特殊方法详解的文章就介绍到这了, 更多相关 Python 基础 - 特殊方法内容请搜索脚本之家以前的文章或继续浏览下面的相关文章希望大家以后多多支持脚本之家！