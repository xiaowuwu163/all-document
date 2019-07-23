# centos7 安装GNOME桌面

[CentOS](https://www.linuxidc.com/topicnews.aspx?tid=14) 7 默认是没有图形化界面的，但我们很多人在习惯了 Windows 的图形化界面之后，总是希望有一个图形化界面从而方便我们使用，这里介绍一下 CentOS７安装图形化桌面系统的方法。

### 一、进入 root 模式

因为权限限制，所以我们需要进入 root 模式，开机使用 root 登陆或者系统运行中切换为 root 用户均可。

![img](https://www.linuxidc.com/upload/2018_04/180422080385831.png)

### 二、安装  X 窗口系统

1、首先安装X(X Window System)，命令为

yum groupinstall "X Window System" 　//注意有引号

![img](https://www.linuxidc.com/upload/2018_04/180422080385832.png)

 然后系统会自动寻找最近的网络进行相关文件的下载

![img](https://www.linuxidc.com/upload/2018_04/180422080385833.png)

选择 y ，然后开始下载需要的 package

![img](https://www.linuxidc.com/upload/2018_04/180422080385834.png)

选择 y，开始进行安装

![img](https://www.linuxidc.com/upload/2018_04/180422080385835.png)

当出现 Complete！说明这里安装成功了。

在这里我们可以检查一下我们已经安装的软件以及可以安装的软件，命令为

yum grouplist

![img](https://www.linuxidc.com/upload/2018_04/180422080385836.png)

### 三、安装图形界面软件 GNOME

然后我们开始安装我们需要的图形界面软件，GNOME(GNOME Desktop)

> 特别注意！！！！一定要注意**名称必须对应，**否则会出现No packages in any requested group available to install or update 的错误**。**这是因为不同版本的CentOS的软件名可能不同（其他 Linux 系统也是类似的）

![img](https://www.linuxidc.com/upload/2018_04/180422080385837.png)

 如上图，安装命令为：

yum groupinstall "GNOME Desktop" "Graphical Administration Tools"

![img](https://www.linuxidc.com/upload/2018_04/180422080385838.png)

 选择 y 开始下载需要安装的 package

![img](https://www.linuxidc.com/upload/2018_04/180422080385839.png)

到这里就安装完成了。

这时，我们可以通过命令 startx 进入图形界面，第一次进入会比较慢，请耐心等待。（可能需要重启，命令为reboot）

> ps：
>
> - 如果安装完成后，虚拟机无法打开，我们需要调整虚拟机**分配内存大小（注意不是磁盘大小），**1024M基本够用。
> - 如果安装完成后，虚拟机报错0x0000005c，请关闭虚拟机的**3D加速**功能（取消勾选）

### 四、更新系统的默认运行级别

经过上面的操作，系统启动默认还是命令行页面的，需要我们进行切换。如果想要使系统启动即为图形化窗口，需要执行下面的命令

ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target



### 五、安装yum groupinstall "GNOME Desktop"时

```
Transaction check error:
  file /boot/efi/EFI/centos from install of fwupdate-efi-12-5.el7.centos.x86_64 conflicts with file from package grub2-common-1:2.02-0.65.el7.centos.2.noarch

Error Summary
```

更新软件包 

```
yum update grub2-common
yum install fwupdate-efi
```