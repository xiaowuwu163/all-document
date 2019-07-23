# 使用Celery踩过的坑

![96](https://upload.jianshu.io/users/upload_avatars/666435/c9edbd7c9bb5?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96)

 

[李俊伟_](https://www.jianshu.com/u/25544b04b1c4)

 

关注

 0.1 2016.05.08 17:02* 字数 1513 阅读 26505评论 10喜欢 31赞赏 1

## 为什么要使用celery

Celery是一个使用Python开发的分布式任务调度模块，因此对于大量使用Python构建的系统，可以说是无缝衔接，使用起来很方便。Celery专注于实时处理任务，同时也支持任务的定时调度。因此适合实时异步任务定时任务等调度场景。Celery需要依靠RabbitMQ等作为消息代理，同时也支持Redis甚至是Mysql，Mongo等，当然，官方默认推荐的是RabbitMQ。

## broker的选择

虽然官方支持的broker有很多，包括RabbitMQ，Redis甚至是数据库，但是不推荐使用数据库，因为数据库需要不断访问磁盘，当你的任务量大了之后会造成很严重的性能问题，同时你的应用很可能也在使用同一个数据库，这样可能导致你的应用被拖垮。如果业务环境比较简单可以选择Redis，如果比较复杂选择RabbitMQ，因为RabbitMQ是官方推荐的，但是比Redis操作起来又相对复杂些。我的选择是broker用RabbitMQ，backend用Redis

## celery不能用root用户启动问题 C_FORCE_ROOT environment

如果使用root用户启动celery会遇到下面的问题

```
Running a worker with superuser privileges when the
worker accepts messages serialized with pickle is a very bad idea!
If you really want to continue then you have to set the C_FORCE_ROOT
environment variable (but please think about this before you do).
```

解决办法：

```
from celery import Celery, platforms

platforms.C_FORCE_ROOT = True  #加上这一行
```

## 任务重复执行

celery执行定时任务的时候遇到了重复执行的问题，当时是用redis做broker和backend。
[官方文档](https://link.jianshu.com/?t=http://docs.celeryproject.org/en/latest/getting-started/brokers/redis.html)中有相关描述。

> If a task is not acknowledged within the Visibility Timeout the task will
> be redelivered to another worker and executed.

> This causes problems with ETA/countdown/retry tasks where the time to execute exceeds the visibility timeout; in fact if that happens it will be executed again, and again in a loop.

> So you have to increase the visibility timeout to match the time of the longest ETA you are planning to use.

> Note that Celery will redeliver messages at worker shutdown, so having a long visibility timeout will only delay the redelivery of ‘lost’ tasks in the event of a power failure or forcefully terminated workers.

> Periodic tasks will not be affected by the visibility timeout, as this is a concept separate from ETA/countdown.

> You can increase this timeout by configuring a transport option with the same name:

> BROKER_TRANSPORT_OPTIONS = {'visibility_timeout': 43200}

> The value must be an int describing the number of seconds.

就是说当我们设置一个ETA时间比visibility_timeout长的任务时，每过一次 visibility_timeout 时间，celery就会认为这个任务没被worker执行成功，重新分配给其它worker再执行。
解决办法就是把 visibility_timeout参数调大，比我们ETA的时间差要大。celery本身的定位就主要是实时的异步队列，对于这种长时间定时执行，支持不太好。
但是第二天依然重复执行了。。。

最后我的解决方法是在每次定时任务执行完就在redis中写入一个唯一的key对应一个时间戳，当下次任务执行前去获取redis中的这个key对应的value值，和当前的时间做比较，当满足我们的定时频率要求时才执行，这样保证了同一个任务在规定的时间内只会执行一次。

## 使用不同的queue

当你有很多任务需要执行的时候，不要偷懒只使用默认的queue，这样会相互影响，并且拖慢任务执行的，导致重要的任务不能被快速的执行。鸡蛋不能放在同一个篮子里的道理大家都懂。
有一种简单的方式设置queue

> Automatic routing

> The simplest way to do routing is to use the CELERY_CREATE_MISSING_QUEUES setting (on by default).

> With this setting on, a named queue that is not already defined in CELERY_QUEUES will be created automatically. This makes it easy to perform simple routing tasks.

> Say you have two servers, x, and y that handles regular tasks, and one server z, that only handles feed related tasks. You can use this configuration:

> CELERY_ROUTES = {'feed.tasks.import_feed': {'queue': 'feeds'}}

> With this route enabled import feed tasks will be routed to the “feeds” queue, while all other tasks will be routed to the default queue (named “celery” for historical reasons).

> Now you can start server z to only process the feeds queue like this:

> user@z:/$ celery -A proj worker -Q feeds

> You can specify as many queues as you want, so you can make this server process the default queue as well:

> user@z:/$ celery -A proj worker -Q feeds,celery

直接使用

```
CELERY_ROUTES = {'feed.tasks.import_feed': {'queue': 'feeds'}}
user@z:/$ celery -A proj worker -Q feeds,celery
```

指定routes,就会自动生成对应的queue，然后使用-Q指定queue启动celery就可以，默认的queue名字是celery。可以看[官方文档](https://link.jianshu.com/?t=http://docs.celeryproject.org/en/latest/userguide/routing.html#automatic-routing)对默认queue的名字进行修改。

## 启动多个workers执行不同的任务

在同一台机器上，对于优先级不同的任务最好启动不同的worker去执行，比如把实时任务和定时任务分开，把执行频率高的任务和执行频率低的任务分开，这样有利于保证高优先级的任务可以得到更多的系统资源，同时高频率的实时任务日志比较多也会影响实时任务的日志查看，分开就可以记录到不同的日志文件，方便查看。

```
$ celery -A proj worker --loglevel=INFO --concurrency=10 -n worker1.%h
$ celery -A proj worker --loglevel=INFO --concurrency=10 -n worker2.%h
$ celery -A proj worker --loglevel=INFO --concurrency=10 -n worker3.%h
```

可以像这样启动不同的worker，%h可以指定hostname，详细说明可以查看[官方文档](https://link.jianshu.com/?t=http://docs.celeryproject.org/en/latest/userguide/workers.html)
高优先级的任务可以分配更多的concurrency，但是并不是worker并法数越多越好，保证任务不堆积就好。

## 是否需要关注任务执行状态

这个要视具体的业务场景来看，如果对结果不关心，或者任务的执行本身会对数据产生影响，通过对数据的判断可以知道执行的结果那就不需要返回celery任务的退出状态，可以设置

```
CELERY_IGNORE_RESULT = True
```

或者

```
@app.task(ignore_result=True)
def mytask(…):
    something()
```

但是，如果业务需要根据任务执行的状态进行响应的处理就不要这样设置。

## 内存泄漏

长时间运行Celery有可能发生内存泄露，可以像下面这样设置

```
CELERYD_MAX_TASKS_PER_CHILD = 40 # 每个worker执行了多少任务就会死掉
```

## 补充

Django下要查看其他celery的命令，包括参数配置、启动多worker进程的方式都可以通过python manage.py celery --help来查看:

![img](https://images2015.cnblogs.com/blog/870989/201607/870989-20160702161929062-1646457731.png)

 　　另外，Celery提供了一个工具flower，将各个任务的执行情况、各个worker的健康状态进行监控并以可视化的方式展现，如下图所示：

![img](https://images2015.cnblogs.com/blog/870989/201607/870989-20160702160947515-1604046730.png)

　　Django下实现的方式如下：　

　　1. 安装flower:

```
pip install flower
```

　　2. 启动flower（默认会启动一个webserver，端口为5555）:

```
python manage.py celery flower
```

　　3. 进入http://localhost:5555即可查看。