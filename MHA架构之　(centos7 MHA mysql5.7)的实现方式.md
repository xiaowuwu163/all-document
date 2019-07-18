# MHA架构之　(centos7 MHA mysql5.7)的实现方式

国内地址：

一：mha4mysql-0.56 ， 0.58版本下载地址:https://github.com/yoshinorim/mha4mysql-manager/releases

二：百度网盘MHA0.56版本

[![img](https://qiniu.wsfnk.com/mha01.png)](https://qiniu.wsfnk.com/mha01.png)
**安装mysql5.7，并配置好主从复制**

```
第一：安装mysql57，并关闭防火墙
	yum install epel*  -y && yum clean all && yum makecache 
	rpm -Uvh http://repo.mysql.com/mysql57-community-release-el7.rpm
	yum clean all && yum makecache
	yum install gcc gcc-c++ openssl-devel mysql mysql-server mysql-devel -y

	systemctl disable firewalld
	systemctl stop firewalld

	systemctl start mysqld
	cat /var/log/mysqld.log | grep 'password is generated'	#找到mysql初始密码
	mysql_secure_installation	#对mysql进行安全加固


第二：修改my.cnf(三台都要，只有server-id修改为不一样的就好啦)
	vi /etc/my.cnf

	[client]
	user=root
	password=123456

	[mysqld]
	datadir=/var/lib/mysql
	socket=/var/lib/mysql/mysql.sock
	log-error=/var/log/mysqld.log
	pid-file=/var/run/mysqld/mysqld.pid

	#每个server上不一致，见规划
	server-id = 1
	#read-only=1	#不在配置文件中限定只读，但是要记得在slave上限制只读

	#mysql5.6已上的特性，开启gtid，必须主从全开
	gtid_mode = on	
	enforce_gtid_consistency = 1
	log_slave_updates = 1

	#开启半同步复制  否则自动切换主从的时候会报主键错误
	plugin_load = "rpl_semi_sync_master=semisync_master.so;rpl_semi_sync_slave=semisync_slave.so"
	loose_rpl_semi_sync_master_enabled = 1
	loose_rpl_semi_sync_slave_enabled = 1
	loose_rpl_semi_sync_master_timeout = 5000

	log-bin=mysql-bin
	relay-log = mysql-relay-bin
	replicate-wild-ignore-table=mysql.%						#要过滤的表
	replicate-wild-ignore-table=test.%
	replicate-wild-ignore-table=information_schema.%

第三：在 3 个 mysql 节点做授权配置(主从复制授权)
	mysql> grant replication slave on *.* to 'repl_user'@'192.168.1.%' identified by '123456';
	mysql> grant all on *.* to 'root'@'192.168.1.%' identified by '123456'; #很重要

第四：在两个salve节点上执行，只读限制（防止意外被写数据，很重要）
	mysql> set global read_only=1;

第五：在主master上查看状态
	mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000002 |     1298 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

第六：在两个slave节点执行下面的操作

	mysql> change master to master_host='192.168.1.57',master_user='repl_user',master_password='123456',master_log_file='mysql-bin.000002',master_log_pos=1298;

	mysql> start slave;
	mysql> show slave status\G;	#查看slave IO和slave sql是否都正常
```

**MHA基本环境准备**

```
第七：配置三台机器的ssh互信（三台都要操作）
	ssh-keygen -t rsa
	ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.56
	ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.57
	ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.58

	#测试是否成功
	ssh 192.168.1.57 date

第八：安装MHA软件（在三个节点上都装mha的node软件）
	#先安装依赖
	wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
	rpm -ivh epel-release-latest-7.noarch.rpm
	yum install perl-DBD-MySQL perl-Config-Tiny perl-Log-Dispatch perl-Parallel-ForkManager　-y

	下载软件（方式任选其一）
	mha 0.58版本不适合mariadb 10.3，需用mha0.56
	(#wget https://qiniu.wsfnk.com/mha4mysql-node-0.58-0.el7.centos.noarch.rpm
	#wget https://github.com/yoshinorim/mha4mysql-node/releases/download/v0.58/mha4mysql-node-0.58-0.el7.centos.noarch.rpm
	#rpm -ivh mha4mysql-node-0.58-0.el7.centos.noarch.rpm)
	
	
	(压缩包中文件)
	rpm -ivh mha4mysql-node-0.56-0.el6.noarch.rpm
	
	
第九：仅在manager节点上安装mha管理软件
	#wget https://qiniu.wsfnk.com/mha4mysql-manager-0.58-0.el7.centos.noarch.rpm
	#wget https://github.com/yoshinorim/mha4mysql-manager/releases/download/v0.58/mha4mysql-manager-0.58-0.el7.centos.noarch.rpm
	#rpm -ivh mha4mysql-manager-0.58-0.el7.centos.noarch.rpm
	#rpm 安装mha4mysql-manager-0.58-0.el7.centos.noarch.rpm会依赖出错，用yum方式安装
	#yum install -y mha4mysql-manager-0.58-0.el7.centos.noarch.rpm
	
	
	yum install -y mha4mysql-manager-0.56-0.el6.noarch.rpm
	yum install mailx -y	#该软件是用来发送邮件的
```

**配置MHA（在manager节点上操作）**

	#创建目录
	mkdir -p /etc/mha/scripts
​	



### 第十：配置全局配置文件



```
vi /etc/masterha_default.cnf	#一定要是这个路径，不然后期masterha_check_ssh会提示未找到全局文件

[server default]
user=root                                    
#mysql用户
password=root     
#设置mysql中root用户的密码，这个密码是前文中创建监控用户的那个密码
ping_interval=2         
#设置监控主库，发送ping包的时间间隔，默认是3秒，尝试三次没有回应的时候自动进行railover
repl_password=repl    
#设置复制用户的密码
repl_user=repl              
#设置复制环境中的复制用户名
ssh_user=root                  
#设置ssh的登录用户名
secondary_check_script=masterha_secondary_check -s 172.10.0.211 -s 172.10.0.210 -s 172.10.0.215 
master_ip_failover_script="/etc/mha/scripts/master_ip_failover"
master_ip_online_change_script="/etc/mha/scripts/master_ip_online_change"
report_script="/etc/mha/scripts/send_report"
```





### 第十一：配置主配置文件

​	

```
vi /etc/mha/app1.cnf

[server default]
manager_log=/var/log/mha/app1/manager.log
manager_workdir=/var/log/mha/app1
 
[server1]
candidate_master=1
check_repl_delay=0

hostname=172.10.0.210
iport=3306
master_binlog_dir=/var/lib/mysql


[server2]
candidate_master=1
check_repl_delay=0
hostname=172.10.0.211
port=3306
#查看方式　find / -name mysql-bin*
master_binlog_dir=/var/lib/mysql


[server3]
#no_master=1		#不作为备用master
#ignore_fail=1	#开启slave宕机，无法启动MHA
hostname=172.10.0.215
port=3306
master_binlog_dir=/var/lib/mysql
```



### 第十二：配置VIP

#### 为了防止脑裂发生,推荐生产环境采用脚本的方式来管理虚拟 ip,而不是使用 keepalived来完成。 （注意：更改vip值，网卡名，$key值要统一，此处为88）

​	

```
#!/usr/bin/env perl

#  Copyright (C) 2011 DeNA Co.,Ltd.
#  You should have received a copy of the GNU General Public License
#   along with this program; if not, write to the Free Software
#  Foundation, Inc.,
#  51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA

## Note: This is a sample script and is not complete. Modify the script based on your environment.

use strict;
use warnings FATAL => 'all';

use Getopt::Long;
use MHA::DBHelper;

my (
  $command,        $ssh_user,         $orig_master_host,
  $orig_master_ip, $orig_master_port, $new_master_host,
  $new_master_ip,  $new_master_port,  $new_master_user,
  $new_master_password
);

my $vip = '172.10.0.218/24';
my $key = '88';
my $ssh_start_vip = "/sbin/ifconfig enp2s0:$key $vip";
my $ssh_stop_vip = "/sbin/ifconfig enp2s0:$key down";
GetOptions(
  'command=s'             => \$command,
  'ssh_user=s'            => \$ssh_user,
  'orig_master_host=s'    => \$orig_master_host,
  'orig_master_ip=s'      => \$orig_master_ip,
  'orig_master_port=i'    => \$orig_master_port,
  'new_master_host=s'     => \$new_master_host,
  'new_master_ip=s'       => \$new_master_ip,
  'new_master_port=i'     => \$new_master_port,
  'new_master_user=s'     => \$new_master_user,
  'new_master_password=s' => \$new_master_password,
);

exit &main();

sub main {
    print "\n\nIN SCRIPT TEST====$ssh_stop_vip==$ssh_start_vip===\n\n";
    if ( $command eq "stop" || $command eq "stopssh" ) {
        my $exit_code = 1;
        eval {
            print "Disabling the VIP on old master: $orig_master_host \n";
            &stop_vip();
            $exit_code = 0;
        };
        if ($@) {
            warn "Got Error: $@\n";
            exit $exit_code;
        }
        exit $exit_code;
    }
    elsif ( $command eq "start" ) {
        my $exit_code = 10;
        eval {
            print "Enabling the VIP - $vip on the new master - $new_master_host \n";
            &start_vip();
            $exit_code = 0;
        };
        if ($@) {
            warn $@;
            exit $exit_code;
        }
        exit $exit_code;
    }
    elsif ( $command eq "status" ) {
        print "Checking the Status of the script.. OK \n";
        exit 0;
    }
    else {
        &usage();
        exit 1;
    }
}
sub start_vip() {
    `ssh $ssh_user\@$new_master_host \" $ssh_start_vip \"`;
}
sub stop_vip() {
     return 0  unless  ($ssh_user);
    `ssh $ssh_user\@$orig_master_host \" $ssh_stop_vip \"`;
}
sub usage {
  print
"Usage: master_ip_failover --command=start|stop|stopssh|status --orig_master_host=host --orig_master_ip=ip --orig_master_port=port --new_master_host=host --new_master_ip=ip --new_master_port=port\n";
}
```

### 第十三：配置报警邮件脚本

	####mail邮件发送程序，需要先配置好发送这信息
​	

```
vi /etc/mail.rc


set from=hewublue@163.com
set smtp=smtp.163.com
set smtp-auth-user=hewublue@163.com
#拿163邮箱来说auth_password这个不是密码，而是授权码
set smtp-auth-password=a123456
set smtp-auth=login
```



	####这是具体的邮件发送脚本
​	

```
vi /etc/mha/scripts/send_report

#!/bin/bash
source /root/.bash_profile
# 解析变量
orig_master_host=`echo "$1" | awk -F = '{print $2}'`
new_master_host=`echo "$2" | awk -F = '{print $2}'`
new_slave_hosts=`echo "$3" | awk -F = '{print $2}'`
subject=`echo "$4" | awk -F = '{print $2}'`
body=`echo "$5" | awk -F = '{print $2}'`
#定义收件人地址
email="904826871@qq.com"

tac /var/log/mha/app1/manager.log | sed -n 2p | grep 'successfully' > /dev/null
if [ $? -eq 0 ]
	then
	messages=`echo -e "MHA $subject 主从切换成功\n master:$orig_master_host --> $new_master_host \n $body \n 当前从库:$new_slave_hosts"` 
	echo "$messages" | mail -s "Mysql 实例宕掉，MHA $subject 切换成功" $email >>/tmp/mailx.log 2>&1 
	else
	messages=`echo -e "MHA $subject 主从切换失败\n master:$orig_master_host --> $new_master_host \n $body" `
	echo "$messages" | mail -s ""Mysql 实例宕掉，MHA $subject 切换失败"" $email >>/tmp/mailx.log 2>&1  
fi
```

### 第十四：配置编写VIP脚本 （注意：更改vip值，网卡名）

​	

```
#!/bin/bash
source /root/.bash_profile

vip=`echo '172.10.0.218/24'`  #设置VIP
key=`echo '88'`

command=`echo "$1" | awk -F = '{print $2}'`
orig_master_host=`echo "$2" | awk -F = '{print $2}'`
new_master_host=`echo "$7" | awk -F = '{print $2}'`
orig_master_ssh_user=`echo "${12}" | awk -F = '{print $2}'`
new_master_ssh_user=`echo "${13}" | awk -F = '{print $2}'`

#要求服务的网卡识别名一样，都为ens192(这里是)
stop_vip=`echo "ssh root@$orig_master_host /usr/sbin/ifconfig enp2s0:$key down"`
start_vip=`echo "ssh root@$new_master_host /usr/sbin/ifconfig enp2s0:$key $vip"`

if [ $command = 'stop' ]
  then
	echo -e "\n\n\n****************************\n"
	echo -e "Disabled thi VIP - $vip on old master: $orig_master_host \n"
	$stop_vip
	if [ $? -eq 0 ]
	  then
	echo "Disabled the VIP successfully"
	  else
	echo "Disabled the VIP failed"
	fi
	echo -e "***************************\n\n\n"
  fi

if [ $command = 'start' -o $command = 'status' ]
  then
	echo -e "\n\n\n*************************\n"
	echo -e "Enabling the VIP - $vip on new master: $new_master_host \n"
	$start_vip
	if [ $? -eq 0 ]
	  then
	echo "Enabled the VIP successfully"
	  else
	echo "Enabled the VIP failed"
	fi
	echo -e "***************************\n\n\n"
fi

```

### 第十五：将脚本赋予可执行权限

​	

```
chmod +x /etc/mha/scripts/master_ip_failover 
chmod +x /etc/mha/scripts/master_ip_online_change 
chmod +x /etc/mha/scripts/send_report 
```

### 第十六：通过 masterha_check_ssh 验证 ssh 信任登录是否成功

​	

```
masterha_check_ssh --conf=/etc/mha/app1.cnf
		Wed May 16 23:17:58 2018 - [info] All SSH connection tests passed successfully.		#表示所有都成功
```

### 第十七：通过 masterha_check_repl 验证 mysql 主从复制是否成功（下面输出表示测试通过）

​	

```
[root#] masterha_check_repl --conf=/etc/mha/app1.cnf

		IN SCRIPT TEST====/sbin/ifconfig ens192:1 down==/sbin/ifconfig ens192:1 192.168.1.59/24===

		Checking the Status of the script.. OK 
		Wed May 16 23:59:42 2018 - [info]  OK.
		Wed May 16 23:59:42 2018 - [warning] shutdown_script is not defined.
		Wed May 16 23:59:42 2018 - [info] Got exit code 0 (Not master dead).

		MySQL Replication Health is OK.
```

## **启动MHA（注意：MHA监控脚本切换一次就会退出，需要再次启动）**

### 第十八：先在master上绑定vip，(只需要在master绑定这一次，以后会自动切换)

​	

```
/usr/sbin/ifconfig enp2s0:88 172.10.0.218/24
```



### 第十九：然后通过 masterha_manager 启动 MHA 监控(在manager角色上执行)

​	

```
mkdir /var/log/mha/app1 -p
touch /var/log/mha/app1/manager.log
```



	###启动mha监控进程，下面是一条命令
​	

```
nohup masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/mha/app1/manager.log 2>&1 &
```



	###这样 MHA 的日志保存在/var/log/mha/app1/manager.log 下
​	

```
nohup masterha_manager --conf=/usr/local/mha/ha1/ha1.cnf --ignore_fail_on_start --ignore_last_failover < /dev/null > /usr/local/mha/ha1/start.log 2>&1 &


```

--remove_dead_master_conf：该参数代表当发生主从切换后，老的主库的ip将会从配置文件中移除。这里暂时不使用该参数，因为发生使用该参数会将ha1.cnf配置文件搞乱。

--start_log:日志。

--ignore_last_failover：发生主从切换后，MHAmanager服务会自动停掉，且在manager_workdir目录下面生成文件app1.failover.complete，若要启动MHA，必须先删除该文件，该参数代表忽略上次MHA触发切换产生的文件，这里设置为-ignore_last_failover。 在缺省情况下，如果MHA检测到连续发生宕机，且两次宕机间隔不足8小时的话，则不会进行Failover，之所以这样限制是为了避免ping-pong效应。

--ignore_fail_on_start： 当有slave 节点宕掉时，默认是启动不了的，加上 --ignore_fail_on_start 即使有节点宕掉也能启动MHA，加上该参数会忽略启动文件中配置ignore_fail=1的server。

```
检查MHA的启动状态
tailf /var/log/mha/app1/manager.log
```


		###如果最后一行是如下，表明启动成功
​		

```
[info] Ping(SELECT) succeeded, waiting until MySQL doesn’t respond..
```

### 第二十：检查集群状态

```
masterha_check_status --conf=/etc/mha/app1.cnf
```

**切换测试之：自动切换（模拟master宕了）(实验测试当物理故障时，没有指定shutdown_script是没用的，不切换)**

```
第一：要实现自动 Failover,必须先启动 MHA Manager,否则无法自动切换
	A、杀掉主库 mysql 进程,模拟主库发生故障,进行自动 failover 操作。
	B、看 MHA 切换日志,了解整个切换过程
		tailf /var/log/mha/app1.log

第二：从上面的输出可以看出整个 MHA 的切换过程,共包括以下的步骤：
	1).配置文件检查阶段,这个阶段会检查整个集群配置文件配置
	2).宕机的 master 处理,这个阶段包括虚拟 ip 摘除操作,主机关机操作（由于没有定义power_manager脚本，不会关机）
	3).复制 dead maste 和最新 slave 相差的 relay log,并保存到 MHA Manger 具体的目录下
	4).识别含有最新更新的 slave
	5).应用从 master 保存的二进制日志事件(binlog events)（这点信息对于将故障master修复后加入集群很重要）
	6).提升一个 slave 为新的 master 进行复制
	7).使其他的 slave 连接新的 master 进行复制

