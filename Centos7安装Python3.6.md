# Centos7安装Python3.6

2018年09月21日 13:31:28 [小龙魂](https://me.csdn.net/yhflyl) 阅读数：54

 版权声明：本文为博主原创文章，未经博主允许不得转载。	https://blog.csdn.net/yhflyl/article/details/82800097

#### 首先安装依赖

```
yum -y groupinstall "Development tools"
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel

```

#### 下载python3的安装包

```
wget https://www.python.org/ftp/python/3.6.2/Python-3.6.2.tar.xz

```

#### 创建python3目录并解压：

```
mkdir /usr/local/python3 
tar -xvJf  Python-3.6.2.tar.xz

```

#### 编译安装

```
cd Python-3.6.2
./configure --prefix=/usr/local/python3
make && make install

```

#### 创建连接测试安装

```
ln -s /usr/local/python3/bin/python3 /usr/bin/python3  # 在/usr/bin 目录下存放python3的快捷方式
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3  # 在/usr/bin 目录下存放pip3的快捷方式

```

```
[root@localhost ~]# python3
Python 3.6.2 (default, Sep 21 2018, 12:04:18) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-28)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 

```

#### 注：Linux目录的作用

```
/bin 二进制可执行命令
/dev 设备特殊文件
/etc 系统管理和配置文件
/etc/rc.d 启动的配置文件和脚本
/home 用户主目录的基点，比如用户user的主目录就是/home/user，可以用~user表示
/lib 标准程序设计库，又叫动态链接共享库，作用类似windows里的.dll文件
/sbin 超级管理命令，这里存放的是系统管理员使用的管理程序
/tmp 公共的临时文件存储点
/root 系统管理员的主目录
/mnt 系统提供这个目录是让用户临时挂载其他的文件系统
/lost+found这个目录平时是空的，系统非正常关机而留下“无家可归”的文件（windows下叫什么.chk）就在这里
/proc 虚拟的目录，是系统内存的映射。可直接访问这个目录来获取系统信息。
/var 某些大文件的溢出区，比方说各种服务的日志文件
/usr 最庞大的目录，要用到的应用程序和文件几乎都在这个目录，其中包含：
————/usr/x11R6 存放x window的目录
————/usr/bin 众多的应用程序
————/usr/sbin 超级用户的一些管理程序
————/usr/doc linux文档
————/usr/include linux下开发和编译应用程序所需要的头文件
————/usr/lib 常用的动态链接库和软件包的配置文件
————/usr/man 帮助文档
————/usr/src 源代码，linux内核的源代码就放在/usr/src/linux里
————/usr/local/bin 本地增加的命令
————/usr/local/lib 本地增加的库根文件系统
```

#### python3.5安装报错

python3.5: error while loading shared libraries: libpython3.5m.so.1.0: cannot open shared object file: No such file or directory

 

原因是因为python运行时没有加载到libpython3.5m.so.1.0 这个库文件     将其复制到响应目录OK

解决方法:

[root@www Python-3.5.0]# cd /root/test/Python-3.5.0     进入解压后的编译目录

[root@www Python-3.5.0]#  cp libpython3.5m.so.1.0 /usr/local/lib64/
[root@www Python-3.5.0]#  cp libpython3.5m.so.1.0 /usr/lib/ 
[root@www Python-3.5.0]#  cp libpython3.5m.so.1.0 /usr/lib64/