# 在CentOS7中yum安装MariaDB10.3

2018年05月26日 16:15:11 [zbljz98](https://me.csdn.net/zbljz98) 阅读数 5609

MariaDB和MySQL的关系：

MariaDB数据库管理系统是MySQL的一个分支。开发这个分支的原因之一是：甲骨文公司收购了MySQL后，有将MySQL闭源的潜在风险，因此社区采用分支的方式来避开这个风险。

MariaDB的目的是完全兼容MySQL，包括API和命令行，使之能轻松成为MySQL的代替品。在存储引擎方面，10.0.9版起使用XtraDB（名称代号为Aria）来代替MySQL的InnoDB。MariaDB由MySQL的创始人麦克尔·维德纽斯主导开发，他早前曾以10亿美元的价格，将自己创建的公司MySQL AB卖给了SUN，此后，随着SUN被甲骨文收购，MySQL的所有权也落入Oracle的手中。

MariaDB直到5.5版本，均依照MySQL的版本。因此，使用MariaDB5.5的人会从MySQL 5.5中了解到MariaDB的所有功能。从2012年11月12日起发布的10.0.0版开始，不再依照MySQL的版号。10.0.x版以5.5版为基础，加上移植自MySQL 5.6版的功能和自行开发的新功能。

现在的MariaDB的10.3版本的吞吐性能高出了MySQL5.6社区版两倍，并且随着请求越来越高，差距越来越大。

添加MariaDB的repo源：

1、进入/etc/yum.repo.d下，添加CentOS-MariaDB.repo文件，其中添加内容如下

```
# MariaDB 10.3 CentOS repository list - created 2018-05-26 07:55 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.3/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

2、清除Yum的缓存并重新建立

```
yum clean all
yum makecache
```

![img](https://img-blog.csdn.net/20180526155825325?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pibGp6OTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180526155724583?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pibGp6OTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3、打印MariaDB源中的软件包：

```
yum list --disablerepo=\* --enablerepo=mariadb
```

![img](https://img-blog.csdn.net/20180526160152146?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pibGp6OTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

test为测试工具，backup为备份工具

4、安装MariaDB数据库：

```
yum install MariaDB-client MariaDB-server MariaDB-devel -y
```

![img](https://img-blog.csdn.net/20180526161300961?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pibGp6OTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

5、启动数据库并设置为开机自启

```
systemctl start mariadb
systemctl enable mariadb
```

![img](https://img-blog.csdn.net/20180526170708240?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pibGp6OTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

6、初始化数据库，并删除测试数据库及更改权限和设置密码

```
mysql_secure_installation
```

![img](https://img-blog.csdn.net/20180526170809746?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pibGp6OTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

输入数据库设置密码

![img](https://img-blog.csdn.net/20180526170836933?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pibGp6OTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

是否设置root密码，输入Y进行设置

![img](https://img-blog.csdn.net/20180526170914790?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pibGp6OTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

是否移除匿名用户，输入Y移除

![img](https://img-blog.csdn.net/20180526170950647?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pibGp6OTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

是否拒绝root用户的远程登陆，根据实际情况选择

![img](https://img-blog.csdn.net/20180526171026172?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pibGp6OTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

是否刷新权限表，输入Y刷新权限表

![img](https://img-blog.csdn.net/20180526171055114?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pibGp6OTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

出现此界面，数据库安全设置完成。

7、连接数据库，并查询版本

```
mysql -uroot -p -A
```

其中-u制定用户，-p使用密码，-A为不预先读取数据库。

```
select version();
```

```
show full processlist;
```

![img](https://img-blog.csdn.net/20180526171355479?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pibGp6OTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

可查看当前用户，及登陆地址，选择的数据库，数据库引擎。