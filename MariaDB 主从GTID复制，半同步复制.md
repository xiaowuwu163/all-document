# MariaDB 主从GTID复制，半同步复制



## 配置文件

```
[client]
port=3306
socket=/var/lib/mysql/mysql.sock
default-character-set=utf8

[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
init-connect='SET NAMES utf8'                                           #连接时执行的SQL
character-set-server=utf8                                               #服务端默认字符集
lower_case_table_names=1
skip_name_resolve													#进制dns域名反解析

server_id = 1000
log-bin= mysql-bin
binlog-format=ROW
#sync-master-info=1                                                      #每次事件提交都进行更新，大量磁盘I/O
slave-parallel-threads=16                                               #并行执行slave复制的工作线程数
binlog-checksum=CRC32									   #主服务端在启动的时候要不要校验bin-log本身的校验码
master-verify-checksum=1        							   #都是在服务器出现故障情况下，读取对服务器有用的数据
slave-sql-verify-checksum=1
gtid_strict_mode=on			#从服务器事务严格模式开启，不执行事务

#过滤某个库，主服务器禁止进行复制
#binlog_ignore_db=mysql
#过滤某个库，从服务器不进行复制
#replicate_ignore_db=mysql

relay_log=mysql-relay-bin
log_slave_updates=1
relay_log_purge=0							#不删除relay-log中继日志
innodb_file_per_table=ON					#一个表一个独立空间
expire_logs_days=3							#日志过期时间
relay_log_recovery
#当slave从库宕机后，假如relay-log损坏了，导致一部分中继日志没有处理，则自动放弃所有未执行的relay-log，并且重新从master上获取日志，这样就保证了relay-log的完整性。


rpl_semi_sync_master_enabled = 1
rpl_semi_sync_slave_enabled = 1
rpl_semi_sync_master_timeout = 5000

[mysql]
no-auto-rehash
default-character-set=utf8

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

```

