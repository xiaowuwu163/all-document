# MHA 清理relay log（purge_relay_logs）

﻿﻿

​    MySQL数据库主从复制在缺省情况下从库的relay logs会在SQL线程执行完毕后被自动删除，但是对于MHA场景下，对于某些滞后从库的恢复依赖于其他从库的relay log，因此采取禁用自动删除功能以及定期清理的办法。对于清理过多过大的relay log需要注意引起的复制延迟资源开销等。MHA可通过purge_relay_logs脚本及配合cronjob来完成此项任务，具体描述如下。

```
1、purge_relay_logs的功能
  a、为relay日志创建硬链接(最小化批量删除大文件导致的性能问题)
  b、SET GLOBAL relay_log_purge=1; FLUSH LOGS; SET GLOBAL relay_log_purge=0;
  c、删除relay log（rm –f  /path/to/archive_dir/*）

2、purge_relay_logs的用法及相关参数
\###用法
\# purge_relay_logs --help
Usage:
    purge_relay_logs --user=root --password=rootpass --host=127.0.0.1

\###参数描述
--user mysql              用户名，缺省为root
--password mysql          密码
--port                    端口号
--host                    主机名，缺省为127.0.0.1
--workdir                 指定创建relay log的硬链接的位置，默认是/var/tmp，成功执行脚本后，硬链接的中继日志文件被删除
                          由于系统不同分区创建硬链接文件会失败，故需要执行硬链接具体位置，建议指定为relay log相同的分区
--disable_relay_log_purge 默认情况下，参数relay_log_purge=1，脚本不做任何处理，自动退出
                          设定该参数，脚本会将relay_log_purge设置为0，当清理relay log之后，最后将参数设置为OFF(0)

3、定制清理relay log cronjob
pureg_relay_logs脚本在不阻塞SQL线程的情况下自动清理relay log。对于不断产生的relay log直接将该脚本部署到crontab以实现按天或按小时定期清理。
$ crontab -l  
\# purge relay logs at 5am  
0 5 * * * /usr/bin/purge_relay_logs --user=root --password=PASSWORD --disable_relay_log_purge >> /var/log/masterha/purge_relay_logs.log 2>&1            更正，移除多余字符app @20150515

4、手动清理示例
\# purge_relay_logs --user=mha --password=mha --disable_relay_log_purge 
2015-04-23 14:33:20: purge_relay_logs script started.
 relay_log_purge is enabled. Disabling..
 Found relay_log.info: /data/mysqldata/relay-log.info
 Opening /data/mysqldata/vdbsrv3-relay-bin.000001 ..
 Opening /data/mysqldata/vdbsrv3-relay-bin.000002 ..
 Executing SET GLOBAL relay_log_purge=1; FLUSH LOGS; sleeping a few seconds so that SQL thread can delete older relay log files (if it keeps up); 
 SET GLOBAL relay_log_purge=0; .. ok.
2015-04-23 14:33:23: All relay log purging operations succeeded.
```

