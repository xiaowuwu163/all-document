# mysql 5.7 开启半同步复制

 版权声明：本文为博主原创文章，未经博主允许不得转载。 https://blog.csdn.net/isoleo/article/details/77892950

**1.安装相关的插件**

```
show plugins;  查看模块

help --uninstall; 查看卸载模块

master:

mysql> install plugin rpl_semi_sync_master soname 'semisync_master.so';  --安装 semisync_master.so插件 

Query OK, 0 rows affected (0.03 sec)

slave:

root@localhost [zw3306]>install plugin rpl_semi_sync_slave soname 'semisync_slave.so'; --安装 semisync_slave.so插件

Query OK, 0 rows affected (0.00 sec)

root@localhost [zw3306]>install plugin rpl_semi_sync_master soname 'semisync_master.so';

Query OK, 0 rows affected (0.00 sec)
```

**2.修改的参数：**

```
set global loose_rpl_semi_sync_master_enabled=1;

set global loose_rpl_semi_sync_master_timeout=1000;

set global loose_rpl_semi_sync_slave_enabled=1;

也可以直接写到配置文件 [mysqld]

master:

[mysqld]

loose_rpl_semi_sync_master_enabled = 1 

loose_rpl_semi_sync_master_timeout = 1000 # 1 second

slave：

[mysqld]

loose_rpl_semi_sync_slave_enabled = 1
```

配置文件：

```
[client]
port=3306
socket=/var/lib/mysql/mysql.sock
default-character-set=utf8

[mysqld]
datadir=/var/lib/mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
init-connect='SET NAMES utf8'                                           #连接时执行的SQL
character-set-server=utf8                                               #服务端默认字符集
lower_case_table_names=1

server_id = 1000
log-bin= mysql-bin
gtid-mode=on                                                            #开启gtid数据一致性
enforce-gtid-consistency=true
master-info-repository=TABLE
relay-log-info-repository=TABLE
#log-slave-updates=true 　　　　 #在从服务器进入主服务器传入过来的修改日志所使用，在Mysql5.7之前主从架构上使用gtid模式的话，必须使用此选项，在Mysql5.7取消了，会增加系统负载。 
binlog-format=ROW
slave-parallel-type=LOGICAL_CLOCK                                       #基于组提交的并行复制
sync-master-info=1                                                      #每次事件提交都进行更新
slave-parallel-workers=16                                               #并行执行slave复制的工作线程数
binlog-checksum=CRC32													#主服务端在启动的时候要不要校验bin-log本身的校验码
master-verify-checksum=1        										#都是在服务器出现故障情况下，读取对服务器有用的数据
slave-sql-verify-checksum=1
read_only=1															#做mha高可用，从服务器可开启

#开启半同步复制  否则自动切换主从的时候会报主键错误
plugin_load = "rpl_semi_sync_master=semisync_master.so;rpl_semi_sync_slave=semisync_slave.so"
loose_rpl_semi_sync_master_enabled = 1
loose_rpl_semi_sync_slave_enabled = 1
loose_rpl_semi_sync_master_timeout = 5000

[mysql]
no-auto-rehash
default-character-set=utf8

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```



修改了参数需要重启：

查看修改的参数

```
master: 

mysql> show global variables like '%rpl_semi%';

+-------------------------------------------+------------+

| Variable_name                             | Value      |

+-------------------------------------------+------------+

| rpl_semi_sync_master_enabled              | ON         |

| rpl_semi_sync_master_timeout              | 1000       |

| rpl_semi_sync_master_trace_level          | 32         |

| rpl_semi_sync_master_wait_for_slave_count | 1          |

| rpl_semi_sync_master_wait_no_slave        | ON         |

| rpl_semi_sync_master_wait_point           | AFTER_SYNC |

+-------------------------------------------+------------+

6 rows in set (0.00 sec)

slave:

root@localhost [(none)]>show global variables like '%rpl_semi%';

+-------------------------------------------+------------+

| Variable_name                             | Value      |

+-------------------------------------------+------------+

| rpl_semi_sync_master_enabled              | ON         |

| rpl_semi_sync_master_timeout              | 1000       |

| rpl_semi_sync_master_trace_level          | 32         |

| rpl_semi_sync_master_wait_for_slave_count | 1          |

| rpl_semi_sync_master_wait_no_slave        | ON         |

| rpl_semi_sync_master_wait_point           | AFTER_SYNC |

| rpl_semi_sync_slave_enabled               | ON         |

| rpl_semi_sync_slave_trace_level           | 32         |

+-------------------------------------------+------------+

8 rows in set (0.00 sec)
```

