# django之使用celery-把耗时程序放到celery里面执行

2018年04月12日 20:17:50 [qq_19339041](https://me.csdn.net/qq_19339041) 阅读数：2632

**１**　在虚拟环境创建项目test和应用booktest（过程省略）,然后安装所需的包

```
pip install celery==3.1.25
pip install celery-with-redis==3.0
pip install django-redis==3.1.17123
```

**2**　配置settings，

```
# 数据库使用mysql
DATABASES = {
    'default': ｛
        'ENGINE':'django.db.backends.mysql',
        'NAME':'test',
        'USER':'root',
        'PASSWORD':'mysql',
        'HOST':'localhost',
        'PORT':3306,
    }
}

# 注册djcelery应用
INSTALLED_APPS = (
    ...
    'djcelery',
)

# celery配置

# 如报错　ImportError: No module named djcelery　，是因为没有在虚拟环境运行导致，　workon h1进入虚拟环境再运行解决
import djcelery

#　初始化所有的task任务，这些任务来自booktest.task模块
djcelery.setup_loader()

# 使用redis第０个数据库，并绑定ip端口
BROKER_URL='redis://127.0.0.1:6379/0'

#　设置初始化的任务来源
CELERY_IMPORTS = 'booktest.task'

123456789101112131415161718192021222324252627282930313233
```

**３**　在应用目录booktest下面创建任务列表文件task.py

```
from celery import task
import time

# 加上@task装饰器，则python函数就变成一个celery任务
@task
def celery_test():
    print('hello...')
    time.sleep(5)
    print('world...')123456789
```

**４**　创建视图，并配置相关的url配置，把耗时任务放入视图被调用

```
#　-*- coding:utf-8 -*-
from django.shortcuts import render
from django.http import HttpResponse
from task import celery_test


# celery练习１：把耗时程序放在celery中执行
def celerytest(request):
    # function.delay(参数),celery任务celery_test调用方法
    celery_test.delay()
    return HttpResponse('ok')

# 根级url配置 test.urls
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    url(r'^celery/', include('booktest.urls')),
]

# 应用下的url配置　ｂooktest.urls
from django.conf.urls import url
import views

urlpatterns=[
    url(r'^celerytest/$', views.celerytest)
]12345678910111213141516171819202122232425262728
```

**５**　迁移，生成celery所需的数据表

```
python manage.py migrate1
```

**６**　启动redis

```
sudo redis-server /etc/redis/redis.conf1
```

**7**　启动worker

```
python manage.py celery worker --loglevel=info1
```

**8**　另开一个终端窗口，启动ｄjango[服务器](https://www.baidu.com/s?wd=%E6%9C%8D%E5%8A%A1%E5%99%A8&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)

```
python manage.py runserver1
```

**9** 测试，输入url，如　<http://127.0.0.1:8000/celery/celerytest/>，则返回’ok’

　　同时，会在worker对应的窗口看到耗时任务程序在此输出，

　　即当用户请求时，不用等待太久就可以得到结果’ok’，同时耗时任务程序也被

　　异步执行，提高用户体验．