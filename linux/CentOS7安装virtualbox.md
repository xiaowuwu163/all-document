# [CentOS7安装virtualbox](https://www.cnblogs.com/wangshuyi/p/6927113.html)

1.进入virtualbox官网

https://www.virtualbox.org/

2.点击download

![img](https://images2015.cnblogs.com/blog/986203/201706/986203-20170601084748914-2059510238.png)

3.点击Linux distributions

![img](https://images2015.cnblogs.com/blog/986203/201706/986203-20170601084936118-144327245.png)

4.向下翻至如图，并且进入同种框选页面

 ![img](https://images2015.cnblogs.com/blog/986203/201706/986203-20170601085229664-1774806353.png)

5.在/etc/yum.repos.d/目录下新建virtualbox.repo并写入如下内容

```
[virtualbox]
name=Oracle Linux / RHEL / CentOS-$releasever / $basearch - VirtualBox
baseurl=http://download.virtualbox.org/virtualbox/rpm/el/$releasever/$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://www.virtualbox.org/download/oracle_vbox.asc
```

6.更新yum缓存

yum clean all

yum makecache

7.安装virtualbox

yum install VirtualBox-5.1