第三：切换完成后,关注如下变化:
	1、vip 自动从原来的 master 切换到新的 master,同时,manager 节点的监控进程自动退出。
	2、在日志目录(/var/log/mha/app1)产生一个 app1.failover.complete 文件
	3、/etc/mha/app1.cnf 配置文件中原来老的 master 配置被删除。
```



### 切换测试之：在线切换（用于硬件升级）(很好使)**



#####MHA 在线切换是 MHA 除了自动监控切换换提供的另外一种方式,多用于诸如硬件升级，MySQL 数据库迁移等等。该方式提供快速切换和优雅的阻塞写入,无关关闭原有服务器，整个切换过程在 0.5-2s 的时间左右,大大减少了停机时间

#### 第一：注意点：前提，mha监控没有运行的情况下，才能进行

A、老master上的vip已经正确生效了
B、各个salve节点数据库的sql_IO和sql_sql进程都正常（即YES)

```
show slave status\G;
```


C、MHA脚本不能运行，若已处于监控状态，需要停掉它

```
masterha_stop --conf=/etc/mha/app1.cnf
```



	####若是mha监控进程在运行，会报如下错误
```
Sat May 19 03:40:00 2018 - [error][/usr/share/perl5/vendor_perl/MHA/MasterRotate.pm, ln143] Getting advisory lock failed on the current master. MHA Monitor runs on the current master. Stop MHA Manager/Monitor and try again.
Sat May 19 03:40:00 2018 - [error][/usr/share/perl5/vendor_perl/MHA/ManagerUtil.pm, ln177] Got ERROR:  at /usr/bin/masterha_master_switch line 53.
```

#### 第二：执行切换

	#####需要填写新的master的IP
```
masterha_master_switch --conf=/etc/mha/app1.cnf --master_state=alive --new_master_host=172.10.0.210 --orig_master_is_new_slave --running_updates_limit=10000 --interactive=0
```

#### 第三：MHA 在线切换基本步骤：

​	a、检测 MHA 配置置及确认当前 master
	b、决定新的 master
	c、阻塞写入到当前 master
	d、等待所有从服务器与现有 master 完成同步
	e、在新 master 授予写权限,以及并行切换从库
	f、重置原 master 为新 master 的 slave
	g、在线切换不会删除/etc/mha/app1.cnf 配置文件中原来老的 master 配置

##### 命令整理

```
1.原master出现故障
masterha_stop --conf=/etc/mha/app1.cnf #停止