**如果：如果原来是已经建好的复制结构很简单：**

```
stop slave io_thread;

start slave io_thread;

3.做同步

change master to master_host='192.168.26.233', master_port=3306, master_user='repl',master_password='repl', master_auto_position=1;

 

root@localhost [(none)]>start slave;

Query OK, 0 rows affected (0.01 sec)

root@localhost [(none)]>show slave status\G;

*************************** 1. row ***************************

               Slave_IO_State: Waiting for master to send event

                  Master_Host: 192.168.26.233

                  Master_User: repl

                  Master_Port: 3306

                Connect_Retry: 60

              Master_Log_File: mysql-bin.000002

          Read_Master_Log_Pos: 194

               Relay_Log_File: relay-bin.000004

                Relay_Log_Pos: 407

        Relay_Master_Log_File: mysql-bin.000002

             Slave_IO_Running: Yes

            Slave_SQL_Running: Yes

              Replicate_Do_DB: 

          Replicate_Ignore_DB: 

           Replicate_Do_Table: 

       Replicate_Ignore_Table: 

      Replicate_Wild_Do_Table: 

  Replicate_Wild_Ignore_Table: 

                   Last_Errno: 0

                   Last_Error: 

                 Skip_Counter: 0

          Exec_Master_Log_Pos: 194

              Relay_Log_Space: 861

              Until_Condition: None

               Until_Log_File: 

                Until_Log_Pos: 0

           Master_SSL_Allowed: No

           Master_SSL_CA_File: 

           Master_SSL_CA_Path: 

              Master_SSL_Cert: 

            Master_SSL_Cipher: 

               Master_SSL_Key: 

        Seconds_Behind_Master: 0

Master_SSL_Verify_Server_Cert: No

                Last_IO_Errno: 0

                Last_IO_Error: 

               Last_SQL_Errno: 0

               Last_SQL_Error: 

  Replicate_Ignore_Server_Ids: 

             Master_Server_Id: 3306100

                  Master_UUID: 7e354a2c-6f5f-11e6-997d-005056a36f08

             Master_Info_File: mysql.slave_master_info

                    SQL_Delay: 0

          SQL_Remaining_Delay: NULL

      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates

           Master_Retry_Count: 86400

                  Master_Bind: 

      Last_IO_Error_Timestamp: 

     Last_SQL_Error_Timestamp: 

               Master_SSL_Crl: 

           Master_SSL_Crlpath: 

           Retrieved_Gtid_Set: 7e354a2c-6f5f-11e6-997d-005056a36f08:1-9

            Executed_Gtid_Set: 7e354a2c-6f5f-11e6-997d-005056a36f08:1-9,

ba0d5587-74d6-11e6-ab5c-005056a3f46e:1-2

                Auto_Position: 1

         Replicate_Rewrite_DB: 

                 Channel_Name: 

           Master_TLS_Version: 

1 row in set (0.00 sec)

ERROR: 

No query specified
```

**5.查看slave是否有数据**

```
root@localhost [zw3306]>show tables;

+------------------+

| Tables_in_zw3306 |

+------------------+

| t1               |

+------------------+

1 row in set (0.00 sec)

root@localhost [zw3306]>select * from t1;

+------+

| id   |

+------+

|    1 |

|    2 |

|    3 |

|    4 |

+------+

4 rows in set (0.00 sec)
```

**6. 怎么确认是同步还是半同步？**

