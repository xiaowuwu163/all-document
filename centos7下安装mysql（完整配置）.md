# centos7下安装mysql（完整配置）

2018年06月03日 16:27:43 [zengrui_0337](https://me.csdn.net/baidu_32872293) 阅读数：11955

 版权声明：本文为博主原创文章，未经博主允许不得转载。	https://blog.csdn.net/baidu_32872293/article/details/80557668

**1. 下载并安装MySQL官方的Yum Repository**

```
[root@localhost ~]# wget -i -c https://repo.mysql.com/mysql57-community-release-el7-10.noarch.rpm
http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
https://repo.mysql.com/mysql57-community-release-el7-10.noarch.rpm
```

**使用上面的命令直接安装Yum Repository**

```
[root@localhost ~]# yum -y install mysql57-community-release-el7-10.noarch.rpm
```

**安装MySQL服务器**

```
root@localhost ~]# yum -y install mysql-community-server
```

**2. MySQL数据库设置** 
启动MySQL

```
[root@localhost ~]# systemctl start  mysqld.service
```

查看MySQL运行状态

```
[root@localhost ~]# systemctl status mysqld.service
```

此时MySQL已经开始正常运行，需要找出root的密码

```
[root@localhost ~]# grep "password" /var/log/mysqld.log
```

如下命令登录mysql

```
# mysql -uroot -p
```

输入初始密码，此时不能做任何事情，因为MYSQL默认必须修改密码才能正常使用

```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new password';

# 这里会遇到一个问题，新密码设置过于简单会报错123
```

可通过如下命令查看完整的初始密码规则

```
mysql>show variables like 'validate_password';
```

可通过如下命令修改

```
mysql> set global validate_password_policy=0;
mysql> set global validate_password_length=1;
```

还有一个问题就是Yum Repository,以后每次 yum 操作都会自动更新，需要把这个卸载掉

```
[root@localhost ~]# yum -y remove mysql57-community-release-el7-10.noarch
```

远程登录数据库出现下面出错信息 
ERROR 2003 (HY000): Can’t connect to MySQL server on ‘xxx.xxx.xxx.xxx’, 
原因是没有授予相应的权限

```
#任何主机
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;

#指定主机
mysql>GRANT ALL PRIVILEGES ON *.* TO 'jack'@’10.10.50.127’ IDENTIFIED BY '654321' WITH GRANT OPTION;

# 然后刷新权限
mysql>flush privileges;
```

修改mysql数据库总的user表使相的用户能从某一主机登录

```
mysql>use mysql;
mysql>update user set host = '%' where user = 'root';
mysql>select host, user from user;
```

客户端提供MYSQL的环境，但是不支持中文,通过以下命令可以查看mysql的字符集

```
mysql>show variables like 'character_set%';
```

显示如下：

![image](https://note.youdao.com/yws/public/resource/3603e7790bd5457e32e7db9b4b934ebf/xmlnote/79CB4D7450C948AC8DE3744F5CF8020D/2946)

为了让 MySQL支持中文，需要把字符集改成UTF-8，方法如下

```
# vim /etc/my.cnf
```

改成如下内容

```
[client]
port=3306
socket=/var/lib/mysql/mysql.sock
default-character-set=utf8

[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
character-set-server=utf8

[mysql]
no-auto-rehash
default-character-set=utf8

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

重启mysql服务

```
# service mysqld restart
```

重新查看数据库编码

```
show variables like 'character_set%';
```