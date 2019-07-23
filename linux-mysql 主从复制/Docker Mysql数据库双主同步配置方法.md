## Docker Mysql数据库双主同步配置方法





## (https://www.cnblogs.com/jinjiangongzuoshi/p/9299567.html)

# 一、背景

 

可先查看第一篇[《](http://www.cnblogs.com/jinjiangongzuoshi/p/9299275.html)[Docker Mysql数据库主从同步配置方法](https://www.cnblogs.com/jinjiangongzuoshi/p/9299275.html)[》](http://www.cnblogs.com/jinjiangongzuoshi/p/9299275.html)介绍

#  

# 二、具体操作

**1、创建目录(~/test/mysql_test1)：**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
--mysql
   --mone
      --data  
      --conf
         --my.cnf     
   --mtwo
      --data  
      --conf
         --my.cnf
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

**2、主主配置文件**

## Mone: my.cnf

```
[mysqld]
server_id = 1
log-bin= mysql-bin

replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema

read-only=0
relay_log=mysql-relay-bin
log-slave-updates=on
auto-increment-offset=1
auto-increment-increment=2

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```

#### 更详细配置文件：

```
[client]
port = 3306
socket = /tmp/mysql.sock
 
[mysqld]
basedir = /usr/local/mysql
port = 3306
socket = /tmp/mysql.sock
datadir = /usr/local/mysql/data
pid-file = /usr/local/mysql/data/mysql.pid
log-error = /usr/local/mysql/data/mysql.err
 
server-id = 1
auto_increment_offset = 1
auto_increment_increment = 2                                            #奇数ID
 
log-bin = mysql-bin                                                     #打开二进制功能,MASTER主服务器必须打开此项
binlog-format=ROW
binlog-row-p_w_picpath=minimal
log-slave-updates=true
gtid-mode=on
enforce-gtid-consistency=true
master-info-repository=TABLE
relay-log-info-repository=TABLE
sync-master-info=1
slave-parallel-workers=0
sync_binlog=0
binlog-checksum=CRC32
master-verify-checksum=1
slave-sql-verify-checksum=1
binlog-rows-query-log_events=1
#expire_logs_days=5
max_binlog_size=1024M                                                   #binlog单文件最大值
 
replicate-ignore-db = mysql                                             #忽略不同步主从的数据库
replicate-ignore-db = information_schema
replicate-ignore-db = performance_schema
replicate-ignore-db = test
replicate-ignore-db = zabbix
 
max_connections = 3000
max_connect_errors = 30
 
skip-character-set-client-handshake                                     #忽略应用程序想要设置的其他字符集
init-connect='SET NAMES utf8'                                           #连接时执行的SQL
character-set-server=utf8                                               #服务端默认字符集
wait_timeout=1800                                                       #请求的最大连接时间
interactive_timeout=1800                                                #和上一参数同时修改才会生效
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES                     #sql模式
max_allowed_packet = 10M
bulk_insert_buffer_size = 8M
query_cache_type = 1
query_cache_size = 128M
query_cache_limit = 4M
key_buffer_size = 256M
read_buffer_size = 16K
 
skip-name-resolve
slow_query_log=1
long_query_time = 6
slow_query_log_file=slow-query.log
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 16M
 
[mysql]
no-auto-rehash
 
[myisamchk]
key_buffer_size = 20M
sort_buffer_size = 20M
read_buffer = 2M
write_buffer = 2M
 
[mysqlhotcopy]
interactive-timeout
 
[mysqldump]
quick
max_allowed_packet = 16M
 
[mysqld_safe]
```

### mysql5.7修改版

```
[mysqld]
server_id = 1
log-bin= mysql-bin
binlog-format=ROW
gtid-mode=on								#开启gtid数据一致性
enforce-gtid-consistency=true
master-info-repository=TABLE
relay-log-info-repository=TABLE
sync-master-info=1							#每次事件提交都进行更新
slave-parallel-workers=5						#并行执行slave复制的工作线程数
slave-parallel-type=LOGICAL_CLOCK					#基于组提交的并行复制
binlog-checksum=CRC32
master-verify-checksum=1
slave-sql-verify-checksum=1
binlog-rows-query-log_events=1
#expire_logs_days=5
max_binlog_size=1024M  

replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema

read-only=0
relay_log=mysql-relay-bin
log-slave-updates=true
auto-increment-offset=1
auto-increment-increment=2

max_connections = 3000
max_connect_errors = 30

skip-character-set-client-handshake                                     #忽略应用程序想要设置的其他字符集
init-connect='SET NAMES utf8'                                           #连接时执行的SQL
character-set-server=utf8                                               #服务端默认字符集
wait_timeout=1800                                                       #请求的最大连接时间
interactive_timeout=1800                                                #和上一参数同时修改才会生效
sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION	#sql模式
#max_allowed_packet = 1024M											#允许最大数据包
#bulk_insert_buffer_size = 8M
#query_cache_type = 1
#query_cache_size = 128M
#query_cache_limit = 4M
#key_buffer_size = 256M
 
skip-name-resolve
#slow_query_log=1												#是否开启慢查询日志收集
#long_query_time = 6											#超过查询时间将记录
#slow_query_log_file=slow-query.log									#慢查询日志文件位置
innodb_flush_log_at_trx_commit = 2									#设置log bufferflush到磁盘操作的模式，每秒一次
innodb_log_buffer_size = 16M										#事务在内存中的缓存，不用太大，一般2-9m

[mysql]
no-auto-rehash
default-character-set=utf8

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/

```



### 特别参数说明

```
log-slave-updates = true     #将复制事件写入binlog,一台服务器既做主库又做从库此选项必须要开启
#masterA自增长ID



auto_increment_offset = 1



auto_increment_increment = 2                                            #奇数ID
#masterB自增加ID



auto_increment_offset = 2



auto_increment_increment = 2                                            #偶数ID
```



## Mtwo: my.cnf

```
[mysqld]
server_id = 2
log-bin= mysql-bin

replicate-ignore-db=mysql
replicate-ignore-db=sys
replicate-ignore-db=information_schema
replicate-ignore-db=performance_schema

read-only=0
relay_log=mysql-relay-bin
log-slave-updates=on
auto-increment-offset=2
auto-increment-increment=2

!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```

#### 更详细配置:

```
[client]
port = 3306
socket = /tmp/mysql.sock
 
[mysqld]
basedir = /usr/local/mysql
port = 3306
socket = /tmp/mysql.sock
datadir = /usr/local/mysql/data
pid-file = /usr/local/mysql/data/mysql.pid
log-error = /usr/local/mysql/data/mysql.err
 
server-id = 2
auto_increment_offset = 2
auto_increment_increment = 2                                            #偶数ID
 
log-bin = mysql-bin                                                     #打开二进制功能,MASTER主服务器必须打开此项
binlog-format=ROW
binlog-row-p_w_picpath=minimal
log-slave-updates=true
gtid-mode=on
enforce-gtid-consistency=true
master-info-repository=TABLE
relay-log-info-repository=TABLE
sync-master-info=1
slave-parallel-workers=0
sync_binlog=0
binlog-checksum=CRC32
master-verify-checksum=1
slave-sql-verify-checksum=1
binlog-rows-query-log_events=1
#expire_logs_days=5
max_binlog_size=1024M                                                   #binlog单文件最大值
 
replicate-ignore-db = mysql                                             #忽略不同步主从的数据库
replicate-ignore-db = information_schema
replicate-ignore-db = performance_schema
replicate-ignore-db = test
replicate-ignore-db = zabbix
 
max_connections = 3000
max_connect_errors = 30
 
skip-character-set-client-handshake                                     #忽略应用程序想要设置的其他字符集
init-connect='SET NAMES utf8'                                           #连接时执行的SQL
character-set-server=utf8                                               #服务端默认字符集
wait_timeout=1800                                                       #请求的最大连接时间
interactive_timeout=1800                                                #和上一参数同时修改才会生效
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES                     #sql模式
max_allowed_packet = 10M
bulk_insert_buffer_size = 8M
query_cache_type = 1
query_cache_size = 128M
query_cache_limit = 4M
key_buffer_size = 256M
read_buffer_size = 16K
 
skip-name-resolve
slow_query_log=1
long_query_time = 6
slow_query_log_file=slow-query.log
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 16M
 
[mysql]
no-auto-rehash
 
[myisamchk]
key_buffer_size = 20M
sort_buffer_size = 20M
read_buffer = 2M
write_buffer = 2M
 
[mysqlhotcopy]
interactive-timeout
 
[mysqldump]
quick
max_allowed_packet = 16M
 
[mysqld_safe]
```

 





**3、创建容器**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//创建并启动主从容器;
//mone
docker run --name monemysql -d -p 3317:3306 -e MYSQL_ROOT_PASSWORD=root -v ~/test/mysql_test1/mone/data:/var/lib/mysql -v ~/test/mysql_test1/mone/conf/my.cnf:/etc/mysql/my.cnf mysql:5.7

//mtwo
docker run --name mtwomysql -d -p 3318:3306 -e MYSQL_ROOT_PASSWORD=root -v ~/test/mysql_test1/mtwo/data:/var/lib/mysql -v ~/test/mysql_test1/mtwo/conf/my.cnf:/etc/mysql/my.cnf mysql:5.7
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

**4、容器设置详细**

mone容器设置：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//进入mone容器
docker exec -it monemysql mysql -u root -p
 
//启动mysql命令，刚在创建窗口时我们把密码设置为：root

 
//创建一个用户来同步数据
//这里表示创建一个slave同步账号slave，允许访问的IP地址为%，%表示通配符
GRANT REPLICATION SLAVE ON *.* to 'slave'@'%' identified by '123456';


//查看状态，记住File、Position的值，在mtwo中将用到
show master status;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

小技巧：

查看容器IP：

```
docker inspect monemysql | grep IPA
```

![img](https://images2018.cnblogs.com/blog/541408/201807/541408-20180712150714838-612080653.png)

 

mtwo容器设置：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//进入mtwo容器
docker exec -it mtwomysql mysql -u root -p
 
//启动mysql命令，刚在创建窗口时我们把密码设置为：root

//设置主库链接，master_host即为容器IP，master_log_file和master_log_pos即为在mone容器中，通过show master status查出来的值；
change master to master_host='172.17.0.11',master_user='slave',master_password='123456',master_log_file='mysql-bin.000004',master_log_pos=154,master_port=3306;

//创建一个用户来同步数据
GRANT REPLICATION SLAVE ON *.* to 'slave'@'%' identified by '123456';

//启动同步
start slave ;
 
//查看状态
show master status;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

设置完后，再次进入Mone容器

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//进入mone容器
//启动mysql命令，刚在创建窗口时我们把密码设置为：root
docker exec -it monemysql mysql -u root -p
 
 //先初始master 
 reset master;

//设置mtwo主库链接，参数详细说明同上
change master to master_host='172.17.0.12',master_user='slave',master_password='123456',master_log_file='mysql-bin.000004',master_log_pos=443,master_port=3306;

change master to MASTER_AUTO_POSITION=1;	#开启gtid复制 = 0 关闭gtid复制

//启动同步
start slave ;
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

配置完成之后，可以验证双主配置是否正确

在mone容器中，查看：

```
show slave status\G;
```

![img](https://images2018.cnblogs.com/blog/541408/201807/541408-20180712151038708-1884647817.png)

 

在mtwo容器中，查看：

```
show slave status\G;
```

![img](https://images2018.cnblogs.com/blog/541408/201807/541408-20180712151138471-282276978.png)

 

当红框两个Running状态都为Yes时，说明双主配置成功了～

 

# 三、验证

1、在mone库中操作：

```
create database mone_demo;
use mone_demo;
create table userinfo(username varchar(50),age int);
insert into userinfo values('Tom',18);
select * from userinfo;
```

![img](https://images2018.cnblogs.com/blog/541408/201807/541408-20180712145719213-200081498.png)

 

2、在mone库操作完后，在mtwo库中查看验证

![img](https://images2018.cnblogs.com/blog/541408/201807/541408-20180712145755462-1012067504.png)

首先查看数据库，发现数据库已经同步过来了，继续验证：

![img](https://images2018.cnblogs.com/blog/541408/201807/541408-20180712145836993-1555009656.png)

发现表的数据也同步过来了。

 

3、在mtwo库中，在此库，此表中，新增记录

```
insert into userinfo values('mtwo',20);
```

在mone库中查看，发现在mtwo库中新增的记录，确实也同步到mone库中来了哦～

![img](https://images2018.cnblogs.com/blog/541408/201807/541408-20180712150019733-1527247839.png)

 

 4、继续走一波验证，在mtwo库中，新增一个数据库，看是否同步到mone库中

```
create database mtwo_demo;
```

在mone库中，查看验证，查看数据库：

![img](https://images2018.cnblogs.com/blog/541408/201807/541408-20180712150239654-196643064.png)

发现在mtwo库新增的数据库，已经同步到了mone容器中来了