masterha_master_switch --master_state=dead --conf=/etc/mha/app1.cnf --dead_master_host=172.10.0.210 --dead_master_port=3306 --new_master_host=172.10.0.211 --new_master_port=3306 --ignore_last_failover --interactive=0

2.把原master变为slave切换
masterha_master_switch --conf=/etc/mha/app1.cnf --master_state=alive --new_master_host=172.10.0.210 --new_master_port=3306 --orig_master_is_new_slave --interactive=0
```





### 如何将故障节点重新加入集群

通常情况下自动切换以后,原 master 可能已经废弃掉,待原 master 主机修复后,如果数据完整的情况下,可能想把原来 master 重新作为新主库的 slave,这时我们可以借助当时自动切换时刻的 MHA 日志来完成对原 master 的修复。(若是下面第二步：出现的是第二种，可以不用借助日志，定位binlog日志点，可以自动定位)

### (1)、修改 manager 配置文件（只针对自动切换的，在线切换不会删除配置）

	####将如下内容添加到/etc/mha/app1.conf 中,mha自动切换后会删除配置文件信息
```
[server1]
candidate_master=1
check_repl_delay=0
hostname=172.10.0.210
port=3306
master_binlog_dir=/var/lib/mysql
```



#### (2)、修复老的 master,然后设置为 slave从自动切换时刻的 MHA 日志上可以发现类似如下信息：

##### 意思是说,如果 Master 主机修复好了,可以在修复好后的 Master 上执行 CHANGE MASTER操作,作为新的 slave 库。

​	

```
cat /var/log/mha/app1/manager.log

