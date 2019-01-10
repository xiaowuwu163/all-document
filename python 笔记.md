# python 笔记

https://www.cnblogs.com/alex3714/articles/5885096.html

https://www.cnblogs.com/wupeiqi/articles/5950372.html mysql视图、存储过程

## 1、内置模块概念、from ，import 使用

import导入package  是执行__init__.py

import 导入.py 文件，是导入模块对象



from 。。。 import  。。。 是取出代码段直接执行

from . import test    在当前对象取出test模块或者方法



导入优化

from module_test import test 



模块的分类：

1、标准库

2、开源模块

3、自定义模块





时间戳 

time.time() 1970年到现在的时间

time.gmtime: UTC时区 

time.localtime: 结果为中国UTC+8时区

sys模块

os模块

json、pipe..模块

xml



## 面向对象

封装、继承、多态

接口



@装饰器

@staticmethod静态方法，和类没有关系，独立方法属性，self参数不起作用

@classmethod类方法，不能调用对象私有属性，只能调用类属性（变量）

@property 属性方法，把一个方法变成一个静态属性，不用加（）调用

​	@属性方法名.setter 设置属性方法的属性

​	@属性方法名.deletter 删除属性方法的属性



## 反射

​	hasattr(obj,name_str ),判断一个对象obj里是否有对应的name_str字符串的方法

​	getattr(obj,name_str),根据字符串去获取obj对象里的对应的方法的内存地址

​	setattr(obj,name_str,value),根据obj.name_str=value,设置一个属性或者方法

​	delattr(obj,name_str)删除一个属性



## 异常exception

try: 试图通过

except   ...Erro as e:抓取单个错误

except (..Error,...Error) as e : 住区多个错误

except Exception as e:抓取子类所有异常，放最后

​	捕获异常

else： 一切正常就执行

finally：最终执行

#### 常用异常

AttributeError 试图访问一个对象没有的树形，比如foo.x，但是foo没有属性x
IOError 输入/输出异常；基本上是无法打开文件
ImportError 无法引入模块或包；基本上是路径问题或名称错误
IndentationError 语法错误（的子类） ；代码没有正确对齐
IndexError 下标索引超出序列边界，比如当x只有三个元素，却试图访问x[5]
KeyError 试图访问字典里不存在的键
KeyboardInterrupt Ctrl+C被按下
NameError 使用一个还未被赋予对象的变量
SyntaxError Python代码非法，代码不能编译(个人认为这是语法错误，写错了）
TypeError 传入对象类型与要求的不符合
UnboundLocalError 试图访问一个还未被设置的局部变量，基本上是由于另有一个同名的全局变量，
导致你以为正在访问它
ValueError 传入一个调用者不期望的值，即使值的类型是正确的



#### 其他异常，更多异常

ArithmeticError
AssertionError
AttributeError
BaseException
BufferError
BytesWarning
DeprecationWarning
EnvironmentError
EOFError
Exception
FloatingPointError
FutureWarning
GeneratorExit
ImportError
ImportWarning
IndentationError
IndexError
IOError
KeyboardInterrupt
KeyError
LookupError
MemoryError
NameError
NotImplementedError
OSError
OverflowError
PendingDeprecationWarning
ReferenceError
RuntimeError
RuntimeWarning
StandardError
StopIteration
SyntaxError
SyntaxWarning
SystemError
SystemExit
TabError
TypeError
UnboundLocalError
UnicodeDecodeError
UnicodeEncodeError
UnicodeError
UnicodeTranslateError
UnicodeWarning
UserWarning
ValueError
Warning
ZeroDivisionError



自定义异常

`class WupeiqiException(Exception):`

    def __init__(self, msg):
        self.message = msg
     
    def __str__(self):
        return self.message

`try:`
   ` raise WupeiqiException('我的异常')`
`except WupeiqiException,e:`
   ` print e`



### socket 网络编程

网络协议种类 osi七层

http(网站) smtp(email邮件) dns(域名解析) ftp(文件传输) ssh(linux登录) snmp(网络监控) icmp(ping 包)dhcp(分配ip 地址)

网络七层协议s

应用

表示

会话

传输 tcp/ip

网络	ip

数据链路层 mac

物理层  

65535 一个IP地址的端口大小

UDP协议，不安全，直接传输，有丢包的可能











