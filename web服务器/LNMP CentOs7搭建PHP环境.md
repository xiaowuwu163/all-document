LNMP CentOs7搭建PHP环境



Yum添加 Epel源

```
wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ivh epel-release-latest-7.noarch.rpm
yum repolist      ##检查是否已添加至源列表
```

安装Nginx

```
yum -y install nginx   使用yum安装nginx

systemctl start nginx  启动nginx，浏览器输入ip就可以看到nginx的欢迎页
```

修改Nginx配置文件以支持PHP解析

```
nginx配置文件默认放在/etc/nginx/nginx.confvi /etc/nginx/nginx.conf在server区间里加入以下内容

注释掉本来的这两行 

        # location / { 

        #  } 

 location / { 

        root   /usr/share/nginx/html; 

        index  index.php index.html index.htm; 

    } 

  location ~ \.php$ { 

         root           html; 

         fastcgi_pass   127.0.0.1:9000; 

         fastcgi_index  index.php; 

         fastcgi_param  SCRIPT_FILENAME  /usr/share/nginx/html$fastcgi_script_name; 

         include        fastcgi_params; 

     } 
     

systemctl restart nginx 重启nginx
```

安装PHP



```
检查当前安装的PHP包

    yum list installed | grep php

如果有安装的PHP包，先删除他们

(这条命令看情况执行看清楚你安装的包用yum remove删除)

 yum remove php.x86_64 php-cli.x86_64 php-common.x86_64 php-gd.x86_64 php-ldap.x86_64 php-mbstring.x86_64 php-mcrypt.x86_643
 
 
 
 
```

添加PHP的yum源

```
Centos 5.X   rpm -Uvh http://mirror.webtatic.com/yum/el5/latest.rpm
CentOs 6.x   rpm -Uvh http://mirror.webtatic.com/yum/el6/latest.rpm
CentOs 7.X   rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm

或者使用wget

wget https://mirror.webtatic.com/yum/el7/epel-release.rpm
rpm -ivh epel-release.rpm
```

如果想删除上面安装的Yum源包，重新安装

```
rpm -qa | grep webstatic

rpm -e  上面搜索到的包即可
```

自行选择要安装什么版本的PHP

```
php5.6
yum install php56w.x86_64 php56w-cli.x86_64 php56w-common.x86_64 php56w-gd.x86_64 php56w-ldap.x86_64 php56w-mbstring.x86_64 php56w-mcrypt.x86_64 php56w-mysql.x86_64 php56w-pdo.x86_64

 

php5.5
yum install php55w.x86_64 php55w-cli.x86_64 php55w-common.x86_64 php55w-gd.x86_64 php55w-ldap.x86_64 php55w-mbstring.x86_64 php55w-mcrypt.x86_64 php55w-mysql.x86_64 php55w-pdo.x86_64

 

php7
yum install php70w.x86_64 php70w-cli.x86_64 php70w-common.x86_64 php70w-gd.x86_64 php70w-ldap.x86_64 php70w-mbstring.x86_64 php70w-mcrypt.x86_64 php70w-mysql.x86_64 php70w-pdo.x86_64
```

安装PHP-FPM

```
5.5yum install php55w-fpm 

5.6yum install php56w-fpm 

7.0yum install php70w-fpm
```

启动php-fpm

```
systemctl start php-fpm
```

配置php.ini

```
vi /etc/php.ini 按下esc进入命令模式，输入:/cgi.fix_pathinfo,按n

进行下一个查找，找到指定cgi.fix_pathinfo, 修改为=0；
```

安装Mysql 我不放mariadb了 直接mysql

```
yum –y install mysql

yum –y install mysql-devel

 

添加官方mysql-server的yum源

 wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
 
 
 安装源
 rpm -ivh mysql-community-release-el7-5.noarch.rpm 

安装mysql-server

 yum install mysql-community-server
 
 
 启动mysql
 systemctl restart mysqld
```

如果有防火墙 要开放80和3306端口



```
centos7用的是firewallfirewall-cmd --zone=public --add-port=80/tcp --permanentfirewall-cmd --zone=public --add-port=3306/tcp --permanent
```

重新加载防火墙



```
firewall-cmd --reload
```