​	Sat May 27 14:59:17 2017 - [info] All other slaves should start replication from here. Statement
	should be: CHANGE MASTER TO MASTER_HOST='172.16.213.232', MASTER_PORT=3306,
	MASTER_LOG_FILE='mysql-bin.000009', MASTER_LOG_POS=120, MASTER_USER='repl_user',
	MASTER_PASSWORD='xxx';
	
	或者

	Sat May 19 05:00:48 2018 - [info]  All other slaves should start replication from here. Statement should be: CHANGE MASTER TO MASTER_HOST='192.168.1.58', MASTER_PORT=3306, MASTER_AUTO_POSITION=1, MASTER_USER='repl_user', MASTER_PASSWORD='xxx';

```


#### 在老的 master 执行如下命令:(具体执行哪条，根据上面输出来确定，区别是一个有日志的定位，一个是自动定位)

​	

```
mysql> change master to master_host='node03',master_user='repl',master_password='repl',MASTER_AUTO_POSITION=1;

mysql> start slave;
mysql> show slave status\G;

```



	####这样,数据就开始同步到老的 master 上了。此时老的 master 已经重新加入集群,变成 mha集群中的一个 slave 角色了。

#### (3)、在 manger 节点上重新启动监控进程

​	

```
nohup masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/mha/app1/manager.log 2>&1 &
```

参看文章：<https://blog.csdn.net/qq_34605594/article/details/77387872?locationNum=4&fps=1>

**附：MHA切换阶段的工作具体干了那些事**
<https://blog.csdn.net/shiyu1157758655/article/details/70242093>

**附：高俊峰老师的课件**以供学习
<https://qiniu.wsfnk.com/%E7%AC%AC12%E8%AF%BE-4%E4%BC%81%E4%B8%9A%E5%B8%B8%E8%A7%81MySQL%E6%9E%B6%E6%9E%84%E5%BA%94%E7%94%A8%E5%AE%9E%E6%88%98%E4%B9%8BMHA%E6%9E%B6%E6%9E%84.pdf>