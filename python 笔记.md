# python 笔记

https://www.cnblogs.com/alex3714/articles/5885096.html

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



反射

​	hasattr(obj,name_str ),判断一个对象obj里是否有对应的name_str字符串的方法

​	getattr(obj,name_str),根据字符串去获取obj对象里的对应的方法的内存地址

​	setattr(obj,name_str,value),根据obj.name_str=value,设置一个属性或者方法

​	delattr(obj,name_str)删除一个属性



异常exception

try: 试图通过

except   ...Erro as e:抓取单个错误

except (..Error,...Error) as e : 住区多个错误

except Exception as e:抓取子类所有异常，放最后

​	捕获异常

else： 一切正常就执行

finally：最终执行







