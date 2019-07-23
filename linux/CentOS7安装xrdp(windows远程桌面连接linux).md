# CentOS7安装xrdp(windows远程桌面连接linux)

2018年08月20日 12:27:31 [xiaowen5555555](https://me.csdn.net/sgrrmswtvt) 阅读数：7484

**前提:** 
CentOS安装桌面，如果无桌面，请执行：

```
yum -y groups install "GNOME Desktop"
startx

```

------

------

> `方法一`

配置源

```
yum install  epel* -y

```

安装xrdp

```
yum --enablerepo=epel -y install xrdp

```

> `方法二`

1、安装xrdp 
更具自己的系统位数选择对应的包(如果是32位使用则选择i386,如果是64位,请选择x86_64),查找的方法是到镜像网站<http://mirrors.ustc.edu.cn/fedora/epel/7>上进入到对应的目录,查找以epel-release-开头的RPM包)

```
http://mirrors.ustc.edu.cn/epel/7/x86_64/Packages/e/epel-release-7-11.noarch.rpm
rpm -Uvh epel-release-7-11.noarch.rpm
yum install xrdp

```

2、安装 tigervnc

```
yum install tigervnc tigervnc-server

```

3、配置SELinux , 否则可能无法启动xrdp服务,或者启动出错

```
chcon -t bin_t /usr/sbin/xrdp
chcon -t bin_t /usr/sbin/xrdp-sesman

```

------

> `下面的内容都是一样的`

------

启动xrdp并设置开机启动

```
systemctl start xrdp
systemctl enable xrdp

vncpasswd #创建vnc远程连接密码

```

安装好了之后将防火墙关闭,或者开放3389端口

```
//开放3389端口
firewall-cmd --permanent --zone=public --add-port=3389/tcp
firewall-cmd --reload
======================或者关闭防火墙
//临时关闭
systemctl stop firewalld
//禁止开机启动
systemctl disable firewalld
```