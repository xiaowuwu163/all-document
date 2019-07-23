# linux远程配置

本文主要是讲解如果理由VNC实现Windows远程访问Ubuntu 16.04，其实网上有很多类似教程，但是很多需要安装第三方桌面（xfce桌面等等），而且很多人不太喜欢安装第三方桌面，很多人像笔者一样喜欢原生自带的桌面（O(∩_∩)O哈哈~有点强迫症）。今天笔者给大家介绍一下，不需要安装其他桌面，使用Ubuntu 16.04原生自带桌面如何进行远程访问。https://www.cnblogs.com/xuliangxing/p/7642650.html

# 一、设置Ubuntu 16.04 允许进行远程控制

 　　首先，我们先设置Ubuntu的远程控制，将其设置为允许被远程连接，进入系统-》首选项-》桌面共享，或者直接搜索桌面共享，如图所示

![img](https://images2017.cnblogs.com/blog/506829/201710/506829-20171011164631684-1366734089.png)

　　将【允许其他人查看您的桌面】这一项勾上，然后在安全那项，勾选【要求远程用户输入此密码】，并设置远程密码。并且我们取消勾选【必须为对本机器的每次访问进行确定】（这样做，是为了被远程的时候不需要再确认，否则每次远程都要人为确认才能被远程，会很繁琐）如图所示：

![img](https://images2017.cnblogs.com/blog/506829/201710/506829-20171011220233980-553462263.png)

# 二、安装vncserver

 　　其次，打开终端，我们需要安装vncserver的基础服务，输入以下命令：

```
sudo apt-get install xrdp vnc4server xbase-clients
```

　　如图所示：

![img](https://images2017.cnblogs.com/blog/506829/201710/506829-20171011220254465-1407251331.png)

# 三、安装dconf-editor(取消权限限制)

 　　再次，我们需要取消掉请求加密的功能，否则缺少这一步是无法远程上的，这个时候我们需要安装dconf-editor工具进行配置，输入以下命令：

```
sudo apt-get install dconf-editor
```

　　如图所示：

![img](https://images2017.cnblogs.com/blog/506829/201710/506829-20171011220330199-1855036667.png)

　　安装完成之后，我们需要打开dconf-editor工具，在桌面搜索dconf-editor打开，如图所示：

![img](https://images2017.cnblogs.com/blog/506829/201710/506829-20171011220344637-1503609352.png)

　　打开之后，依次展开org->gnome->desktop->remote-access，然后取消 “requlre-encryption”的勾选即可。如图所示：

![img](https://images2017.cnblogs.com/blog/506829/201710/506829-20171011220355762-1134949156.png)

　　至此，前期准备工作已经完成，后面直接通过VNC工具或者Windows自带的mstsc(远程桌面控制)进行访问就行。

# 四、远程连接Ubuntu 16.04

　　获取当前的IP地址，命令ifconfig即可得到，笔者的当前的Ubuntu的IP地址为：192.168.8.203，然后通过IP地址就可以远程访问了。

### 　　方法一、通过VNC Viewer客户端进行访问

　　大家可以到VNC官网(<https://www.realvnc.com/en/connect/download/viewer/>)下载最新的版本，根据自己实际情况，选择相对应的版本，如图所示：

![img](https://images2017.cnblogs.com/blog/506829/201710/506829-20171011220522871-833869384.png)

　　输入我们需要远程控制的PC主机的IP，如图所示：

![img](https://images2017.cnblogs.com/blog/506829/201710/506829-20171011220540402-1270925189.png)