```
show global variables like '%semi%';

show global status like '%semi%';

master:

root@localhost [zw3306]>show global status like '%semi%';

+--------------------------------------------+-------+

| Variable_name                              | Value |

+--------------------------------------------+-------+

| Rpl_semi_sync_master_clients               | 1     | 有多少个Semi-sync的备库

| Rpl_semi_sync_master_net_avg_wait_time     | 0     | 事务提交后，等待备库响应的平均时间

| Rpl_semi_sync_master_net_wait_time         | 0     | 等待网络响应的总次数

| Rpl_semi_sync_master_net_waits             | 7     | 总的网络等待时间

| Rpl_semi_sync_master_no_times              | 0     | 一共有几次从Semi-sync跌回普通状态

| Rpl_semi_sync_master_no_tx                 | 0     | 库未及时响应的事务数,如果这个值很大就有问题

| Rpl_semi_sync_master_status                | ON    | 主库上Semi-sync是否正常开启

| Rpl_semi_sync_master_timefunc_failures     | 0     | 时间函数未正常工作的次数

| Rpl_semi_sync_master_tx_avg_wait_time      | 410   | 开启Semi-sync，事务返回需要等待的平均时间

| Rpl_semi_sync_master_tx_wait_time          | 2876  | 事务等待备库响应的总时间

| Rpl_semi_sync_master_tx_waits              | 7     | 事务等待备库响应的总次数

| Rpl_semi_sync_master_wait_pos_backtraverse | 0     | 改变当前等待最小二进制日志的次数

| Rpl_semi_sync_master_wait_sessions         | 0     | 当前有几个线程在等备库响应

| Rpl_semi_sync_master_yes_tx                | 7     | Semi-sync模式下，成功的事务数

+--------------------------------------------+-------+
```

15 rows in set (0.00 sec)

```
root@localhost [zw3306]>show global variables like '%semi%';

+-------------------------------------------+------------+

| Variable_name                             | Value      |

+-------------------------------------------+------------+

| rpl_semi_sync_master_enabled              | ON         |

| rpl_semi_sync_master_timeout              | 1000       |

| rpl_semi_sync_master_trace_level          | 32         |

| rpl_semi_sync_master_wait_for_slave_count | 1          |

| rpl_semi_sync_master_wait_no_slave        | ON         |

| rpl_semi_sync_master_wait_point           | AFTER_SYNC |

| rpl_semi_sync_slave_enabled               | ON         |

| rpl_semi_sync_slave_trace_level           | 32         |

+-------------------------------------------+------------+

8 rows in set (0.00 sec)

Rpl_semi_sync_master_no_tx  如果这个值很大就有问题

也有其他的原因；

mysql> show global status like '%semi%';

+--------------------------------------------+-------+

| Variable_name                              | Value |

+--------------------------------------------+-------+

| Rpl_semi_sync_master_clients               | 1     |

| Rpl_semi_sync_master_net_avg_wait_time     | 0     |

| Rpl_semi_sync_master_net_wait_time         | 0     |

| Rpl_semi_sync_master_net_waits             | 17    |

| Rpl_semi_sync_master_no_times              | 1     |

| Rpl_semi_sync_master_no_tx                 | 10    | 不是用半同步复制的

| Rpl_semi_sync_master_status                | OFF   |

| Rpl_semi_sync_master_timefunc_failures     | 0     |

| Rpl_semi_sync_master_tx_avg_wait_time      | 410   |

| Rpl_semi_sync_master_tx_wait_time          | 2876  |

| Rpl_semi_sync_master_tx_waits              | 7     |

| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |

| Rpl_semi_sync_master_wait_sessions         | 0     |

| Rpl_semi_sync_master_yes_tx                | 7     |

+--------------------------------------------+-------+
```

14 rows in set (0.01 sec)

操作10个事物，可以发现都是  Rpl_semi_sync_master_no_tx 

master接收到N个slave的应答后，才commit事物，等待1s用户可以设置应道slave的数量。

rpl_semi_sync_master_wait_for_slave_count=1 默认是1 

set global rpl_semi_sync_master_wait_for_slave_count=2;

意思是多少个半同步的slave;